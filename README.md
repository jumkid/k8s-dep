# k8s-dep
configuration and deployment for Kubernetes

## Build git project

In case of any update on project source file, build it and test it. Once it is done, commit the update and push to github. If there is link between github and dockerhub, the auto build could be triggered and sync to dockerhub automatically, which is not the case to be mentioned here.

## Docker image
### New version
After commit the change, you could tag the new version. Say the old version number is 1.0.0, then the new version could be 1.0.1. Then create the docker image by
```
docker build -t 
```

### Replace the existing version

## Push image to docker hub

## Create yaml file

## Setup and start Kubernetes

