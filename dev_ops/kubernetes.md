# Kubernetes

## Minikube Basics

`minikube start` to start the minikube server (minikube is for learning)

`minikube dashboard --port=` to open up minikube dashboard

use `minikube stop` and `minikube delete` to remove old clusters

## Pods

A pod represents one or more running containers. 

`kubectl get pods` to see pods

`kubectl port-forward port:port`

`kubectl logs PODNAME`

`kubectl delete pod PODNAME`

Every pod has an IP address internal to Kubernetes

`kubectl get pods -o wide` to see IP addresses of your pods

## Deployments

A deployment is an update to pods or replicasets
The desired state of the cluster is declared in the config file

`kubectl get deployment APPNAME -o yaml` to see config file

`kubectl edit deployment APPNAME` to edit config file

`kubectly apply -f file.yaml` to apply config file

Thrashing pods are pods that keep crashing and restarting.

Possible causes of thrashing pods include:
- a bug was introduced in the latest image version
- application is misconfigued and cant start properly
- a dependency of the application is misconfigured and the app cant start properly
- the app is trying to use too much memory and is being killed off by Kubernetes

## Config Maps

Config maps are used to managed environment variables in a way that's decoupled from container images.

`kubectl appply` is also used to activate a config map

Config maps are good for managing:
- Ports
- URLs of other services
- Feature flags
- Settings that change between environments

For sensitive info (like database credentials), Kubernetes Secrets should be used instead of config maps because config maps are not encrypted









