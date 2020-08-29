
# Learning Locker for OpenShift 

This is a OpenShift translation of the [HT2 Labs Learning
Locker](https://www.ht2labs.com/learning-locker/) project. It was
influenced by an open source [Dockerized
version](https://github.com/michzimny/learninglocker2-docker), which is
based on the [custom installation instructions for Learning
Locker](http://docs.learninglocker.net/guides-custom-installation/) as
well as the `.env` and `nginx.conf.example` files in the Learning
Locker repo.

OpenShift is Kubernetes. On Amazon, Azure, etc.

## Components

Learning Locker is composed of several microservices - there is a UI
server and an API server that comprise the Learning Locker UI. The CLI
and worker components are also built from the main Learning Locker [mono
repo](https://github.com/LearningLocker/learninglocker). 

The xAPI microservice is built from a separate repo. HT2 labs makes
a docker image available on Docker Hub and the dockerized version of
Learning Locker utilizes this image. It appears to be updated so I
decided to use it as well for now.

It may be helpful to check out the [architectural
overview](http://docs.learninglocker.net/overview-architecture/) on
Learning Locker's site to get a deeper understanding of the application.

## Prerequisites

The resources as currently set up, rely on the following images
being available in your cluster's `openshift` namespace:

* nodejs:10 (builder image for the app itself)
* nginx:1.16
* mongodb:3.6
* redis:5

## Storage

There are three PVCs included. `app-storage` is used by the UI, API and
workers. `xapi-storage` is used by the xAPI service. `mongo-storage`
is used to persist MongoDB's data. You may wish to tweak the amounts
for each PV. 

TODO: Ask for some guidance from community on default sizes for these PVCs.

## The Learning Locker Build

The resources in the app/ subdirectory are used to build the Learning
Locker image used by several of the deployments. The build is an s2i
build, using the Node builder image from RedHat. The only
customization is the injection of the [s2i scripts](app/s2i) from this
repo, necessary to build and run the app.

## Bootstrapping the Project

Visit the example config maps in the config/maps directory and convert
them to `.yaml` files with the correct values for your environment.

Log in to your cluster with the `oc` client, create your project and from the root directory:

1. Create the configuration for the app and mongodb:

`oc create -f config/maps/*.yaml`

1. Create the Nginx configuration:

`oc create configmap proxy-config --from-file=./config/proxy/`

(or to replace the config):

`oc create configmap proxy-config --dry-run --from-file=./config/proxy | oc replace -f -`

1. Create all of the resources (deployments, services, routes,
storage, image streams):

`oc create -f resources/*.yaml`
        
1. Kick off a build of learning-locker image:

`oc start-build learning-locker`

The build will take several minutes. Once build finishes, it will be
rolled out for the API, UI and worker services.

Get a console on any of API, UI or worker pods and run

`yarn migrate`

to set up the MongoDB schema and indexes.


### Certificates

You'll want to instal your certs for the proxy as well.

Drop your key and certs into a directory and create the secret.

`oc create secret generic proxy-ssl --from-file=./config/proxy-ssl/ `

or to update:

`oc create secret generic proxy-ssl --from-file=./config/proxy-ssl/ -o yaml --dry-run | oc replace -f -`

## Updates

Coming soon...
