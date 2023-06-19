# Cluster Maintenance 


### OS upgrades 

what happens if a node goes down?

- when a node goes offline, the default time to wait is 5 minutes, after 5 minutes the nodes will be brought back up on another host, if they were part of a replicaSet

**NOTE** This is called the node eviction timeout 

What do you do if you aren't sure if the nodes will come back up

- you can drain the nodes using the following command: ``` kubectl drain $NODENAME ``` 
- when you do this, the node will be cordoned (meaning no pods can be scheduled on it)
- you then need to uncordon the node with the following command: ``` kubectl uncordon $NODENAME ``` 

**Draining:** gracefully shuts down pods and brings them up on another node

### Kubernetes Releases 

How does kubernetes manage their project releases? 
- releases have 3 parts MAJOR.MINOR.PATCH ex: v1.11.3
- patches are released most often 
- every few months new features com out through minor releases 

What does a bug fix look like?

- bug code is written 
- deployed to alpha, well tested 
- moved to beta
- move to main release 

### How to upgrade kubernetes from one version to another?
- The main components of k8s can be on different release versions 
ex: kube-apiserver, controller-manager, kube-scheduler, kubelet, kube-proxy, kubectl 

- No components can be higher than the kube-apiserver

Kube-apiserver -> x (highest version)
controller manager and kube-scheduler -> x-1 ()
kubelet and kube-proxy -> x-2 
kubectl -> x+1 > x-1


When should you upgrade?

if you're on 1.10 and 1.11, 1.12 are released, you should upgrade before 1.13 is release because 1.10 will become unsupported at that time 


Recommended approach:
- Upgrade one minor version at a time 

How? 

This is going to depend on the way that your cluster was deployed in the first place

- cloud provider: use your cloud providers recommended methods of deployment 
- kubeadm: use the CLI utility to upgrade 
- the hard way: you will have to upgrade each component by hand

You should first upgrade your master nodes, then upgrade your worker nodes. 

- the master going down, will not have any impact on your applications but, all your management functions are not working
    ex: you can't deploy any applications, if your pod goes down, it wont be redeployed 

Upgrade the worker nodes:
- Strat 1: upgrade all the worker nodes at once, this will bring all your applications down, and your users will not be able to access them
- Strat 2: upgrade one node at a time, workloads will move off that node 
- Strat 3: add new nodes to the cluster with the newer software version, move workloads over to the new nodes, remove your old nodes 

performing an upgrade with kubeadm 

``` kubeadm upgrade plan ``` -> This command gives you a lot of good information regarding your current cluster versions and the available versions 

1. upgrade your kubeadm tool to the version of k8s you want to get to 

``` apt-get upgrade -y kubeadm=1.12.0-00 ``` 

2. run your upgrade command 

``` kubeadm upgrade apply v1.12.0 ``` 

3. upgrade your kubelets 

``` apt-get upgrade -y kubelet=1.12.0-00 ``` 
``` systemctl restart kubelet ``` 

4. worker node upgrades 

- move all the workloads to another node in the cluster using the drain command, which migrates all the pods and cordons the node 

``` kubectl drain $nodename ``` 

- upgrade the kubeadm package, the kubelet, restart the kubelet 

``` apt-get upgrade -y kubeadm=1.12.0-00 ``` 

``` apt-get upgrade -y kubelet=1.12.0-00 ``` 

``` kubeadm upgrade node config --kubelet-version v1.12.0 ``` 

``` systemctl restart kubelet ``` 

``` kubectl uncordon $nodename ``` 



### Backup and Restore methods 

What should we be backing up? 
- resource configuration
- ETCD cluster
- Persistent Volumes 

Resource configuration 
- store manifests in source code repositories 
- when you lose your cluster, you can apply your configuration files on the new clusters 

best approach:
- query the kubeapi server 
- save all resource configurations 

``` kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml ```  

ETCD cluster 
- stores information about the cluster itself 
- when we configure ETCD, we picked where we would store the data directory for this service 
- you can backup that directory 
- etcd comes with a backup snapshot utility 

``` ETCDCTL_API=3 etcdctl snapshot save snapshot.db ``` 

view the snapshot status -> use the etcdctl status command 

restore the cluster from the backup 

1. stop the kube-apiserver 

``` service kube-apiserver stop ``` 

2. run the restore command for ectdctl
    - the etcd configuration will init a new cluster config 

    ``` etcdctl snapshot restore snapshot.db --data-dir /var/lib/etcd-from-backup ``` 
3. reload the service daemon 

``` sytemctl daemon-reload ``` 

4. restart the ETCD service 

``` service etcd restart ``` 

**NOTE: when running etcdctl, you must specify the configuration for certificates, and keys, and the endpoints where etcd server is listening**