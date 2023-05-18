# Cluster Architecture 

Worker nodes:
- host applications as containers
Master nodes:
- manage, plan, schedule, and monitor nodes

etcd: 
 - database that stores information about the state of the cluster
 - stores in key:value pairs

kube scheduler:
 - identifies the right node to place a kubernetes application on 

kube controller manager:
 - takes care of different functions like the node controller and replication controller

kube api-server:
 - responsible for orchestrating all operations within the cluster
 - exposes the api that manages the kubernetes cluster
 - monitors the state of the cluster 

container runtime:
 - this needs to be installed on the controllers in order to run containers 

kubelet:
 - runs on each node in a cluster 
 - listens for instructions from the kube api-server
 - deploys / destroys containers on the nodes 

kube proxy:
 - allows the containers on each worker node to talk to each other 

## Docker vs containerd
docker uses containerd, but the support for dockers container runtime itself was removed. with containerd, images built using docker are still compliant 

containerd:
 - part of the CNCF 
 - it's a container runtime supported by kubernetes 
 - ctr is a command line tool for debugging containerd
 - use nerdctl because that will give you more robust features 

crictrl:
 - used to interact with the container runtime 
 - used for inspecting and debugging 
 - works across different runtimes 

# ETCD

what is a key:value store? 
 - stores information in the form of documents 
 - changes to one file doesnt impact others 
 - etcdctl is the command line utility for using etcd

how to store a key 
``` etcdctl set key1 value1 ```

- stores information in the cluster, all the information when you run kubectl comes from etcd

etcd could be different based on the different ways to setup a cluster 
- setup from scratch:
    - you download and configure the etcd binaries yourself 
    - lots of config comes from the TLS certificates
    - setup as a service running on the advertised client url
- steup using kubeadm
    - etcd is setup as a pod in the cluster 

- advertised client url = where etcd is listening 

## kube api-server 
- this is the layer that controls interactions with a kubernetes cluster 
- only component that interacts directly with the etcd cluster 

if setting up the manual way
- the api-server binaries must be installed and run on the cluster as a service 
- there is a large amount of configuration that must be done 

what happens when you run ``` kuebectl get nodes  ```
1. request is sent to the kube api-server 
2. the kube api-server authenticates the request and gets the information from etcd

# kube controller manager 
- services that handles the core control loop that comes with kubernetes 
- a control loop is a non terminating loop that regulates the status of the system 
- a controller watches the state of the kubernetes cluster and makes sure that that the actual state, matches the desired state in the cluster 

node controller: checks the status of the nodes every 5 seconds
replication controller: monitors the pods on the cluster and makes sure the desired state is matched

- those are just two controllers, there are a lot more controllers on the cluster. this is the brain behind a lot of kubernetes stuff

how to install the kubernetes controller manager 
- takes a bunch of CLI options on install 
- you download the binary from the kubernetes website  
- by default, all controllers are installed 

## Kube-scheduler 
- decides which pod goes where based on , doesn't actually place the pod 

How do you install the kube-scheduler 
- install the binary from the kubernetes website 
- extract it, run it as a service and specify the configuration 

using kubeadm 
- it will deploy it as a pod in the kube-system namespace


## Kubelet 
- the agent that runs on the nodes in the cluster 
- registers the node with the k8s cluster 
- creates pods on the nodes 
- reports back to the kube api-server 


using kubeadm 
- does not automatically create the kubelet 
- kubelet must always be manually installed 


## Kube proxy 
- all pods in a kubernetes cluster can talk to each other, this is because of a pod networking solution



## Replica Sets 
controllers: brains behind kubernetes, they monitor objects and respond accordingly 

what is a replication controller?
- helps us run more than one single pod in a kubernetes cluster so we can achieve HA 
- you can use a replication controller with one pod, it will help respond to failures 
- spans across multiple nodes, helps to scale our application and provide load balancing 

Replication controller and replica set

- replication controller: the old way of setting up replication
- replica set: the new way to setup replication

example of replica controller: 

```
apiVersion: v1 
kind: ReplicationController
metadata:
  name: myapp-rc

spec:
  replicas: 3
  template:
    metadata:
      name: mypod
      labels:
        app: fronted
    spec:
      containers: 
        - name: mypod 
          image: nginx
```

example of a replica set: 

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs

spec:
  replicas: 3 
  selector:
    matchLabels:
      name: mypod
  template:
    metadata:
      name: mypod
    spec:
      containers:
        - name: mypod 
          image: nginx 
```

## Services 


## Deployments 

