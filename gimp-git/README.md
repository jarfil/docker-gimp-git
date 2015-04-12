Latest version of Gimp compiled from git master.

Based on Ubuntu Vivid

## Usage

Local X11 server
> `docker run -it --rm -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix jarfil/gimp-git`

Remote SSH into docker host, low isolation

> `docker run -it --rm --net=host -e DISPLAY=$DISPLAY -v $HOME:/hosthome:ro -e XAUTHORITY=/hosthome/.Xauthority jarfil/gimp-git`

You can also install sshd onto the image and ssh directly into it.

## Maintenance

To automate the image building process with reasonable throttle enforcement, there is a [web trigger](http://jarfil.net/dockerfiles.auto/autobuild_gimp-git.php) available.

![graph](https://jarfil.github.io/dockerfiles/data/gimp-git/docker%20automate%2003.png)