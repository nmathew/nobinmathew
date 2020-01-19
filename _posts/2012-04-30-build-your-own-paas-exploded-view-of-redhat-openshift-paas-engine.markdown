---
author: Cloud Fundoo
comments: true
date: 2012-04-30 17:54:27+00:00
layout: post
slug: build-your-own-paas-exploded-view-of-redhat-openshift-paas-engine
title: 'Build Your own PaaS : Exploded View of Redhat OpenShift PaaS Engine'
wordpress_id: 113
tags:
- Architecture
- AWS
- Cloud
- Crankcase
- IaaS
- Open Source
- OpenShift
- PaaS
- RedHat
---

Build your own PaaS (Platform as a Service), sounds like a impossible dream? No it is not, now with open source Redhat OpenShift Engine [OpenShift Origin](https://github.com/openshift) you can build PaaS Engine, in your laptop, inside your company or you can provide public PaaS like Redhat OpenShift.

[![](https://cloudfundoo.files.wordpress.com/2012/04/finished_crankcase2.jpg?w=300)](https://cloudfundoo.files.wordpress.com/2012/04/finished_crankcase2.jpg)

The picture above is of a crankcase, this is what you see when you open a reciprocal engine, crankcase is the one which holds various engine parts like cylinders, heads and crankshaft.

OpenShift engine also has a Crankcase, which does similar functionality of reciprocal engine crankcase. Openshift Crankcase the is the core of  Redhat OpenShift PaaS, which is completely open source and you can find entire code in github, see [Crankcase](https://github.com/openshift/crankcase).

Redhat open sourced the entire software which powers the OpenShift PaaS, see the release [here](http://www.redhat.com/about/news/archive/2012/4/Announcing-OpenShift-Origin-Open-Source-Code-For-Platform-as-a-Service?utm_source=twitterfeed&utm_medium=twitter) and entire source code in [Github](https://github.com/openshift). See this [page](https://openshift.redhat.com/app/opensource/download) for more information on source code.

[![](https://cloudfundoo.files.wordpress.com/2012/04/architecture_overview-preview.jpg)](https://cloudfundoo.files.wordpress.com/2012/04/architecture_overview-preview.jpg)

The Architecture of OpenShift is there in the [wiki](https://openshift.redhat.com/community/wiki/architecture-overview). You can use this to power a PaaS from your laptop, there is a wiki [entry](https://openshift.redhat.com/community/wiki/build-your-own) describing how you can setup a local PaaS using a KVM virtual machine instance.

There is a even simpler method to power your own PaaS from your laptop, for that you need to download LiveCD from [here](http://mirror.openshift.com/pub/fedora-remix/16/x86_64/openshift_origin_livecd.iso). If you mount this LiveCD in a virtual Machine (VirtualBox/KVM) instance, you can get your local OpenShift powered from your laptop. See this [page](https://openshift.redhat.com/community/wiki/getting-started-with-openshift-origin-livecd) for steps.

[![](https://cloudfundoo.files.wordpress.com/2012/04/cloud_computing_layers.png?w=300)](https://cloudfundoo.files.wordpress.com/2012/04/cloud_computing_layers.png)

So how we will power Public PaaS?  for that you need huge infrastructure like servers, storage, networks, power, cooling etc. There is a cloud service segment called IaaS, which provide these infrastructure as a service. [Amazon Web Services](http://aws.amazon.com/) (AWS) is a commercial IaaS provider, which Redhat uses for powering it's [OpenShift](https://openshift.redhat.com/app/) PaaS. With IaaS you can run OpenShift Engine on top of that cloud infrastructure to create your own PaaS.  If you want to have your own IaaS, then you can use [OpenStack](http://openstack.org/), which is a open source cloud OS/Framework for building IaaS.
