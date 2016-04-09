---
layout: post
title: Using Ansible conditionals to avoid repetitive downloads
categories:
- blog
---

Lately, I've been working on Ansible, Jenkins and Vagrant almost everyday.
Working on Ansible is especially fun as it's exciting to see things get set up
automatically with a single command. And that on multiple systems while I
sit back and invest my time in checking new stuff on Twitter, Hacker News or
reddit. ;-) 

As a part of working on CentOS ContainerPipeline I was testing one of the
Vagrantfiles and Ansible
[configuration](https://github.com/rtnpro/centos-cccp-ansible) and figured
that the way our playbooks are written, caused unncessary removal and
reinstall of the same package. To know more about ContainerPipeline, you can
check either the [CentOS wiki](https://wiki.centos.org/ContainerPipeline) or
its [GitHub repo](https://github.com/kbsingh/cccp-index).

If you'd directly like to check the GitHub diff of the Ansible playbook, skip
to the [end of the post](#githubdiff).

As a part of setting up the CentOS 7 Vagrant box with necessary packages and
configurations we ensure that Java package `java-1.7.0-openjdk` is present on
the system. Earlier, the Ansible playbook was written such that it would
delete any Java package installed on the system and then install the desired
`java-1.7.0-openjdk` package. This was far from ideal because if the playbook
was executed again, 1) it'd remove the `java-1.7.0-openjdk` package as well and
2) it'd re-download and install the same package. That's a waste of time as
well as network bandwidth.

Since Ansible's quite feature-rich, I was expecting some kind of option with
its [yum module](docs.ansible.com/ansible/yum_module.html) to prevent the
unncessary reinstall I explained earlier. Unfortunately there's no option for
that. So, I headed to the `#ansible` channel on Freenode and asked
how do I go about it. Turns out [Ansible
Conditionals](http://docs.ansible.com/ansible/playbooks_conditionals.html)
is our friend here.

Before the task that removed any installed Java package on the box, we added a
task that'd check if the desired version of Java is installed. If that check
was affirmative, we would skip the next task which removed the Java package.
And that'd automatically cause the task to install the Java package to skip
(not in the Ansible terms) as well.

<a name="githubdiff"></a>

###GitHub diff of playbook
You can check the diff between the playbooks on our [GitHub
repo](https://github.com/rtnpro/centos-cccp-ansible/pull/4/files?diff=split).

If you've any comments or feedback, reach out to me on
[Twitter](https://twitter.com/dharm1t). Maybe we can improve the playbook even
more! :-)
