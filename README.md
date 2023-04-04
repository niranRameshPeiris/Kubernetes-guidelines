# Kubernetes Guidelines

## Creating Kubernetes Cluster
Below link can be used as a reference to setup a kubernetes cluster. 

https://phoenixnap.com/kb/install-kubernetes-on-ubuntu https://computingforgeeks.com/deploy-kubernetes-cluster-on-ubuntu-with-kubeadm/

## View all kubernetes resources
```
kubectl api-resources
```
## Get HELP
```
kubectl run --help
```
## Delete something
```
kubectl delete svc servicename
```
## Pods
### Create and edit pods
From yaml
```
kubectl create -f newpod.yaml
```
From command
```
kubectl run <name-of-pod> --image=<image>
```
View pod yaml without creating pod
```
kubectl run <name-of-pod> --image=<image> --dry-run=client -o yaml > pod.yaml
```
Change pod image
```
kubectl set image pod <pod-name> <current-image>=<new-image>
```
Edit pod
```
kubectl edit pod <pod-name>
```
### View pod information
View Labels
```
kubectl get pods --show-labels
```
Filter using labels
```
kubectl get pods --selector=foo=bar
```
```
kubectl get pods -l foo=bar
```
View the yaml file
```
kubectl get pod <pod-name> -o yaml
```
View more info
```
kubectl get pod <pod-name> -o wide
```
Describe a pod
```
kubectl describe pod <pod-name>
```
### Add/ Edit/ Delete Labels
Add labels
```
kubectl label pod <pod-name> k1=v1 k2=v2 k3=v3
```
Add labels by filtering pods
```
kubectl label pod <pod-name> k1=v1 k2=v2 k3=v3 -l key=selected
```
Delete Labels
```
kubectl label pod <pod-name> k1-
```
Edit Labels
```
kubectl label pod <pod-name> k2=v2.1 --overwrite

```
### Add/ Edit/ Delete Annotations
Add Annotation
```
kubectl annotate pod <pod-name> k1=v1 k2=v2 k3=v3
```
Delete Annotation
```
kubectl annotate pod <pod-name> k1-
```
Edit Annotation
```
kubectl annotate pod <pod-name> k2=v2.1 --overwrite
```
### Create a pod with arguments 
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - image: ubuntu
    name: mypod
    args:
    - /bin/sh
    - -c
    - touch /niran/info.txt; sleep 600
```
### Setting Pod Resource Limits and Requirements
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - image: ubuntu
    name: mypod
    resources:
        limits:
            cpu: "5"
            memory: "56Gi"
        requests:
            cpu: "0.5"
            memory: "30Mi"
```
## Namespaces
### List Namespacses
```
kubectl get namespaces
```
Create Namespace
```
kubectl create namespace mynamespace
```
List pods from all namespaces
```
kubectl get pods -A 
```
```
kubectl get pods --all-namespaces
```
List pods under a namespacse
```
kubectl get pods -n mynamespace
```
Create a pod in a differenet namespace
```
kubectl run ntest --image=nginx -n mynamespace
```
## Services





## 
```

```

