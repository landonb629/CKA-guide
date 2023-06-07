## Labs - Core Concepts section 

1. create a replicaset with 4 replicas, using the nginx image.
    - delete one of the pods and notice what happens 

2. create a pod using the nginx image 

## Labs - Scheduling section 

1. create your minikube cluster without the scheduler 
    - create an nginx pod, check and see what state its in 
    - manually schedule the nginx pod using the nodeName attribute in the pod file 

2. using nodeAffinity 
    - add a taint to the second node with the key/value pair of role=worker
    - create a deployment, add a nodeAffinity section to make sure that the pod gets scheduled on the worker node 

## Labs - Scheduling 



## Labs - Monitoring and Logging

1. create a pod and check the output logs 

2. create a new minikube cluster with 2 nodes, add the metrics server addon
    - deploy the manifest files called lab2Logging.yml, wait a few minutes and check the pods resource consumption and node resource consumption


## Labs - Application Lifecycle Management

1. deploy the manifest file called mgmtlab01.yml 
    - upgrade the manifest to use nginx:1-alpine-slim
    - deploy the upgrade 
    - check the status of the upgrade
    - rollback to the previous version