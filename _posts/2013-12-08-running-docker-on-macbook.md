---
layout: post
title: "Running docker on macbook"
description: "How to run docker in vagrant and expose it to macbook"
category: "docker"
tags: [docker, apache, macbook, vagrant]
---
{% include JB/setup %}


## TL;DR

Run docker using

{% highlight bash %}
sudo docker run -p 0.0.0.0:49022:22 -p 0.0.0.0:49080:80 -t -i asood/supervisord
{% endhighlight %}


to redirect port 80 of docker to port 49080 on vagrant. And use

{% highlight bash %}
config.vm.forward_port 49080, 49080
{% endhighlight %}

in VagrantFile to expose the vagrant port to local machine.


### Use case

Docker blog has a comprehensive list of what can be done with docker and the docker twitter feed is an excellent source
 of how various folks are using docker for their use cases.

 Working in my last project, one of the thing that was clear from the start was to have a distributed environment, i.e.
 it should be possible to run messaging service in one box, database in another, and multiple instances of code in several
 boxes on web servers and each of these should be able to communicate with each other.

 The biggest problem with this setup is making sure that every person in the team is able to have the same topology setup
  fairly quick and repeatedly with ease. Also this ensures that from day 0, the entire stack is tested in the same fashion
  as it would be running in production.

 And this is where docker comes into picture. The concept of making container a unit of deployment makes it easy to have
  this different units of deployments as individual containers which can be repeatedly and easily setup using docker file.

#### Using the docker example to setup apache webserver using docker

I referred to the example [here] (http://docs.docker.io/en/latest/examples/using_supervisord/) for setting up apache webserver
using supervisor. I like the supervisor pattern because it lets you control the dependencies on what should be started
before apache webserver actually gets started.

This makes more sense when you are aiming for a more complicated setup like what is mentioned in this [blog] (http://blog.relateiq.com/a-docker-dev-environment-in-24-hours-part-1-of-2/)


### Gotchas

Most of it was really easy to setup, and here is the docker file that I ended up creating is present [here] (https://github.com/atinsood/dockerFiles/tree/master/apache2_supervisor)

The only problem was that I needed to expose my 80 port on which the apache webserver was running to my native macbook.
Note that the docker is running inside virtual machine on vagrant. So, I need to expose port 80 from docker to my vagrant
box and then expose that port to my macbook.

{% highlight bash %}
EXPOSE 80 22
{% endhighlight %}

exposes port 80 and 22

{% highlight bash %}
sudo docker run -p 0.0.0.0:49022:22 -p 0.0.0.0:49080:80 -t -i asood/supervisord
{% endhighlight %}

exposes port 80 to vagrant box on port 49080 of vagrant.

Now to expose 49080 port of vagrant to macbook, add the following in the vagrant file

{% highlight bash %}
  config.vm.forward_port 49080, 49080
{% endhighlight %}

so that the final config on the file looks something like

{% highlight bash %}
Vagrant::Config.run do |config|
  # Setup virtual machine box. This VM configuration code is always executed.
  config.vm.box = BOX_NAME
  config.vm.box_url = BOX_URI
  config.vm.forward_port 49080, 49080
{% endhighlight %}

That should do. Now you should be able to run http://localhost:49080 from your macbook's browser.