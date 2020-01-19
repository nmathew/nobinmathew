---
author: Cloud Fundoo
comments: true
date: 2015-03-16 11:13:19+00:00
layout: post
slug: devstack-in-single-vm-containers-lxc-on-openstack
title: 'Devstack in single VM: Containers( LXC ) on Openstack'
wordpress_id: 298
tags:
- Cloud
- container
- devstack
- Linux
- lxc
- OpenStack
- python
- ubtunu
- vm
---

Linux containers are lightweight virtualization (no virtualization, only compartmentalization). I was trying to bring plain containers using lxc, not using other frameworks like docker, or many others.

There is a blog entry from canonical developer, [here](https://zulcss.wordpress.com/2014/11/14/nova-compute-flex-introduction-and-getting-started/). But when I tried it was giving some problems. So I had to solve some issues.

First we need to create a Ubuntu 14.04 server/desktop VM, for server un-install juju(which may give some port binding issues).

Follow instructions given in the link.

{% highlight bash %}

dd if=/dev/zero of=<name of your large file> bs=1024k count=4000

(Create 4GB partition instead of 2GB)

sudo mkfs.btrfs <name of your large file>
sudo mount -t brfs -o user_subvol_rm_allowed <name of your large file> <mount point>

{% endhighlight %}



This is the directory where nova LXC driver will store configuration and lxc images.

Now we should install devstack and ncflex (which is the nova LXC driver)

{% highlight bash %}

mkdir -p /opt/stack

git clone https://github.com/zulcss/nova-compute-flex -b stable/juno
git clone https://github.com/openstack-dev/devstack -b stable/juno

{% endhighlight %}

Then run

{% highlight bash %}

/opt/stack/nova-compute-flex/contrib/devstack/prepare_devstack.sh

{% endhighlight %}

after this localrc(from devstack directory) will look like

{% highlight bash %}
cat localrc

virt_type=flex
{% endhighlight %}

Then we should edit the localrc to contain this

{% highlight bash %}

VIRT_DRIVER=flex

disable_service n-net
enable_service q-svc
enable_service q-agt
enable_service q-dhcp
enable_service q-l3
enable_service q-meta

GIT_BASE=https://github.com
DATA_DIR=/home/cloudfundoo/btrfs_dir

ADMIN_PASSWORD=devstack
MYSQL_PASSWORD=devstack
RABBIT_PASSWORD=devstack
SERVICE_PASSWORD=devstack
SERVICE_TOKEN=token

NOVA_BRANCH=stable/juno
CINDER_BRANCH=stable/juno
GLANCE_BRANCH=stable/juno
HORIZON_BRANCH=stable/juno
KEYSTONE_BRANCH=stable/juno
NEUTRON_BRANCH=stable/juno
LOG=True

FLOATING_RANGE=192.168.1.96/27
PUBLIC_NETWORK_GATEWAY=192.168.1.97 < this is the gateway>

{% endhighlight %}

my lan is 192.168.1.xx/24, in that one section is assigned to lxc which will get spawned so FLOATING_RANGE=192.168.1.96/27

Then we need some hacks to bring up the lxc.

we need to have

{% highlight bash %}
iniset $NOVA_CONF DEFAULT scheduler_default_filters "AllHostsFilter"
{% endhighlight %}

in devstack/lib/nova. This is required because we are bringing up all the stuff in single VM. nova scheduler filters are there to choose the best host for spawning the lxc. For our case there is only one VM, so take all hosts. For more info see [here](https://ask.openstack.org/en/question/21820/nova-scheduler-failed-to-schedule_run_instance-no-valid-host-was-found/).

neutron host not reachable.

Comment out this line in /etc/neutron/neutron.conf and restart it. see [here](https://ask.openstack.org/en/question/32666/connection-to-neutron-failederrno-11-connection-refused/).

{% highlight bash %}
#signing_dir = /var/cache/neutron
{% endhighlight %}

Then add a python binary /usr/bin/lxc-usernet-manage
{% highlight bash %}

chmod a+x /usr/bin/lxc-usernet-manage

cat /usr/bin/lxc-usernet-manage
{% endhighlight %}
{% highlight python %}

#! /usr/bin/python
# PBR Generated from u'console_scripts'

import sys

from ncflex.nova.virt.flex.lxc_usernet import manage_main

if __name__ == "__main__":
sys.exit(manage_main())

{% endhighlight %}

This python script is executed by rootwrap of openstack.

From here it is normal process of spawning VM.

When I spawned lxc was coming up, but I was unable to login with key pairs.
