# docker build -t $USER/smartpm .
# xhost +
# xhost + is stupid and dangerous, so please run 'xhost -' after launching apps 
# docker run -v /tmp/.X11-unix/X0:/tmp/.X11-unix/X0 -i -t $USER/smartpm /bin/bash
# export DISPLAY=0.0
# smart update
# smart --gui
# View > Tree Style > Channels & Groups
# docker export $USER/smartpm | gzip -c > /media/lenovo_mandeep/smartpm.tgz
# docker import $USER/smartpm < /media/lenovo_mandeep/smartpm.tgz
# echo 'DOCKER_OPTS="-H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock"' \
# | sudo tee -a /etc/default/docker 

FROM ubuntu:12.04
MAINTAINER David Hart "dhart@opencog.org"

RUN echo "deb http://hk.archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list && \
    echo "deb http://hk.archive.ubuntu.com/ubuntu precise-updates main universe" >> /etc/apt/sources.list && \
    apt-get -y update && \
    apt-get -y install python-software-properties wget less vim-gnome smartpm
