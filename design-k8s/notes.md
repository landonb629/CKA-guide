# Design and install a kubernetes cluster

#### Purpose 

Education
- minikube 
- single node cluster with kubeadm/GCP/AWS

Development and testing 
- multi-node cluster with a single master and multiple workers
- setup using kubeadm tool or quick provision on GCP or AWS or AKS


#### Production clusters 

- HA multi node cluster with multi masters 
- up to 5k nodes
- up to 150k pods 
- up to 300k containers
- up to 100 pods per node 

Nodes 
- can be physical or virtual 
- minimum of 4 node cluster 
- master vs worker nodes 
- must use 64 bit linux operating systems 

- master nodes should just be hosting control plane components 

#### Install kubernetes the ADM way

1. setup 3 nodes (designate one as the admin)
2. install a container runtime on the nodes 
3. install the kubeadmin tool on the nodes
4. init the master server
5. ensure network prerequisites are met, install and configure the pod network 
6. join the worker nodes to the master ndoe