
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

The resources as currently set up, work on OpenShift 3.7 and rely on
the following images in your cluster's `openshift` namespace:

* nodejs:10 (builder image for the app itself)
* nginx:1.16
* mongodb:3.6
* redis:5

The config file for Nginx depends on cluster DNS based on the project
name `learning-locker`.

## Storage

There are three PVCs included. `app-storage` is used by the UI, API and
workers. `xapi-storage` is used by the xAPI service. `mongodb-storage`
is used to persist MongoDB's data. `mongodb-backup-storage` is
dedicated to archives of the DB and is also mounted in the DB pods so
you can grab the backups there. 

TODO: Ask for some guidance from community on default sizes for these PVCs.

## The Learning Locker Build

The image containing the application code for the UI, API and workers
is built using Redhat's S2I technology and is specified in the [build.yaml
file](resources/build.yaml). The build is a standard s2i build, using
the Node builder image from RedHat. The only customization is the
injection of the [s2i scripts](s2i) from this repo, necessary to build
and run the app.

## Bootstrapping the Project

Visit the example config maps in the config/maps directory and convert
them to `.yaml` files with the correct values for your environment.

Log in to your cluster with the `oc` client, create your project and
from the repo's root directory:

1. Create App and MongoDB Configuration:

`oc create -f config/maps/`

2. Create Nginx configuration:

The configuration depends on dhparam.pem coexisting in the directory
with the config file. If you don't have one, create it:

`openssl dhparam -out config/from-file/proxy/dhparam.pem 2048`

then, create the config map from the files:

`oc create configmap proxy-config --from-file=config/from-file/proxy/`

(or to replace the config after it's been created):

`oc create configmap proxy-config --dry-run --from-file=config/from-file/proxy/ | oc replace -f -`

Next, drop your key and certs into a directory with the names lrs.key and
lrs.crt and create the secret:

`oc create secret generic proxy-ssl --from-file=<your-ssl-directory>/ `

(or, again, to replace after initial creation):

`oc create secret generic proxy-ssl --from-file=<your-ssl-directory>/ -o yaml --dry-run | oc replace -f -`

3. Create the route:

This resource will enable users to reach the app. Visit the
example route at `config/routes/route.yaml.example` and add your LRS
hostname. Save the file with a `.yaml` extension. The .gitignore file
for this repo is set to ignore config/routes/*.yaml. You may wish to add
routes for all of your environments in this directory using a naming
structure such as `config/routes/dev.yaml`, `config/routes/stage.yaml`,
etc. Finally, create the route:

`oc create -f config/routes/<env>.yaml`

4. Create all of the resources (deployments, services, routes,
storage, image streams):

`oc create -f resources/`
        
The creation of the build config will kick off an initial build. The
build will take several minutes. Once the build finishes, it will be
rolled out for the API, UI and worker services. 

5. Rollout services

Trigger a rollout for the deployments that depend on images other than
the `learning-locker` image.

`oc rollout latest redis`
`oc rollout latest mongodb`
`oc rollout latest xapi`

6. Create the MongoDB schema and indexes:

After the app services are running, get a console on any of API, UI or
worker pods (e.g. `oc rsh ui...`) and run:

`yarn migrate`

## Updates

Coming soon...
