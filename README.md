# Kubernetes cheat sheet for Quarkus workshop
The instructions assume the user runs on Linux


## Installing kubectl

Download it:
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl
```

And add to `$PATH`.

## Installing minikube

Install KVM or Virtualbox.
E.g., if you choose KVM and you're on Fedora, run:
```
sudo dnf install qemu-kvm
```

Download it:
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
```

For instructions for the other operating systems and more information see https://kubernetes.io/docs/tasks/tools/install-minikube/

If you wish to run minikube on KVM, configure it as follows:
```
minikube config set vm-driver kvm2
```

Add it to the `$PATH` and start by:
```
minikube start --cpus 2 --memory 8096
```

## Building a docker image
A generated Quarkus application provides Dockerfiles, with building instructions, in `src/main/docker`.
To use it:
1. build your application with Maven (add `-Dnative -Dnative-image.docker-build=true` to build the native binary)
```
mvn clean package
```
2. to be able to push images to the minikube image registry, run
```
eval $(minikube docker-env)
```
3. build the image for hotspot
```
docker build -f src/main/docker/Dockerfile.jvm -t <image tag>:<version> .
```
or for the native binary
```
docker build -f src/main/docker/Dockerfile.native -t <image tag>:<version> .
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

#### Expose a service externally
This is the simplest yet the ugliest solution is to create a NodePort service:
```
kubectl expose deployment <deployment name> --type=NodePort
```

#### Access a service exposed with NodePort
You can use minikube to get a URL of a service exposed with NodePort:
```
minikube service <deployment name> --url
```

#### Access a service from within the cluster
In the cluster the services can be addressed by their name.

##### Testing in-container access
If you'd like to see what's going on inside Kubernetes, e.g. check what's accessible from it, you can create a pod
and access bash in it, e.g.:
```
kubectl run my-shell --rm -i --tty --image registry.access.redhat.com/ubi8/ubi-minimal -- bash

bash-4.4# curl <service-name>:<service-port>/path
```

### Redeploying applications

To deploy a newer version of an application:
```
kubectl set image deployments/<deployment name> <container name>=<image name>:<version>
```

To restart an application (effectively, delete the pod and let Kubernetes create a new one):
```
kubectl delete pod <pod-name>
```

For one-container deployments, created with kubectl run, `container-name` is the same as `deployment-name`

### Forwarding local ports to kubernetes
E.g. to test Jaeger without deploying it in the cluster, you can forward its port to the cluster:
```
ssh -i $(minikube ssh-key) docker@$(minikube ip) -R 14268:localhost:14268 -o "UserKnownHostsFile /dev/null"
```

The known hosts for the command are not stored because the host fingerprint may change on minikube's restart.

With it, you can access the exposed port at `minikube:<port>` (`minikube:14268` in the example above)

### ConfigMap
You can use `ConfigMap` to propagate configuration to a kubernetes cluster.

To create it, create a YAML file that defines the `ConfigMap` and apply it:
```
kubectl apply -f config-map.yaml
```
To update it, use `kubectl replace` instead of `apply`.

### How to get rid of it
```
minikube delete
```

### Troubleshooting

A few helpful commands:
```
kubectl get pod [-w] # list pods running in the current namespace, use -w to watch for changes
kubectl describe pod <pod name> # get more information about a pod
kubectl logs -f <pod name> # get logs of the pod

kubectl exec -it <pod-name> -- /bin/sh # start an interactive shell in a running container/pod
```

[Kubectl cheatsheet on viewing resources](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#viewing-finding-resources)

[Debugging applications in Kubernetes](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/)


## Web UI
Minikube doesn't come with a web console installed but it's possible to add one

### Installation
To add it to your installation, run:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml
```

Optionally, you can verify if the dashboard pod is started in the `kube-system` namespace:
```
kubectl get pod -n kube-system
```

To be able to access the dashboard, you need to proxy kubernets to localhost:
```
kubectl proxy
```

### Access
A dedicated user is required to access the UI (a.k.a. Dashboard).

To create a user run:
```
kubectl apply -f https://raw.githubusercontent.com/michalszynkiewicz/simple-kubernetes-cheat-sheet/master/1-create-dashboard-user.yml
```

And to create a role for the user:
```
kubectl apply -f https://raw.githubusercontent.com/michalszynkiewicz/simple-kubernetes-cheat-sheet/master/2-create-dashboard-user-role.yml
```

Then, take the token for `admin-user` from the output of:
```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```

Now, finally, you can go with your browser to:
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login, select `Token` and paste the token from the output of the above command.


That's it, you should be able to access the web console now.


### More info
https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/
