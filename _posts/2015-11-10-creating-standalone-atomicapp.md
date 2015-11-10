---
layout: post
title: Creating a standalone Atomicapp
categories:
- blog
---

I started exploring
[Nulecule](github.com/projectatomic/nulecule){:target="_blank"} and
[atomicapp](github.com/projectatomic/atomicapp) recently. This post is about
creating a simple standalone atomicapp that can be deployed using the
`atomicapp run` command.

I decided to write a Golang based application in order to keep the resultant
[Docker](http://docker.com){:target="_blank"}  image size small. In this case, it turned out to be
just 7 MB! All that the app does is print the number of times the web page
was visited in past one minute.

###Go Application

Code for the Golang app can be found
[here](http://github.com/dharmit/gocounter){:target="_blank"} . To test this
app before packaging it into a Docker image, simply do `go run` or build it
using `go build` and run it. If you want to do a `go build`, do it using
following command:

    $ CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o gocounter .

This is because we are packing the Go binary in scratch image (refer Dockerfile
linked later). Now, to access the web page being served by this code, open up
`localhost` on your web browser and refresh it a few times.

###Dockerize the application

Use the
[Dockerfile](https://github.com/dharmit/gocounter/blob/master/Dockerfile){:target="_blank"}
in the same repo to build a Docker image. Once the image is ready, you can test
it using `docker run` command like below:

    docker run -d -p 80:80 <image-name>

Again, you can verify that it's running by checking the output of `curl
localhost` or opening up `localhost` in your web browser.

###Nuleculize the applicaiton

Coming to the main part of the post, we now Nuleculize the application by
writing a Nulecule file (simply called as `Nulecule`) and building a Docker
image later.

[Nulecule](https://github.com/dharmit/gocounter/blob/master/nulecule/Nulecule){:target="_blank"}
file for the `gocounter` app is provided in the repository. We'll do a quick
walkthrough of the contents in that file. Note that it's a yaml file.

    ---
    specversion: 0.0.2
    id: gocounter
    
    metadata:
        name: gocounter
        appversion: 0.0.1
        description: Atomicapp of a golang based app that prints number of hits in
    past one minute
    graph:
        - name: gocounter
          params:
            - name: goImage
              description: Go image
              default: dharmit/gocounter:master # Default tag given by Docker Hub
    on the basis of name of branch
            - name: NODE_PORT 
              description: Port on the host system to be used
              default: 32000
          artifacts:
            docker:
              - file://artifacts/docker/gocounter
            kubernetes:
              - file://artifacts/kubernetes/gocounter-service.yaml
              - file://artifacts/kubernetes/gocounter-rc.yaml

First line indicates the spec version to be used. This is the version of the
Nulecule spec that is supposed to be used. The `metdata` section contains info
that one can set as per his/her app. `graph` is a representation of the
dependencies of the container app. It becomes really interesting when there's
a multi-container app to be deployed. The values you see udner `params` can be
found under the kubernetes artifacts stored under
`nulecule/artifacts/kubernetes/gocounter-rc.yaml`. All we have done here is
provided default values for the params. The artifacts section indicates the
artifacts to be referred to when deploying using a specific provider. So far,
Docker and Kubernetes are the supported providers.

With that in place, we have a
[Dockerfile](https://github.com/dharmit/gocounter/blob/master/nulecule/Dockerfile){:target="_blank"}
which uses the Nulecule and artifacts to build a Docker image which has the
information about how to deploy the app.

This might sound like just a layer to
deploy Dockerized application but the true power of this can be exploited when
there's a multi-container application and you can pack all the metadata into
Nulecule spec compliant Docker image which will deploy all the containers with
a simple `atomicapp run` command

###Deploy the application

Sorry if that description made you grab a coffee. :wink:

To deploy this application, you'll need a system that has atomicapp installed.
I'd recommend using the CentOS based vagrant box `atomicapp/dev` for the
purpose. Make sure to:

- Update the system with `yum -y update`
- Install `atomicapp` by cloning the
  [repo](https://github.com/projectatomic/atomicapp/){:target="_blank"}. It
might require you to install some dependencies.

Coming to the part of deploying, below are the three options you have.

####Option 1 - Interactive

Run the image with below command. It'll use `kubernetes` as the default
provider. It will prompt for the parameters in Nulecule file that do not have
default values.

    $ [sudo] atomicapp run dharmit/gocounterapp

####Option 2 - Unattended

Create a new directory and in that, create a file called `answers.conf` with
below contents:

    [gocounter]
    goImage = dharmit/gocounter:master
    NODE_PORT = 32000
    [general]
    namespace = default
    provider = docker


Now, from the directory containting `answers.conf` file, run the application:

    $ [sudo] atomicapp run -a answers.conf dharmit/gocounterapp

####Option 3 - Install and Run

You can download the application, review the `Nulecule` file and edit
answerfile before running the application.

1. Download the app using `atomicapp install`:

        $ [sudo] atomicapp install dharmit/gocounterapp

2. `cd` into the directory where the files were loaded by above command. Rename
   `answers.conf.sample`:

        $ mv answers.conf.sample answers.conf

3. Edit `answers.conf`, review the files if desired and run:

        $ [sudo] atomicapp run -a answers.conf dharmit/gocounterapp


###Check the app

To check the app, open [http://localhost:32000](http://localhost:32000) in the
web browser. If you have chosen some other value for the environment variable
`NODE_PORT`, use that port instead of 32000.
