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


#### Pod Networking

- every pod should have its own IP 
- every pod should be able to communicate with every pod in the same node
- every pod shouldbe able to communicate with every pod in other nodes without NAT

How do we do this?

- create a bridge network on each node 

``` ip link add v-net-0 type bridge ``` 

- bring up the bridge networks on each node 

``` ip link set dev v-net-0 up ``` 

- assign an ip address to the networks 
    - each bridge will be on it's own subnet

``` ip addr add 10.233.1.1/24 dev v-net-0 ``` 

#### CNI in kubernetes 

Configuring CNI 
- configured in the kubelet service in each cluster 
- you can view the configuration fil on the node at /etc/cni/
- there can be multiple networking configurations available but they will be chosen in alphabetical order

kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml


#### CNI weave 
- you deploy this on each node in the cluster in the form of a daemonset
- each agent on the nodes, creates a bridge on the nodes 
- each agent will handle ip address management

How to deploy weave on a kubernetes cluster
- deploy as pods (daemonset) on the cluster 
- can be deployed as a service as well

#### Ipam weave
- the CNI plugin is responsible for assigning IP address's to containers

weave by default allocates 10.32.0.0/12 for the entire network 
- this is about 1 million IP addresses 
- the peers will split this range between them
- the ranges are also configurable 


#### Service Networking 
- when a service is created in a cluster, it's acessible by all pods in the cluster

Services are not created on each node, they exist across all nodes in the cluster
- kube-proxy is responsible for tracking the changes made to services
- services are given an IP address from a specific range, when they are created, the kube-proxy will get that IP information and create a forward rule 

How are the rules created by kube-proxy?
- three methods: userpace, iptables, ipvs
- you set this from a configuration when starting the kube-proxy service (--proxy-mode)
- the default is iptables

How are ip tables configured by kube-proxy?  ex: you have a database running in a pod, and you create a service for this database
- your service gets assigned an ip address from the service-cluster-ip-range configured in your kube-api-server configuration
- kube-proxy will create a new iptables rule, which you can see by listing iptables nat rules, and grepping for your service name ``` iptables -L -t nat | grep db-service ```

#### DNS in kubernetes
- k8s deploys a built in kubernetes server by default 
- whenever a service is created, the kube-dns server maps the name of the service to the IP address
- to address a service in another namespace, you need to append the namespace to the end of the service name 

ex: you have a service called web in the frontend namespace ``` http://web.frontend ```

all of the records in kubernetes DNS are grouped into a namespace, type, and root 

ex: web-service.apps.svc.cluster.local


#### CoreDNS in Kubernetes 
- CoreDNS server is deployed as a pod in the kube-system namespaces in the kubernetes cluster (2 pods in a replicaSet)
- these pods run the coredns executable, that requires a configuration file
- you use plugins in this file, example below
- the kubernetes plugin is used for configuring the root domain name 
- the config file that is passed to coredns is passed in as a config map 
- the pods will use a service to resolve, this is configured as the nameserver in each pod

```
.:53 { 
    errors
    kubernetes cluster.local in-addr.arpa ip6.arpa { 
        pods insecure
        upstream 
        fallthrough in-addr.arpa ip6.arpa
    }
    prometheus :9153
    proxy . /etc/resolv.conf 
    cache 30 
    reload
}

```

#### Ingress 
- when doing ingress, you need to deploy a solution like nginx, traefik, HAProxy to handle the routing (ingress controller)
- you create your rules for routing, that are then applied by the proxies deployed throughout the cluster

Ingress Controller 
- you don't have one deployed by default 
- you can install the nginx ingress controller using a helm chart
- if you want to install the ingress controller manually
    - you will need a deployment manifest, with arguments, and you will need to pass in a config map, and a service
examples: 
  - nginx, istio, contour, HAProxy, istio 


Ingress resources 

- you can configure path based routing, or host name based routing 
- ingress resources can be used to route to different services for different paths in an application
- you should also deploy a default service

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress
spec:
  rules:
  - http:
      paths:
      - path: /dev
        pathType: Prefix
        backend:
          service:
            name: dev-service
            port:
              number: 80
      - path: /prod
        pathType: Prefix
        backend:
          service:
            name: prod-service
            port:
              number: 80
```

Ingress - Annotations and rewrite-target

example: you have a dev server and a prod server 
- http://dev-service/$port
- http://prod-service/$port 

what are we trying to achieve? 
- when a user visits this url http://ingress-service:ingress-port/dev-service they are forwarded to http://dev-service:$port 

without the rewrite target option, this would happen

- http://ingress-service:ingress-port/dev-service -> http://dev-service:$port/dev-service

This would cause errors because the backend service is looking for the path to be / not /dev-service

the fix for this is to use the rewrite URL, when the request is passed on to the dev-service or prod-service.
 
example: 

```
apiVersions: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rewrite-test
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
```




