---
author: Cloud Fundoo
comments: true
date: 2012-07-09 16:05:02+00:00
layout: post
slug: ssltls-sockets-programming-using-openssl-and-polarssl
title: SSL/TLS sockets programming using openssl and polarssl
wordpress_id: 246
tags:
- Client
- libssl
- Linux
- mongoose
- openssl
- polarssl
- server
- sockets
- SSL
- TLS
---

I was trying to do some modifications to mongoose http server to use polarssl, to do that I tried to understand openssl and polarssl libraries, for that I wrote some example client/server programs, See [https://github.com/CloudFundoo/SSL-TLS-clientserver](https://github.com/CloudFundoo/SSL-TLS-clientserver), may be useful.

To compile this with libssl, first install openssl(or libssl) library, dev package and tools package.

Then sync our example source code and give "make libssl", this will give both server and client executables.

Compiling with polarssl has some problems, first we need to compile the polarssl library since we don't have polarssl packages for ubuntu.

Download polarssl source tarball and unzip to some folder, then issue following commands

{% highlight bash %}
$make 
$make install
$make -C library/ shared
{% endhighlight %}

"make all/make" will not create shared library, so we need to issue "make shared" for that. After creating shared library copy it to "/usr/lib". Polarssl library installation is complete now.

Now issue "make polarssl" in our example directory, it will give both server and client executables.

Next step is creating certificate file,  to generate CA, server and client certificates, follow below instructions

{% highlight bash %}
##generate CA cert and key files
$openssl genrsa -des3 -out ca.key 4096
$openssl req -new -x509 -days 365 -key ca.key -out ca.crt

##generate Server cert and key files
$openssl genrsa -des3 -out ssl_server.key 4096
$openssl req -new -key ssl_server.key -out ssl_server.csr

##Sign using CA
$openssl x509 -req -days 365 -in ssl_server.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out ssl_server.crt

##generate Client cert and key files
$openssl genrsa -des3 -out ssl_client.key 4096
$openssl req -new -key ssl_client.key -out ssl_client.csr

##Sign using CA
$openssl x509 -req -days 365 -in ssl_client.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out ssl_client.crt
{% endhighlight %}
