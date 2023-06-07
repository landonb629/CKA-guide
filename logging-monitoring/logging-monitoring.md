# Logging and Monitoring 

## Monitoring cluster 

Metrics Server
    - retrieves metrics from each kubernetes pod and nodes 
    - in memory monitoring solution
    - there is no historical data 

How are metrics generated?
- kubelet has a subcomponent called the cAdvisor
- cAdvisor: responsible for collecting the metrics from pods and making it available to the kube api-server

minikube 
``` minikube addons enable metrics-server ```

all other k8s clusters 
``` git clone  $METRICSSERVERREPO ```

``` kubectl create -f $FILES ```

you can view the cluster node information with 

``` kubectl top node ```

you can view pod metrics with 

``` kubectl top pod ```

## Managing application logs 

listing pod logs 

``` kubectl logs $podName ``` 

what if there are multiple containers in a pod

``` kubectl logs -f logger-pod $podName ``` 
