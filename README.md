docker-ubuntu-vnc-desktop
=========================

[![Docker Pulls](https://img.shields.io/docker/pulls/esraghu/docker-ubuntu-vnc-desktop.svg)](https://hub.docker.com/r/esraghu/docker-ubuntu-vnc-desktop/)
[![Docker Stars](https://img.shields.io/docker/stars/esraghu/docker-ubuntu-vnc-desktop.svg)](https://hub.docker.com/r/esraghu/docker-ubuntu-vnc-desktop/)
<a href="https://ko-fi.com/dorowu" target="_blank"><img src="https://az743702.vo.msecnd.net/cdn/kofi2.png" width="142" height="36"></a>

Docker image to provide a Virtual Box kind of functionality while having the ability to use the User Interface through a Browser, being able to install and run Code Editors and more importantly being able to run docker containers inside this. A HTML5 VNC interface to access Ubuntu 16.04 LXDE desktop environment has been provided.
If you are looking for a place where you can bring the latest Ubuntu with Desktop experience, this is it. You don't have to spend a lot of time downloading a ISO image, installing Virtual Box etc and then installing it. This comes up in matter of minutes. While this works in any platform, I've mainly been building it on Google Cloud Platform.

To run on Google Cloud Platform (GCP)
-------------------------------------
I have been trying to spin up a server in GCP which has a GUI with a good desktop experience, while also being able to install and run any programs I want, including other Docker containers. 
What you need to do to bring up this in a GCP VM is while creating a new instance: 
1. check the prompt 'Deploy a container image to this instance' and then paste `esraghu/docker-ubuntu-vnc-desktop`. 
2. I recommend you to run a minimum 1vCPU and 3.75GB RAM to get a decent experience and depending on the applications you're going to run, it's upto you to have a more vCPUs bigger RAM.
3. Click on Advanced container options and most definitely select `Run as Previliged` to be able to further run dockers inside it
4. Add Environment variables based for passing those variables - various variables have been mentioned below
5. You can also add mount points to keep data persistent and also add 
6. Under the firewall, select both HTTP and HTTPS as this will give you the ability to access desktop using both these protocols. All that you need to do is type in http://ipaddress or https://ipaddress (for https you will have to add environment variable SSL_PORT=443 and mount a location where you have ssl keys as /etc/nginx/ssl)
For further details on deploying containers through GCP, refer to the following link: https://cloud.google.com/compute/docs/containers/deploying-containers?hl=en_GB&_ga=2.77405397.-1992917830.1508901091


Quick Start
-------------------------

Run the docker container and access with port `6080`

```
docker run -p 6080:80 -v /dev/shm:/dev/shm dorowu/ubuntu-desktop-lxde-vnc
```

Browse http://127.0.0.1:6080/

<img src="https://raw.github.com/fcwu/docker-ubuntu-vnc-desktop/master/screenshots/lxde.png?v1" width=700/>

**Ubuntu Version**

Choose your favorite Ubuntu version with [tags](https://hub.docker.com/r/dorowu/ubuntu-desktop-lxde-vnc/tags/)

- bionic: Ubuntu 18.04 (latest)
- bionic-lxqt: Ubuntu 18.04 LXQt
- xenial: Ubuntu 16.04
- trusty: Ubuntu 14.04

VNC Viewer
------------------

Forward VNC service port 5900 to host by

```
docker run -p 6080:80 -p 5900:5900 -v /dev/shm:/dev/shm dorowu/ubuntu-desktop-lxde-vnc
```

Now, open the vnc viewer and connect to port 5900. If you would like to protect vnc service by password, set environment variable `VNC_PASSWORD`, for example

```
docker run -p 6080:80 -p 5900:5900 -e VNC_PASSWORD=mypassword -v /dev/shm:/dev/shm dorowu/ubuntu-desktop-lxde-vnc
```

A prompt will ask password either in the browser or vnc viewer.

HTTP Base Authentication
---------------------------

This image provides base access authentication of HTTP via `HTTP_PASSWORD`

```
docker run -p 6080:80 -e HTTP_PASSWORD=mypassword -v /dev/shm:/dev/shm dorowu/ubuntu-desktop-lxde-vnc
```

SSL
--------------------

To connect with SSL, generate self signed SSL certificate first if you don't have it

```
mkdir -p ssl
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ssl/nginx.key -out ssl/nginx.crt
```

Specify SSL port by `SSL_PORT`, certificate path to `/etc/nginx/ssl`, and forward it to 6081

```
docker run -p 6081:443 -e SSL_PORT=443 -v ${PWD}/ssl:/etc/nginx/ssl -v /dev/shm:/dev/shm dorowu/ubuntu-desktop-lxde-vnc
```

Screen Resolution
------------------

The Resolution of virtual desktop adapts browser window size when first connecting the server. You may choose a fixed resolution by passing `RESOLUTION` environment variable, for example

```
docker run -p 6080:80 -e RESOLUTION=1920x1080 -v /dev/shm:/dev/shm dorowu/ubuntu-desktop-lxde-vnc
```

Default Desktop User
--------------------

The default user is `root`. You may change the user and password respectively by `USER` and `PASSWORD` environment variable, for example,

```
docker run -p 6080:80 -e USER=doro -e PASSWORD=password -v /dev/shm:/dev/shm dorowu/ubuntu-desktop-lxde-vnc
```

Deploy to a subdirectory (relative url root)
--------------------------------------------

You may deploy this application to a subdirectory, for example `/some-prefix/`. You then can access application by `http://127.0.0.1:6080/some-prefix/`. This can be specified using the `RELATIVE_URL_ROOT` configuration option like this

```
docker run -p 6080:80 -e RELATIVE_URL_ROOT=some-prefix dorowu/ubuntu-desktop-lxde-vnc
```

NOTE: this variable should not have any leading and trailing splash (/)

Sound (Preview version and Linux only)
--------------------------------------

It only works in Linux. 

First of all, insert kernel module `snd-aloop` and specify `2` as the index of sound loop device

```
sudo modprobe snd-aloop index=2
```

Start the container

```
docker run -it --rm -p 6080:80 --device /dev/snd -e ALSADEV=hw:2,0 dorowu/ubuntu-desktop-lxde-vnc
```

where `--device /dev/snd -e ALSADEV=hw:2,0` means to grant sound device to container and set basic ASLA config to use card 2.

Launch a browser with URL http://127.0.0.1:6080/#/?video, where `video` means to start with video mode. Now you can start Chromium in start menu (Internet -> Chromium Web Browser Sound) and try to play some video.

Following is the screen capture of these operations. Turn on your sound at the end of video!

[![demo video](http://img.youtube.com/vi/Kv9FGClP1-k/0.jpg)](http://www.youtube.com/watch?v=Kv9FGClP1-k)


Generate Dockerfile from jinja template
-------------------

Dockerfile and configuration can be generated by template. 

- arch: one of `amd64` or `armhf`
- flavor: refer to file in flavor/`flavor`.yml
- image: base image
- desktop: desktop environment which is set in flavor
- addon_package: Debian package to be installed which is set in flavor

Dockerfile and configuration are re-generate if they do not exist. Or you may force to re-generate by removing them with the command `make clean`.

Troubleshooting and FAQ
==================

1. boot2docker connection issue, https://github.com/fcwu/docker-ubuntu-vnc-desktop/issues/2
2. Multi-language supports, https://github.com/fcwu/docker-ubuntu-vnc-desktop/issues/80
3. Autostart, https://github.com/fcwu/docker-ubuntu-vnc-desktop/issues/85
4. x11vnc arguments(multiptr), https://github.com/fcwu/docker-ubuntu-vnc-desktop/issues/101
5. firefox/chrome crash (/dev/shm), https://github.com/fcwu/docker-ubuntu-vnc-desktop/issues/112
6. resize display size without destroying container, https://github.com/fcwu/docker-ubuntu-vnc-desktop/issues/115#issuecomment-522426037

License
==================

See the LICENSE file for details.
