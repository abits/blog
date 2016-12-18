---
layout: post
title:  "FreeBSD service for aria2 download manager"
date:   2016-02-03 22:05:00
categories: freebsd,server,admin,utils
---
I have to confess. I am falling in love with FreeBSD. It is so darn simple to set up and control a custom service, that I am going to cry when I remember systemd.  I am using [aria2](https://github.com/tatsuhiro-t/aria2) as a download manager on my local server, which I control via RPC. Recently I discovered the beautiful [webui-aria2](https://github.com/ziahamza/webui-aria2), which is a browser
application connecting to aria2. I figured it might be nice to have a custom service, so that aria2 can be launched during boot. Apparently, this is all
you have to do:
{% highlight bash %}
#!/bin/sh
. /etc/rc.subr
name=aria2d
rcvar=aria2d_enable
CONF=/usr/local/etc/aria2/aria2.conf
command="/usr/local/bin/aria2c"
command_args="--conf-path=${CONF} --enable-rpc --rpc-listen-all --daemon"
load_rc_config $name
run_rc_command "$1"
{% endhighlight %}
Put this under `/usr/local/etc/rc.d/aria2d` and you can work all the magic like
`/usr/local/etc/rc.d/aria2d onerestart`. Eat this `systemctl`!