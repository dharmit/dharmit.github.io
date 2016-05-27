---
layout: post
title: Get rid of using sudo when using Vagrant with libvirt provider
categories:
- blog
---

If you're using Vagrant on Fedora box then chances are you use `sudo` to bring
up the box every time. I could see two problems with that:

1. It asks for password.
2. If you have pulled the box as normal user and use `sudo` to bring up the
   box, the box gets re-downloaded, now as privileged user. This, unless you've
   copied the box yourself to the home directory of privileged user.

To get rid of this is very trivial and the documentation for [Container
Development Kit on Red Hat Customer
Portal](https://access.redhat.com/documentation/en/red-hat-enterprise-linux-atomic-host/version-7/container-development-kit-installation-guide/#fedora-soft-install) explains this very nicely!

To put it even more shortly, all you need to do is:

~~~
$ sudo dnf install vagrant-libvirt-doc
$ sudo cp /usr/share/vagrant/gems/doc/vagrant-libvirt-0.0.30/polkit/10-vagrant-libvirt.rules
/etc/polkit-1/rules.d
$ sudo systemctl restart libvirtd
$ sudo systemctl restart polkit
~~~

The path under `/usr/share/vagrant/gems/doc/` is dependent on the
verion of `vagrant-libvirt-doc` installed on your system. 

That's it. Now you can play with Vagrant commands without having to bother
going the `sudo` route. :)
