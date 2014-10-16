XPaaS haproxy Docker Image
=======================

This project builds a [docker](http://docker.io/) container for running XPaaS based applications.

This image is based on a <code>fedora</code> version <code>20</code> and provides a container including:     
* Supervisor daemon     
* Git client      
* Python PIP      
* Telnet client   
* Net tools       
* Open SSH server & client     
* wget tool     
* Java 1.7     
* Unzip tool    
* Maven 3.0.5 with jboss repositories configured    

Table of contents
------------------

* **[Control scripts](#control-scripts)**
* **[Building the docker container](#building-the-docker-container)**
* **[Running the container](#running-the-container)**
* **[Connection to a container using SSH](#connection-to-a-container-using-SSH)**
* **[Starting, stopping and restarting the SSH daemon](#starting,-stopping-and-restarting-the-SSH-daemon)**
* **[Logging](#logging)**
* **[Stopping the container](#stopping-the-container)**
* **[Experimenting](#Experimenting)**
* **[Notes](#notes)**

Control scripts
---------------

There are three control scripts:    
* <code>build.sh</code> Builds the docker image    
* <code>start.sh</code> Starts a new XPaaS base  docker container based on this image    
* <code>stop.sh</code>  Stops the runned XPaaS base  docker container    

Building the docker container
-----------------------------

We have a Docker Index trusted build setup to automatically rebuild the xpass-base/xpass-base container whenever the
[Dockerfile](https://github.com/pzapataf/xpaas-docker-containers/blob/master/xpaas-base-docker/Dockerfile) is updated, so you shouldn't have to rebuild it locally. But if you want to, here's now to do it...

Once you have [installed docker](https://www.docker.io/gettingstarted/#h_installation) you should be able to create the containers via the following:

If you are on OS X then see [How to use Docker on OS X](DockerOnOSX.md).

    git clone git@github.com:jboss-xpaas/docker-containers.git
    cd xpaas-docker-containers/xpaas-base-docker
    ./build.sh

Running the container
---------------------

To run a new container from XPaaS base run:
    
    ./start.sh [-c <container_name>] [-p <root_password>]


Or you can try it out via docker command directly:

    docker run -P -d [--name <container_name>] [-e ROOT_PASSWORD="<root_password>"] xpaas/xpaas_base:<version>

These commands will start a new XPaas Base container with SSH daemon enabed.     

**Environment variables**

These are the environment variables supported when running the JBoss Wildfly/EAP container:       

- <code>ROOT_PASSWORD</code> - The root password for <code>root</code> system user. Useful to connect via SSH

**Notes**           
* If no container name argument is set, it defaults to <code>xpaas-base</code>       
* If no root password argument is set, it defaults to <code>xpaas</code>    

Connection to a container using SSH
-----------------------------------

When running a new container over this docker image, the SSH daemon is started by default and waiting for connections.     

In order to connect to the container using SSH you must know the container binding SSH port. If you type:

    docker ps
    
you should see the port mappings for each docker container. For example you may see something like this in the PORTS section....

    0.0.0.0:49001->22/tcp
    
This means that from outside the docker container; you need to use port 49001 to access port 22 inside the container. Note this number changes for each container; outside of each docker container there are different ports that forward to the 22 port.     

So if the port number is 49001 then you can type something like this:

    ssh root@localhost -p 49001
    
**Notes**        
* By default, the only available user to connect using SSH is <code>root</code>     
* By default, the <code>root</code> user password is <code>xpaas</code>     
* You can change the default root password when running the container. See **[Running the container](#running-the-container)**      

Starting, stopping and restarting the SSH daemon
------------------------------------------------

To start the SSH daemon run:
    
    ssh root@localhost -p <ssh_port> sh /jboss/scripts/start.sh sshd

To stop the SSH daemon run:
    
    ssh root@localhost -p <ssh_port> sh /jboss/scripts/stop.sh sshd

To restart the SSH daemon run:
    
    ssh root@localhost -p <ssh_port> sh /jboss/scripts/restart.sh sshd

Logging
-------
You can see all logs generated by supervisor daemon & programs by running:

    docker logs [-f] <container_id>
    
You can see only the SSH daemon logs by running this command:

    ssh root@localhost -p <ssh_port> tail -f /var/log/supervisord/sshd-stdout.log
    ssh root@localhost -p <ssh_port> tail -f /var/log/supervisord/sshd-stderr.log

Stopping the container
----------------------
To stop the previous container run using <code>start.sh</code> script just type:

    ./stop.sh

Experimenting
-------------
To spin up a shell in one of the containers try:

    docker run -P -i -t xpaas/xpaas_base /bin/bash
    
You can then noodle around the container and run stuff & look at files etc.

In order to run all container services provided by this image, you have to run the following command:

    supervisord -c /etc/supervisord.conf

Notes
-----

* The default daemon enabled with the container is <code>supervisor</code>.The other daemon and program executions are handled by the supervisor daemon     
* This container publishes the following environment variables:     
<table>
    <tr>
        <td><b>Name</b></td>
        <td><b>Value</b></td>
    </tr>
    <tr>
        <td>JAVA_HOME</td>
        <td>usr/lib/jvm/jre</td>
    </tr>
    <tr>
        <td>M2_HOME</td>
        <td>/opt/apache-maven</td>
    </tr>
    <tr>
        <td>MAVEN_HOME</td>
        <td>/opt/apache-maven</td>
    </tr>
    <tr>
        <td>M2</td>
        <td>/opt/apache-maven/bin</td>
    </tr>
</table>