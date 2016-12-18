---
layout: post
title:  "Setting up a FreeBSD Droplet with Ansible"
date:   2015-06-15 23:00:00
categories: freebsd,server,devops,ansible,python
---
A few months ago I moved my private server over to [DigitalOcean](https://www.digitalocean.com). Coming from OpenBSD, I gave DigitalOcean's FreeBSD option a try. However, this time I wanted to do it right: essential services were to be provisioned properly. I never want to configure a mail server from scratch again.

That's where [Ansible](http://docs.ansible.com) comes in. While I [played around with Puppet](https://github.com/abits/blueprints) before, Ansible convinced me, since it works over simple SSH and doesn't need any dedicated client pull. However, DigitalOcean's FreeBSD image doesn't bring Python out of the box, leaving us in a catch-22. There is a nice Ansible module for setting up a FreeBSD droplet over DigitalOcean's API.  But once this is done, there's nothing that could run Ansible's tasks on the new host. Too bad.

The trick is to use Ansible's `raw` module. It allows you to run any command on the target host's shell, which means you can install Python:

{% highlight bash %}
# task for installing Python on a host via raw
- name: install Python
  raw: env ASSUME_ALWAYS_YES=YES pkg install Python
{% endhighlight %}

This Ansible task will install Python on a FreeBSD droplet.

There's another gotcha: You will need to pass the path to the Python interpreter when you store the host's IP in-memory. I did this with the following tasks:

{% highlight yaml %}
---
- name: Create new droplet
  digital_ocean: state=present
                 command=droplet
                 name="{{ droplet_name }}"
                 size_id="{{ droplet_size_id }}"
                 region_id="{{ droplet_region_id }}"
                 image_id="{{ droplet_image_id }}"
                 ssh_key_ids="{{ ssh_pub_key }}"
                 wait_timeout=600
                 api_key="{{ do_api_key }}"
                 client_id="{{ do_client_id }}"
                 wait=yes
  register: do_droplet

- name: Add droplet ip to inventory
  add_host: hostname="{{ do_droplet.droplet.ip_address }}"
            groups=newdroplets
            ansible_python_interpreter=/usr/local/bin/Python

- name: wait a little
  pause: minutes=1 prompt="wait a little"

{% endhighlight %}

I let the process wait a minute to make sure that the SSH daemon on the droplet is started before I start provisioning.

Finally you'll need to do everything in the right order: create droplet, install Python, do stuff on target host. I use a playbook along these lines:

{% highlight yaml %}
- hosts: localhost
  roles:
    - digitalocean

- hosts: newdroplets
  user: freebsd
  sudo: yes
  gather_facts: no
  roles:
    - Python

- hosts: newdroplets
  user: freebsd
  sudo: yes
  roles:
      - common
      - user
      - mail
      - gitolite
{% endhighlight %}

Some minor traps: do not gather facts before installing Python (as this needs Python already, catch-22, remember?). Also, you need to run tasks as user `freebsd`, since this is Ansible's default sudoer in FreeBSD.

If you are interested, you can watch [the whole show here](https://github.com/abits/ansible-freebsd). Hopefully, there is more to come, like documentation, IPv6 and a proper web server config. Finally I can spawn a working mail server in a matter of minutes!
