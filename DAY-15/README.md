# Kubernetes Pod Scheduling with Node Affinity

This guide covers the steps to create and schedule Kubernetes pods using node affinity based on specific labels on nodes. The example demonstrates scheduling an NGINX pod and a Redis pod on specific nodes by applying node labels.

## Prerequisites

- A running Kubernetes cluster with at least two worker nodes (e.g., `worker01` and `worker02`).
- `kubectl` configured to interact with the Kubernetes cluster.

## Steps

### 1. Create a Pod with NGINX Image and Node Affinity

#### Create the NGINX Pod YAML

Create a `nginx-pod.yaml` file with the following content:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
```
This pod uses node affinity to only be scheduled on nodes with the label disktype=ssd.

#### Deploy the Pod
```
kubectl apply -f nginx-pod.yaml
```
#### Check the Pod Status
The pod will not be scheduled as no node has the label disktype=ssd yet. To check the status:

```
kubectl get pod nginx-pod -o wide
kubectl describe pod nginx-pod
```
You will see an error related to unsatisfiable node affinity.

#### Add Label to Worker01 Node
To schedule the pod, we need to label the node worker01 with disktype=ssd:

```
kubectl label node worker01 disktype=ssd
```
#### Verify the Pod is Scheduled
Check the pod status again:

```
kubectl get pod nginx-pod -o wide
```
The pod should now be scheduled on worker01.

### 2. Create a Pod with Redis Image and Node Affinity Without Value

#### Create the Redis Pod YAML
Create a redis-pod.yaml file with the following content:

```
apiVersion: v1
kind: Pod
metadata:
  name: redis-pod
spec:
  containers:
  - name: redis
    image: redis
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: Exists
```
This pod uses node affinity to be scheduled on nodes that have the disktype key, regardless of its value.


#### Deploy the Pod
```
kubectl apply -f redis-pod.yaml
```
#### Label Worker02 Node Without a Value
To schedule the Redis pod, we need to label the node worker02 with the disktype key but no value:

```
kubectl label node worker02 disktype=
```
#### Verify the Pod is Scheduled

Check the pod status again:
```
kubectl get pod redis-pod -o wide
```
The pod should now be scheduled on worker02.


