---
layout: post
title: Getting started with Ansible
categories:
- blog
---

Lately I have been working as a DevOps Engineer. I am exploring Mesos, Marathon, Prometheus, ELK stack, etc. as a part of it. It doesn't seem like work. Seems more like fun to me. ;)

As you might know, [Mesos](http://mesos.apache.org/){:target="_blank"} is a cluster manager that abstracts the resources in your datacenter and presents it as a single pool of resources. It is an interesting project and I could appreciate it once I started exploring it. More about Mesos, maybe, some other time.

As a part of setting up Mesos cluster, I had to setup three systems as mesos-master nodes and about four systems as mesos-slave nodes. I was following [this](http://open.mesosphere.com/getting-started/datacenter/install/){:target="_blank"} tutorial by the amazing folks at [Mesosphere](https://mesosphere.com/){:target="_blank"} to set things up on CentOS 7 boxes.

As I was going through the blog, I realized how boring and cumbersome it'd be to manually install everything on almost seven systems. Even *Copy-Paste* seemed like a tiring thing to do.

On the other hand, it seemed like a good opportunity to learn some automation tool. I chose [Ansible](http://www.ansible.com/){:target="_blank"} as it is written in Python and, morever, it works over SSH - no need to install any agent.

Before being able to use Ansible from a system, you need to make sure that you've copied your SSH key into the remote systems and can do password-less login to those systems. [There's plethora of tutorials](http://lmgtfy.com/?q=ssh+passwordless+login){:target="_blank"} at your disposal to set this up.

Installing Ansible on an Ubuntu system is very simple. All you need to do is:

{% highlight sh %}
$ sudo apt-get install ansible
{% endhighlight  %}

Next, modify `/etc/ansible/hosts` to add information about the systems in Mesos cluster. If you're entering hostnames, make sure they're resolvable in your environment. Below is how hosts' information in my configuration file looks like:

{% highlight sh  %}
[mesos-master]
mesos-master-1.example.com
mesos-master-2.example.com
mesos-master-3.example.com

[mesos-slave]
mesos-slave-1.example.com
mesos-slave-2.example.com
mesos-slave-3.example.com
mesos-slave-4.example.com

{% endhighlight  %}

Here, `[mesos-master]` and `[mesos-slave]` are two groups with their set of systems. You'll refer the systems by their group name when using `ansible` or writing a playbook to be fed to `ansible-playbook` utility. More on playbooks a bit later.

Now that Ansible configuration and password-less login are done, go ahead and check if Ansible can communicate with your systems. This can be done using the simplest `ping` module:

{% highlight sh %}

$ ansible all -m ping

{% endhighlight %}

Here, `-m ping` tells Ansible to using its [ping module](http://docs.ansible.com/ansible/ping_module.html){:target="_blank"} and check if all the configured systems are reachable. **All** the configured systems because we have specified `all` in our command. `all` makes sure to execute specific command on all the groups specified in `/etc/ansible/hosts`. If things have been setup correctly, you would get pong response from the configured nodes. If any of the systems is not reachable, Ansible prints an error for the server while continuing with other systems.

Ping test is short and simple. True power of Ansible lies in executing a series of tasks on the systems in one go while you spend time learning something new! This is where Ansible Playbook comes into the picture.

With an Ansible Playbook, you can specify a list of tasks that are supposed to be executed on the group(s). Examples of tasks that can be executed are - update a system, install new packages, enable/disable daemons, execute specific command, clone a repo, etc. Ansible does all these with the help of [exhaustive set of modules](http://docs.ansible.com/ansible/list_of_all_modules.html){:target="_blank"} it has to offer to us!

This doesn't mean you can't specify just one task in the playbook - you very well can. Ansible Playbook is a yaml file. Below is an example Playbook that would do `yum -y update` on the groups `mesos-master` and `mesos-slave`:

***book-1.yaml***
{% highlight yaml %}

- hosts: mesos-master:mesos-slave
  tasks:
  - name: yum update
    yum: name="*" state=latest

{% endhighlight %}

As you can see, Ansible Playbook is a `yaml` file. The very first statement tells Ansible which hosts it is supposed to work on. Here, I have used `mesos-master:mesos-slave` which would ensure that the `tasks` are executed on both the groups. If I wanted to execute it on only one of those two groups, I'd have only specified `mesos-master` or `mesos-slave` without a `:`. The `name` directive gives a name to the task. Here, I have kept it *yum update* but you can even call it *Update all packages* or anything that pleases you. Next, we use the [yum module](http://docs.ansible.com/ansible/yum_module.html){:target="_blank"} and specify `name="*"` to indicate *all* packages installed in the system. `state=latest` tells yum to make sure that all the installed packages are updated to their latest version. 

This translates to doing `yum -y update` on all the seven systems.

To execute the Playbook, you simply do:

{% highlight sh %}
$ ansible-playbook book-1.yaml -u someuser -s
{% endhighlight %}

Above command would execute the tasks in `book-1.yaml` on the groups `mesos-master` and `mesos-slave` as the user `someuser` with sudo privileges because of the `-s` switch given at the end of the command. A few points to note here:

* You should be able to perform password-less SSH login on the systems in aforementioned two groups for the user `someuser`.
* `someuser` needs to be able to execute `sudo` commands without providing the password. 

For the second point, you need to put below in the `/etc/sudoers` on the systems in question:

{% highlight sh %}
someuser ALL=(ALL) NOPASSWD:ALL
{% endhighlight %}

Above line will grant the user `someuser` the privilege to execute any command using `sudo` *without* providing the password. Of course you can modify the `ALL` part at the end of the line to specify specific commands `someuser` is allowed to execute without the password. In that case, `someuser` will be prompted for password for all but those commands.

Let's say you want to install few packages on the systems. Here's how you do it:

***book-2.yaml***
{% highlight yaml %}

- hosts: mesos-slave
  tasks:
    - name: Install packages
      yum: name=git,vim state=latest

{% endhighlight  %}

Here, we are installing `git` and `vim` on the group `mesos-slave`. Note that latest version from your system's yum repos will be downloaded. This might not be the same as latest versions available on resepctive utilities' websites.

That's it for a basic getting started guide on Ansible. In the next blog, I will cover more examples than talk. ;)

Till then, if you have any feedback/comments, please let me know on [twitter](https://twitter.com/dharm1t){:target="_blank"}.
