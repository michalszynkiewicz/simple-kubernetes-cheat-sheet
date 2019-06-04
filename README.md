# Kubernetes cheat sheet for Quarkus workshop
The instructions assume the user runs on Linux

## Installing minikube

## Building a docker image
A generated Quarkus application provides Dockerfiles with building instructions in `src/main/docker`.
To use it:
1. build your application with 
```
mvn clean package
```
2. to be able to push images to the minikube image registry, run
```
eval $(minikube docker-env)
``` 
3. build the image for hotspot
```
docker build -f src/main/docker/Dockerfile.jvm -t <image tag> .
```
or for the native binary
```
docker build -f src/main/docker/Dockerfile.native -t <image tag> .
```

## Working with Kubernetes

### Deploying applications

### Exposing services

### Redeploying applications
```
kubectl rolling-update <your deployment name>
```

### ConfigMap

### How to start over
