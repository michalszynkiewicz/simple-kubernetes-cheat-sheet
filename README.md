# Kubernetes cheat sheet for Quarkus workshop
The instructions assume the user runs on Linux

## Installing minikube

Download it:
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64 \
  && chmod +x minikube
```

Add it to the `$PATH` and start by:
```
minikube start --cpus 2 --memory 8096
```

More info: https://kubernetes.io/docs/tasks/tools/install-minikube/

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

### Exposing applications
The element of the Kubernetes ecosystem that is responsible for exposing applications is called a *service*.

#### Expose a service internally:
```
kubectl expose deployment <deployment name>
```

#### Expose a service externally, one can use NodePort.
This is the simplest yet the ugliest solution:
```
kubectl expose deployment <deployment name> --type=NodePort
```

#### Access a service exposed with NodePort use the following URL:
```
minikube service <deployment name> --url
```

#### Access a service from within the cluster


##### Testing in-container access
TODO: exec some container and a request within

### Redeploying applications
```
kubectl set image deployments/<deployment name> <container name>=<image name>/<version>
```

For one-container deployments, created with kubectl run, `container-name` is the same as `deployment-name`

### ConfigMap

### How to start over


## A Web UI
Minikube doesn't come with a web console installed but it's possible to create one.

To add it to your installation, run:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml
```

A dedicated user is required to access the UI (a.k.a. Dashboard).

To create a user run:
