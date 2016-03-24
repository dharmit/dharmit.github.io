---
layout: post
title: Using Atomic Developer Bundle (ADB) to setup Mesos-Marathon based development environment
categories:
- blog
---

Since 1.6.0 release, the [Atomic Developer Bundle
(ADB)](https://github.com/projectatomic/adb-atomic-developer-bundle) 
includes support for Mesos-Marathon as an orchestrator. This is in conjunction
with the support for Mesos-Marathon that was added to
[AtomicApp](https://github.com/projectatomic/atomicapp) 0.3.0. This feature
supports developers choosing to work with atomicapps on Mesos-Marathon.

Mesos Marathon is a distributed control system which can be used for container
orchestration in large server clusters.  Many Docker-based infrastructure teams
use Mesos instead of Kubernetes. Learn more here:

- [Mesos](http://mesos.apache.org/)
- [Marathon](http://mesosphere.github.io/marathon/)

A [Vagrantfile](https://github.com/projectatomic/adb-atomic-developer-bundle/blob/master/components/centos/centos-mesos-marathon-singlenode-setup/Vagrantfile)
has been created that will use the ADB and provision it with a fully setup and
configured single node Mesos-Marathon installation.

One can either clone the ADB repo to use this setup or download only the
Vagrantfile and Ansible playbook. To avoid any errors, it'd be best to clone
the repo and spin up the vagrant box:

~~~
$ git clone https://github.com/projectatomic/adb-atomic-developer-bundle/
$ cd adb-atomic-developer-bundle/components/centos/centos-mesos-marathon-singlenode-setup/
$ vagrant up
~~~

If you need help with setting up Vagrant or a virtualization provider, refer
[this
document](https://github.com/projectatomic/adb-atomic-developer-bundle/blob/master/docs/installing.rst).

Note that when you do `vagrant up`, the [Ansible
playbook](https://github.com/projectatomic/adb-atomic-developer-bundle/blob/master/components/centos/centos-mesos-marathon-singlenode-setup/provisioning/playbook.yml),
used to do the provisioning, downloads certain packages from the Internet.
Hence, the time for box to get ready for use is directly dependent on your
Internet connection. It pulls around 130 MB from the Internet.

Please provide feedback about your experience with the process of setting up
Mesos-Marathon provider on ADB. You can do this by engaging with developrs on
#nulecule IRC channel on Freenode or open an issue under [ADB repo on
github](https://github.com/projectatomic/adb-atomic-developer-bundle).
