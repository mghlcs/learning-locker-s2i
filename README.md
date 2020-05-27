
# Learning Locker for OpenShift 

This is a OpenShift translation of the [HT2 Labs Learning
Locker](https://www.ht2labs.com/learning-locker/) project. It was
influenced by an open source [Dockerized
version](https://github.com/michzimny/learninglocker2-docker), which is
based on the [custom installation instructions for Learning
Locker](http://docs.learninglocker.net/guides-custom-installation/) as
well as the `.env` and `nginx.conf.example` files in the Learning
Locker repo.

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

## Storage

There are three PVCs included. `app-storage` is used by the UI, API and
workers. `xapi-storage` is used by the xAPI service. `mongo-storage`
is used to persist MongoDB's data.

## The Learning Locker Build

The resources in the app/ subdirectory are used to build the Learning
Locker image used by several of the deployments. The build is an s2i
build, using the Node builder image from RedHat. The only
customization is the injection of the [s2i scripts](app/s2i) from this
repo, necessary to build and run the app.

## Bootstrapping the Project

Create OpenShift resources from the root directory.

1) Create the build resources required to build the Learning Locker Node app image:

`oc create -f app/build.yaml`
    
    
1) Kick off a build of learning-locker:

`oc start-build learning-locker`
    
    
1) Create the proxy config map resources:

`oc create configmap proxy-config --from-file=./proxy/config/`
    

(or to replace the config):

`oc create configmap proxy-config --dry-run --from-file=./proxy/config/ | oc replace -f -`
    
1) Create the Storage, UI, API, xAPI, MongoDB and Proxy (Nginx)
resources:

`oc create -f *.yaml`
    
