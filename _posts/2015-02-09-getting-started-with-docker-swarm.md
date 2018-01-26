---
layout: post
title: "Getting Started with Docker Swarm"
---

![docker-swarm](/images/posts/docker-swarm.jpg)

I’ve recently had the chance to play around with Docker’s new clustering tool – Docker Swarm, and I like it. Since the tool is still in a pre-alpha things are changing quickly and it can be a bit difficult to figure out how to get started, so I’m sharing my notes for anyone else interested in playing with Docker Swarm. Please note that while this is meant to get someone up-and-running it is by no means a deep dive into the capabilities of Swarm and I highly recommend that you study their documentation at [https://github.com/docker/swarm](https://github.com/docker/swarm).

### DEFINITIONS
First I’m going to quickly define a couple of terms that I make use of this article.
- **Docker Node** – a machine running the Docker daemon
- **Swarm Host** – a machine running the Swarm daemon
- **Swarm** – a series of Docker nodes

### WHAT IS DOCKER SWARM?
Docker Swarm is a native clustering system for Docker. It allows you to define your cluster (swarm) and create/control docker images and containers throughout the cluster through the Docker Swarm daemon.

### WHY USE DOCKER SWARM?
Docker swarm is a relatively simple tool for optimizing your container workloads across your cluster while using the standard docker commands you’re already familiar with.

### OK, HOW DO WE USE DOCKER SWARM?
**Basic Infrastructure**  
I’m sure there’s many ways to set this up, here’s how I did it: 1 Swarm Host and 2 Docker Nodes.

**SWARM HOST**  
The only requirements for a Swarm host is that it has Docker Swarm installed.

I started with a t2.micro instance on AWS running Ubuntu 14.04. From there you’ll need to install Docker Swarm and since it’s still pre-beta they’re not offering binaries yet.

**Install Docker Swarm**  
`$ sudo apt-get install golang git`  
`$ mkdir ~/gocode; export GOPATH=~/gocode`  
`$ cd ~/gocode`  
`$ go get -u github.com/docker/swarm`  

**Test Docker Swarm**  
`$ ~/gocode/bin/swarm --help`

**DOCKER NODE**  
Any node in the swarm requires Docker to be installed and the Docker daemon to be running and bound to a tcp socket, which is how the Swarm Host communicates with the nodes.

For my project I started with two t2.micro [Project Atomic](http://www.projectatomic.io/) Fedora ami instances, you can use any node you’d like so long as the following steps are taken.

**Install Docker**  
Docker is changing very quickly, I highly recommend installing the latest version directly [from them](https://docs.docker.com/installation/).

**Bind the Docker daemon to a tcp port**  
You can manually add the `-H` flag every time you run the daemon or you can edit your Docker settings so that it runs with it by default, I chose the latter:  
`# vi /etc/sysconfig/docker`  
Then add `-H 0.0.0.0:2375` to OPTIONS

Reload the daemon so the new settings take effect and then restart it  
`# systemctl daemon-reload`  
`# systemctl start docker`  

If you don’t want to edit your default Docker behavior then just make sure that you start your Docker daemon on every node with the following flag and options: `-H 0.0.0.0:2375`

### Putting it all Together
Now that you have the infrastructure in place it’s time to put it all together.  
_All of the following commands will be issued from the Swarm Host_

**Create your swarm**  
`$ ~/gocode/bin/swarm create`  

This command will spit out your `swarm_id` that looks something like this: `2a0e3b11ce210ae859a141b473abdd34`. Don’t lose it, this is the token used to identify your swarm.

**Add nodes to your swarm**  
`$ ~/gocode/bin/swarm join --addr=<ip-of-node>:2375 token://<swarm_id>`

Issue the above command for all nodes that you want to add to the swarm, change the ip accordingly but obviously use the same swarm id.

Note that this command does not currently return so once the IP has been added to the swarm, you can ctrl-c out. The join command does not need to keep running to stay joined.

**List the nodes in your swarm**  
`$ ~/gocode/bin/swarm list token://<swarm_id>`

**Start the Swarm manager (daemon)**  
`$ ~/gocode/bin/swarm -debug manage --host=0.0.0.0:2375 token://<swarm_id>`

I like running with the -debug flag for more info.

Once this command is running the Swarm manager daemon will continue to listen for incoming Docker commands to execute on your swarm. It needs to stay running for Swarm to work.

**The MAGIC of Docker Swarm**  
Now that everything is in place, let’s see what we can do with it.

Log into a new machine, any machine, that has Docker installed and is able to access the IP of your Swarm Host.

**Launch a Docker container**  
`$ docker -H <ip-of-swarm-host>:2375 run blackfinsecurity/tha-kali`

This command is telling docker to connect to the Swarm host on port 2375, the Swarm daemon then relays a standard docker run command.

Since Swarm is listening it will receive the command, do some calculations about the status of your nodes in the swarm, and then issue the docker run command on the node. That’s right, you can now issue standard Docker commands via your Swarm host and they will be issued throughout your swarm.

### FINAL THOUGHTS
Swarm offers a decent amount of power in controlling how it places containers within your swarm. For further information on how it works and the settings available you should check out their documentation on [scheduler filters](https://github.com/docker/swarm/tree/master/scheduler/filter) and [scheduler strategies](https://github.com/docker/swarm/tree/master/scheduler/strategy).

Another thing to keep in mind is that if you followed this guide you have 3 AWS instances that are visible to the internet and anyone with their ip and port  can issue docker commands to them. Needless to say that’s insecure and I wouldn’t recommend leaving them running when you’re not using them unless you’ve taken steps to secure them.

**Continue reading part 2 of this series – [Docker Swarm wit TLS authentication](/blog/2015/docker-swarm-tls-auth/)**
