# CKA - Networking 

#### CoreDNS

CoreDNS binaries can be downloaded from github, or as a docker image. 

steps for downloading

``` wget $url ``` 

``` tar -xzvf $binary ``` 

``` ./coredns ``` 

Configuring CoreDNS 
- uses a file called Corefile 
- this is where we can specify where to get the IP to hostname mappings 

```
. { 
    hosts /etc/hosts
}
```

#### CNI 
- set of standards for how programs should be developed to solve networking challenges in a container runtime environment

ex:
- container runtime must create network namespace
- identify network the container must attach to
- JSON format of the network configuration


#### cluster networking 

IP + FQDN 
- each host must have a unique IP address
- each host must have a domain name 
- there also must be the following open ports: kube-api (6443), kubelet (10250), kube-scheduler (10251), kube-controller-manager (10252), services on the workers (30k - 32k), etcd (2379)

