---
layout: post
title: Using CentOS CI to automatically test GitHub pull requests
categories:
- blog
---

Being able to automatically test pull requests is one of the most important
requirements for most open source projects. Doing this is very easy when you
use the [Jenkins instance](https://ci.centos.org/) provided by [CentOS
project](https://www.centos.org/).

If you don't already have access to CentOS CI, you can read all about it on its
[wiki page](https://wiki.centos.org/QaWiki/CI/). You can also come and talk
with the team on `#centos-devel` channel on Freenode. One of the interesting
and important things to understand when using CentOS CI is its middle layer
that manages the provisioning, maintenance and teardown/rebuild of the Nodes -
[Duffy](https://wiki.centos.org/QaWiki/CI/Duffy). I'd highly recommend anyone
getting started with CentOS CI to read through [this amazing
blog](http://dustymabe.com/2016/01/23/the-centos-ci-infrastructure-a-getting-started-guide/)
written by Dusty Mabe. It helped me more than any other piece of documentation
around!

You can either configure the job using Jenkins Web UI or using the amazing
[Jenkins Job Builder](http://docs.openstack.org/infra/jenkins-job-builder/)
tool created by the folks from OpenStack project. In this post, we'll explore
both the ways to configure a job.

**NOTE**: Before starting configuration you'll have to ensure that your GitHub
token is stored on Jenkins. This is to ensure that Jenkins instance provided by
CentOS CI can interact with GitHub and perform necessary actions like reading
and sending notifications on your repo.

### Using the Jenkins UI

1. From homepage of the Jenkins UI, click on "New Item".
2. Enter project name and select "Freestyle Project".
3. "Configure" page that appears next is where we do all the necessary
   configuration.
4. Check the "GitHub Project" check-box and enter the URL to your GitHub
  project in the input box that appears once you check it.
5. Under the "Source Code Management" select "Git". This enables a few items
   where we need to enter or modify details. These items are divided into two
   sections: *Repositories* & *Branches to build*.
6. Repositories section:
    - Repository URL - enter the URL to your repo that you entered earlier
    - Click on "Advanced"
        - Name: write `origin` in the input box
        - Refspec: write `+refs/pull/*:refs/remotes/origin/pr/*`. This is
          specific to pull requests.
7. In the Branches to build section, enter `${sha1}`. This is an environment
   variable specific to GitHub Pull Request Builder plugin
8. Under "Build Triggers" select "GitHub Pull Requets Builder" option. In the
   items that appear after selecting it, click on check-box for "Use github
   hooks for build triggering". In the "Admin list" textarea, enter your
   GitHub username. This will make sure that pull requests you raise will be
   automatically tested. If you'd like all pull requests to be built
   irrsepective of the user who raised it, click on "Advanced" and check the
   option "Build every pull request automatically without asking (Dangerous!)."
9. Next select "Build Step" that's most relevant to you. As a simple example,
   you can select "Execute shell" and in the textarea write `echo "Hello
   Jenkins"`.
10. The last option is "Post-build Actions" where selecting "Set build status
   on GitHub commit" is relevant for our useacase.

That's it. Click on "Save" and go to your GitHub repo and create a simple PR
to test things. With step 9 mentioned above, you'll see a job triggered and
`echo "Hello Jenkins"` getting executed.


### Using Jenkins Job Builder

Jenkins Job Builer (JJB) takes yaml file that has details about the job to be
triggered. It parses this yaml file and configures the job accordingly on
Jenkins. It also requires a file that contains details about the Jenkins
instance.

Files you'll need when working with JJB:

- YAML file that has job details
- `.ini` file that has Jenkins instance's details like username, password,
  url, etc.

In simplest case, our `.ini` file will look like below:

~~~
$ cat jenkins_jobs.ini
user=<username>
password=<password>
url=https://ci.centos.org
~~~

Of course the `url` section needs to change if you're not using Jenkins
instance provided by CentOS CI.

Above job that we configured using Jenkins UI would look like below if we were
to configure it using YAML:

~~~
$ cat job.yaml
- job:
    name: test
    properties:
        - github:
            url: <your-gh-repo-url>
    builders:
        - custom-builder
    triggers:
        - custom-trigger
    scm:
        - custom-scm

- builder:
    name: custom-builder
    builders:
        - shell: "echo Hello Jenkins"

- trigger:
    name: custom-trigger
    triggers:
        - github-pull-request:
            admin-list:
                - <your-gh-username>
            github-hooks: true

- scm:
    name: custom-scm
    scm:
        - git:
            url: <your-gh-repo-url>
            refspec: +refs/pull/*:refs/remotes/origin/pr/*
            branches:
                - ${sha1}
~~~

Execute below command to generate job from the YAML file on the Jenkins instance:

~~~
$ jenkins-jobs --conf jenkins_jobs.ini update job.yaml
~~~

Just generate a pull request and you're good.

For any feedback/comments, ping me on twitter [@dharm1t](https://twitter.com/dharm1t).
