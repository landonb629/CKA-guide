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
