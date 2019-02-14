---
layout: post
title: 'Docker In Your HomeLab - Getting Started'
tags: [ 'Docker', 'Homelab', 'Containers' ]
---

I know I was intimidated at first, another tool hit the market that that's getting adoption like wildfire. There are an endless number of avenues to go down in IT, why should I consider this one? I found it was much simpler than I anticipated and the payoff was big. It took me a few days and paradigm changes to get to my final setup and configuration. Hopefully I can cut that timeline down and speed things up for you, to be able to get you on the best Docker solution path for your homelab environment by the time you're done reading this.

# Benefits Of Docker In The Homelab

*"Why would I do this anyway? Everything runs fine now the way it is."* The benefits to using Docker over traditional server administration methods are plentiful, most of which I didn't realize until I had Docker setup wrong. However, once right, I have only reaped the fruits of my labors. Let's talk about the benefits:

<!--more-->

* __Ease of Use__ - Once a container is setup and configured, seldom do need to worry about maintenance. Need to update a service? Docker will download the new image and swap out the old one with as little downtime as a restart. Did the update break and need to revert? Downgrading is quicker than updating with the previous image still saved on the host. Underlying disk failure? Install Docker on your new system, restore the configuration directory, run Docker Compose and you're back up and fully running in just a few minutes.
* __Speed and Efficiency__ - Get the segregation of virtualization without the resource waste of guest operating systems. Because there is only one operating system in the virtualization stack under Docker you’re not wasting CPU and memory running many containers. Thereafter I observed an increase in unused resources and a boost in performance of the service I run.
* __Dependency Freedom__ - A few services I run in my homelab like HomeAssistant and ZoneMinder have a lot dependencies. When updating I don't have to worry about cleaning up after them overtime. The core app and its decencies are all contained into a single image. Swapping images in and out is with all of their underlying packages / libraries is effortless.
* __Configuration Consolidation__ - Through the container parameters I can source the individual configurations and persistent data in a singular folder hierarchy. There are reasons you may not want to have all of that sitting in one place but for home use it makes backing up all of my services and their persistent data uncomplicated with a utility like [Borg](https://borgbackup.readthedocs.io/en/stable/).

# Getting Up And Running

## Underlying operating system

You're going to have to start with a base operating system to put Docker atop of. The hip kids these days seem to be using [Rancher OS](https://rancher.com/) a lightweight operating system built for running containers. [Ubuntu](https://www.ubuntu.com/) is always an easy go to, chances are you’re familiar with it and will be easy to setup. Personally I'm a vanilla [Debian](https://www.debian.org/) guy. It's simple, concise and the source for a lot of operating systems today. That speaks more to my personal and professional past. Choose what you're most comfortable with and will have the easiest time maintaining going forward.

## Components and pieces of it all

Under the covers Docker utilizes a few different configuration options. Below are the ones I found myself using the most and are probably going to be helpful for you.

* __Container Name__ - A name for the container. I keep them short and lowercase to for easy of reference in the CLI and GUI.
* __Image__ - The underlying source for a container. Images from the community in [Docker Hub](https://hub.docker.com/) are referenced by `username/imagename` unless its an official Docker image. Specific versions or releases are denoted with a *tag* or trailing colon `:rc` or `:0.86.1`. ie `robwolff3/ga-webserver` or `robwolff3/ga-webserver:0.2.0`
* __Restart Behavior__ - What it should do if the container halts or is stopped. I find myself almost always using `unless-stopped`, always restart the container regardless of the exit status unless it was stopped by me. `always` in the case of a management container that would be problematic if stopped running, it always restarts it even if I stop it.
* __Volumes__ - Used for mounting host paths or persistent volumes. I tend to only use them to mount configuration and service data stored on the host to the container. Using a folder hierarchy: `/home/user/docker/config/gawebserver:/config` everything is organized and condensed. If I blow away the container and recreate it, it'll pull the same configuration, data and be back up running.
* __Ports and Network Mode__ - Ports are exposed one by one, in ranges with and with protocols. Some containerized services don't play nice in the specified port paradigm, in those cases you can specify `network_mode: host` giving the container free reign to open whichever ports it would like. Be careful with that last one, you can't use host networking at the same time as specifying ports.
* __Environment Variables__ - How to pass variables into your container. Used heavily for configuring the containerized service. Pretty self explanatory.
* __Host Devices__ - For when a container needs access to host hardware. I've used them in the cases of USB Z-wave, USB Bluetooth, mic and speakers.
* __Depends On__ - If a container is dependent on another starting before it does, set a Depends On and they will come up in the correct order.

More on ports: When utilizing a single Docker host all of the running services will share the ip address assigned to that host. The best way to manage this with many services is to utilize a port schema i.e. 9000-9020. Ports exposed inside a container map to ports on the host. They do not have to match allowing a service that listens on port 80 to be exposed on the host as 9003. Further separation can be accomplished with a reverse proxy like [Nginx](https://hub.docker.com/r/linuxserver/letsencrypt/) to map services to domain names and more.

*Docker Run* utilizes command line switches for the parameters above and *Docker Compose* formats them in a yaml file that is executes into state, more on that later. For information on these parameters and more, reference the Docker documentation I link below, it's quite good and thorough.

## What can I containerize?

Probably more than you think. Try the applications site for documentation or for specific container instructions. If they don't have a sanctioned container try searching [Docker Hub](https://hub.docker.com/) for a community contributed variant. The Docker community has built out a myriad of containers for services otherwise neglected. There are some good resources including [Awesome-docker](https://awesome-docker.netlify.com/) out there making an index. In my homelab I'm running both official and community contributed images:

* [HomeAssistant](https://www.home-assistant.io/) - [homeassistant/home-assistant](https://hub.docker.com/r/homeassistant/home-assistant/)
* [Mosquitto](https://mosquitto.org/) - [eclipse-mosquitto](https://hub.docker.com/_/eclipse-mosquitto)
* [Node-RED](https://nodered.org/) - [nodered/node-red-docker:v8](https://hub.docker.com/r/nodered/node-red-docker/)
* [Plex](https://www.plex.tv/) - [plexinc/pms-docker](https://hub.docker.com/r/plexinc/pms-docker/)
* [Samba](https://www.samba.org/) - [dperson/samba](https://github.com/dperson/samba)
* [Nginx](https://www.nginx.com/) and [Let's Encrypt](https://letsencrypt.org/) - [linuxserver/letsencrypt](https://hub.docker.com/r/linuxserver/letsencrypt/)
* [MySQL](https://www.mysql.com/) - [mysql/mysql-server:5.7](https://hub.docker.com/r/mysql/mysql-server/)
* [ZoneMinder](https://zoneminder.com/) - [quantumobject/docker-zoneminder](https://hub.docker.com/r/quantumobject/docker-zoneminder/)

Some maintenance containers including:

* [Portainer](https://www.portainer.io/) - [portainer/portainer](https://hub.docker.com/r/portainer/portainer/) - A GUI interface to manage your Docker environment
* [Watchtower](https://github.com/v2tec/watchtower) - [v2tec/watchtower](https://hub.docker.com/r/v2tec/watchtower) - Automatically keep containers up to date with latest version

And other supporting services. Chances are you will find a way to run your service in Docker. If not, maybe once you're persuaded by Docker you can contribute an image of your own.

# The Different Methods

There were three different methods I progressed through in my journey to get Docker optimized in my homelab. It started with [DockSTARTer](https://dockstarter.com/). I realized pretty quickly it did not give me the level of control I desired coming from a systems administrations background. I then moved everything over to Docker Run commands. It sounded like a smart idea at the time but when you hear about Docker Compose it should be a no brainer.

## DockSTARTer

<img src="/assets/getting-started-with-docker/dockstarter.png" alt="DockSTARTer" width="800">

A great utility that will kickstart you into the world of Docker fast. In just a few clicks the utility takes care of the install of Docker, setup and configuration of the containers in a managed Docker Compose for you behind the scenes. It take some of the control away but little investment is needed on your part.

This makes it great to test out Docker if you're still unsure. Or if you're not really keen on the components details I've described above and just want it to "work" then DockSTARTer may be the approach for you. One drawback to DockSTARTer is that it only supports a limited set of containers. Check that they support the ones you intend to run first before diving in.

[Learn more about DockSTARTer and how to install it at their site here.](https://dockstarter.com/)

## Docker Run

```bash
$ docker run -d --name=gawebserver \
    --restart on-failure \
    -v /home/user/docker/config/gawebserver:/config \
    -p 9324:9324 \
    -p 5000:5000 \
    -e CLIENT_SECRET=client_secret.json \
    -e DEVICE_MODEL_ID=device_model_id \
    -e PROJECT_ID=project_id \
    --device /dev/snd:/dev/snd:rwm \
    robwolff3/ga-webserver
```

Above is an example of a Docker Run command in bash. I figured out that I could use these one off commands to run containers not supported by DockSTARTer. Once running then using the standard Docker commands to administer them `docker [ pull start stop restart exec logs ps ]`. Later I ended up with a list of all the Docker Run commands I could execute to get my environment up and running saved in a file. Separating myself from DockSTARTer and giving me more parameter control. After moving over to Docker Compose I believe Docker Run is still useful for running one off containers or testing.

[You can reference the Docker Run documentation here.](https://docs.docker.com/engine/reference/run/)

## Docker Compose - __*Recommended Method*__

```yaml
version: "3.4"
services:

  gawebserver:
    container_name: gawebserver
    image: robwolff3/ga-webserver
    restart: on-failure
    volumes:
      - /home/user/docker/config/gawebserver:/config
    ports:
      - 9324:9324
      - 5000:5000
    environment:
      - CLIENT_SECRET=client_secret.json
      - DEVICE_MODEL_ID=device_model_id
      - PROJECT_ID=project_id
    devices:
      - "/dev/snd:/dev/snd:rwm"
```

*"Sometimes you have to do it wrong a few times before you learn enough to know what the right way is. Or just read the docs, yeah, that would definitely be easier."* This is the an example of a truncated Docker Compose file showing a single service. Additional services would be listed below it, 13 in my case at the time of writing.

All of the Docker components are structured in a yaml file. To implement the compose file execute `docker-compose up -d` in the directory of the file. Docker will then apply that config and getting it running to that state. Any time Docker Compose is run after, Docker will apply whatever state is specified in the compose file, even if it requires recreating the container. I would recommend this method if you want granular control and to take advantage of the idempotency it has to offer.

[You can reference the Docker Compose documentation here.](https://docs.docker.com/compose/compose-file/)

# Maintaining Your Container Environment

## Helpful Commands - Docker Compose

These are the commands I use the most to administer my Docker environment. Using the service HomeAssistant as an example:

### The Basics

* Starting a container: `$ docker start homeassistant`
* Stopping a container: `$ docker stop homeassistant`
* Restarting a container: `$ docker restart homeassistant`
* Listing the running containers: `$ docker ps`
* View the logs of a container: `$ docker logs -f homeassistant`
* Drop a shell into a container: `$ docker exec -it homeassistant /bin/bash`


### Bringing all of the services up or back to state

```bash
$ cd /home/user/docker/compose && \
    docker-compose up -d
```

### Updating a specific container

```bash
$ docker pull homeassistant/home-assistant && \
    cd /home/user/docker/compose && \
    docker-compose up -d
```

### Downgrading a container

Edit the image tag of the container in your Docker Compose file to reflect the version you want running i.e. `image: homeassistant/home-assistant:0.86.4` and then execute `docker-compose up -d`

## Portainer - End Game

<img src="/assets/getting-started-with-docker/portainer.png" alt="Portainer" width="1000">

I wanted to add a quick note about Portainer. At first I wasn't sure how useful it was going to be but turned out to be really handy. It's great for all of the management tasks I mentioned above but within a web based GUI. When you need to do a quick thing but don't have a terminal up or like to view your Docker environment visually this tool is great.

[Check out their site to learn more and how to install it here.](https://www.portainer.io/)

# Conclusion

Before I started this endeavor I was running HomeAssistant, Mosquitto, Samba and some other services on a Raspberry Pi and was never really thrilled with the performance. Another computer I had was running Plex, ZoneMinder and others as well, everything kind of a mess. Now I have all of my resources consolidated on a single machine, services running off of SSD, archive data on mechanical disk with actual running backups. CPU and memory are way better utilized and shared under Docker than individual guest operating systems. All services running better and easier to manage going forward.

I'm sure I've only scratched the surface of what Docker is capable of. Hopefully my experiences can help you better manage and run your homelab. It would really be a missed opportunity to pass it up.

---

## Further Reading

* [Docker - Get Started](https://docs.docker.com/get-started/)
* [Docker Run](https://docs.docker.com/engine/reference/run/)
* [Docker Compose](https://docs.docker.com/compose/compose-file/)
* [freeCodeCamp - A Beginner-Friendly Introduction to Containers, VMs and Docker](https://medium.freecodecamp.org/a-beginner-friendly-introduction-to-containers-vms-and-docker-79a9e3e119b)
* [Digital Ocean - How To Install and Use Docker on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04)
