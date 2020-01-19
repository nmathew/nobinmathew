---
author: Cloud Fundoo
comments: true
date: 2012-04-14 07:05:16+00:00
layout: post
slug: write-an-openshift-app-using-play
title: Write an OpenShift app using Play !
wordpress_id: 29
tags:
- Cloud
- JAVA
- OpenShift
- Play
- Play Framework
---

First create a simple play application, for that run the following commands from cmdline

{% highlight bash %}    
    play new simpleapp
{% endhighlight %}    

where simpleapp is the name of the application, this will create a folder named simpleapp containing play auto generated stuffs.

Now edit the simpleapp/app/views/Application/index.html, replace the contents of index.html with
    
{% highlight html %}    
    <h1>simple play application</h1>
{% endhighlight %}    

Running this app locally is very simple, just run the command below 
{% highlight bash %}    
play run simpleapp
{% endhighlight %}    

You can view your application at localhost:9000 in some web browser.

Now lets go to Redhat OpenShift PaaS, and create an account at [https://openshift.redhat.com](https://openshift.redhat.com). Once account is created and OpenShift client tools are setup, we can go for creating an application in OpenShift.

Basically app URL will be like 
{% highlight bash %}    
http://<application name>-<domain-name>.rhcloud.com
{% endhighlight %}    
, so create a domain and application. Domain creation is done by running

{% highlight bash %}    
rhc-create-domain -n cloudfundoo -l mail@service.com -p password
{% endhighlight %}    

After that create the application 

{% highlight bash %}    
rhc-create-app -l mail@service.com -p password -t jbossas-7 -a simpleapp
{% endhighlight %}    

On application creation it will create a directory with same application name, contents of directory will be

{% highlight bash %}    
#ls simpleapp
deployments pom.xml README src
{% endhighlight %}    

we will remove both pom.xml and src directory, which is not needed for our play app, remove from git tree also.

{% highlight bash %}    
#rm -rf pom.xml
#rm -rf src
#git add -A
#git commit -m "Remove the default code from git"
#git push origin
{% endhighlight %}    

Lets create a war for our simpleapp play framework application. Run this

{% highlight bash %}    
play war <directory of play simpleapp> 
 -o <directory of OpenShift simpleapp directory>/deployments/simpleapp.war
{% endhighlight %}    

Now create a simpleapp.war.dodeploy file to tell jboss to go ahead and deploy our app.

{% highlight bash %}    
#touch simpleapp/deployments/simpleapp.dodeploy
{% endhighlight %}    

Finally we need to push our play app's war to OpenShift app git repository. For that we need to run these.

{% highlight bash %}    
#cd simpleapp/deployments
#git add -A
#git commit -m "Checking in new play app war"
#git push origin
{% endhighlight %}    

Now time to see your live cloud app at [http://simpleapp-cloudfundoo.rhcloud.com/simpleapp/](http://simpleapp-cloudfundoo.rhcloud.com/simpleapp/)







