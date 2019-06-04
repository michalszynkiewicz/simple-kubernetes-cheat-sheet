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
docker build -f src/main/docker/Dockerfile.jvm -t <image tag>:version .
```
or for the native binary
```
docker build -f src/main/docker/Dockerfile.native -t <image tag>:version .
```

## Working with Kubernetes

### Deploying applications
```
kubectl run <deployment name> --image=<image-tag>:<version> --port=8080 --image-pull-policy=IfNotPresent
```

### Exposing services
To expose externally using NodePort (the simplest yet ugliest solution):
```
kubectl expose deployment <deployment name> --type=NodePort
```

To expose internally:
```
kubectl expose deployment <deployment name>
```

To access a service exposed with NodePort use the following URL:
```
minikube service <deployment name> --url
```

### Redeploying applications
```
kubectl set image deployments/<deployment name> <deployment name>=<image name>/<version>
```

### ConfigMap

### How to start over
