---
layout: post
title: Deploying Applications on your Docker JBoss EAP Image
modified: 2016-03-22
tags: [DevOps, Red Hat, Docker, RHEL, Cloud]
comments: true
---


Docker is a container based software framework for automating deployment of applications. Docker enables developers and sysadmins to build, ship and run distributed applications anywhere. You can think of Docker
containers as sort-of lightweight virtual machines and can run on almost any platform.

Docker have been released as part of the extras channel in Red Hat Enterprise Linux 7. The steps for installing  are generally as simple. Install it using the following command:

{% highlight bash %}
[root@mmagnani ~]# yum -y install docker
{% endhighlight %}

Now you have Docker installed onto your machine, start and enable the Docker service incase if it not started automatically after the installation

{% highlight bash %}
[root@mmagnani ~]# systemctl start docker.service
[root@mmagnani ~]# systemctl enable docker.service
{% endhighlight %}

Once the service is started, verify your installation by running the following command:

{% highlight bash %}
[root@mmagnani ~]# docker run -it rhel echo Hello-World
{% endhighlight %}

This command will look for an image named "rhel" on your local machine, and in its absence, try to download it from the registry.access.redhat.com. The "registry.access.redhat.com" is a central repository of Docker images(you must have a valid Redhat subscription).


## Creating a Docker container for JBoss EAP 6.4

Now that we have a basic understanding of what Docker we will learn how to deploy applications by using Docker Files. Dockerfile has a special mission: automation of Docker image creation. A Dockerfile consists of a set of commands which can be written in a text file named "Dockerfile". The purpose of our Docker file will be adding an application named "myapp.war" in the deployments folder of the standalone installation. Now lets look at the Dockerfile:

{% highlight bash %}
# dockerfile to build image for JBoss EAP 6.4

# start from rhel 7.2
FROM rhel

# file author / maintainer
MAINTAINER "Mauricio Magnani" "mmagnani@redhat.com"

# update OS
RUN yum -y update && \
    yum -y install sudo openssh-clients telnet unzip java-1.8.0-openjdk-devel  && \
    yum clean all

# enabling sudo group
# enabling sudo over ssh
RUN echo '%wheel ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers && \
    sed -i 's/.*requiretty$/Defaults !requiretty/' /etc/sudoers

# add a user for the application, with sudo permissions
RUN useradd -m jboss ; echo jboss: | chpasswd ; usermod -a -G wheel jboss

# create workdir
RUN mkdir -p /opt/rh

WORKDIR /opt/rh

# install JBoss EAP 6.4.0
ADD jboss-eap-6.4.0.zip /tmp/jboss-eap-6.4.0.zip
RUN unzip /tmp/jboss-eap-6.4.0.zip

# set environment
ENV JBOSS_HOME /opt/rh/jboss-eap-6.4

# create JBoss console user
RUN $JBOSS_HOME/bin/add-user.sh admin admin@2016 --silent

# configure JBoss
RUN echo "JAVA_OPTS=\"\$JAVA_OPTS -Djboss.bind.address=0.0.0.0 -Djboss.bind.address.management=0.0.0.0\"" >> $JBOSS_HOME/bin/standalone.conf

# set permission folder 
RUN chown -R jboss:jboss /opt/rh

# JBoss ports
EXPOSE 8080 9990 9999

# start JBoss
ENTRYPOINT $JBOSS_HOME/bin/standalone.sh -c standalone-full-ha.xml

# deploy app
ADD myapp.war "$JBOSS_HOME/standalone/deployments/"

USER jboss
CMD  /bin/bash
{% endhighlight %}


Make sure that the myapp.war application and jboss-eap-6.4.0.zip  are in the same folder as the Dockerfile:

{% highlight bash %}
[root@jboss ~]# ls 
Dockerfile  jboss-eap-6.4.0.zip  myapp.war
{% endhighlight %}

If you do not already have the ZIP archive for JBoss EAP 6.4 then you should go download it here https://access.redhat.com/downloads.


## Building the container

To build a new Docker image is simple, you have to choose a tag and issue a docker build command:

{% highlight bash %}
[root@mmagnani ~]# docker build -q --rm --tag=jboss-myapp .
{% endhighlight %}

Now check images by issuing a docker command:

{% highlight bash %}
[root@mmagnani ~]# docker images
REPOSITORY                                                  TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
jboss-myapp                                                 latest              bd14a8aad563        2 days ago          912.7 MB
{% endhighlight %}

Once the image has completed building, you will want to run it.

{% highlight bash %}
[root@mmagnani ~]# docker run -it jboss-myapp

22:23:51,316 INFO  [org.xnio] (MSC service thread 1-8) XNIO Version 3.0.13.GA-redhat-1
22:23:51,342 INFO  [org.xnio.nio] (MSC service thread 1-8) XNIO NIO Implementation Version 3.0.13.GA-redhat-1
22:23:51,353 INFO  [org.jboss.as.server] (Controller Boot Thread) JBAS015888: Creating http management service using socket-binding (management-http)
22:23:51,416 INFO  [org.jboss.remoting] (MSC service thread 1-8) JBoss Remoting version 3.3.4.Final-redhat
{% endhighlight %}

The application server will start.

Now we need to find the IP address which has been chosen by Docker to bind the application server:

{% highlight bash %}
[root@mmagnani ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                          NAMES
65009707e8b4        jboss-myapp         "/bin/sh -c '$JBOSS_H"   17 seconds ago      Up 13 seconds       8080/tcp, 9990/tcp, 9999/tcp   clever_jones
{% endhighlight %}

{% highlight bash %}
[root@mmagnani ~]# docker inspect -f '{{ .NetworkSettings.IPAddress }}' 65009707e8b4
172.17.0.2
{% endhighlight %}

Your application is ready on address [http://172.17.0.2:8080/myapp](http://172.17.0.2:8080/myapp).

References: https://goldmann.pl/blog/2014/07/23/customizing-the-configuration-of-the-wildfly-docker-image