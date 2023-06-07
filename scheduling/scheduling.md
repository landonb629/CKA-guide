# Scheduling 

## Manually Scheduling 

How does a scheduler work?
- when you create a pod, there is a field that you don't set called nodeName
- k8s goes through all the pods and finds the pods that have no nodeName, those get scheduled
- scheduling algorithm is run and then the pod gets assigned to a node 

what if you dont have a scheduler?
- set the nodeName property on the pods manually
- if you have already created the pod, you can't edit that nodeName attribute 

how to schedule an existing pod? 
- create a Binding object, send a post request to the kube api-server

binding definition file 

```
apiVersion: v1 
kind: Binding 
metadata:
  name: nginx 
target:
  apiVersion: v1 
  kind: Node 
  name: node02
```

- convert the definition file to JSON and send it to the binding API on the kubernetes cluster


## Labels and Selectors 

Labels: properties that are attached to items in kubernetes that allows you to distinctly filter them 
selectors: selectors are what you use to filter for certain labels in kubernetes 

example: adding a label to pods so a deployment can find and monitor them 

ex:

``` 
apiVersion: v1 
kind: Pod 
metadata:
  labels: 
    type: frontend
    env: development 
    owner: frontend-team
```

## Taints and Tolerations 

taint + tolerations: 
  - used to set restrictions of which pods can be placed on a node 
  - taints are placed on nodes to specify what pods can be placed on that node 
  - pods can be tolerant to the taint, and then they can be placed on that node 
  - taints are added to nodes using labels 

How to add a taint to a node 

``` kubectl taint nodes $nodeName key=value:taint-effect ```

How to add a toleration to a pod 

``` 
spec:
  containers: 
    - name: test
      image: nginx
  tolerations:
  - key: "key"
    operator: "Exists"
    value: "value"
    effect: "NoSchedule"

```

How to remove a taint from a node 
ex: controlplane node has a taint of control-plane:NoSchedule

``` k taint node controlplane control-plane:NoSchedule- ```

What are the different taint-effects?
- defines what happens to the pods if they are not tolerant 

- NoSchedule: if the pod isnt tolerant, it will not be scheduled on that node 
- PreferNoSchedule: can still be scheduled, even if the pod isnt tolerant
- NoExecute: new pods cannot be scheduled, existing pods that dont tolerate the taint will be evicted 


## Node Selectors 
- allows you to choose the node based on labels that the pod will be scheduled on 
- you must first label your node 
- limitations when trying to do complex node scheduling 
  ex: place on any node that isnt small 

ex: 

label your kubernetes node
``` kubectl label nodes controlplane size=small ```

add the nodeSelector to the pod 
```
apiVersion: v1 
kind: Pod 
metadata:
  name: large-pod 
spec:
  containers:
    - name: nginx
      image: nginx 
  nodeSelector:
    size: small
```

## Node Affinity 
- ensure pods are scheudled on specific nodes but allows for complex logic / advanced expressions 
- declaration of node affinity configuration is much more complex 
- if there are no nodes that match the affinity rule, that is answered by the property under node affinity
- if the affinity rule contains "IgnoredDuringExecution" if affinity rules changed once the pod has already been scheduled, it will not be moved


example: 
```
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: size
          operator: In
          values: 
          - Large 
```

operator options 
- In
- NotIn
- Exists: check if the label exists, you don't need the value 

Node affinity types:
- requiredDuringSchedulingIgnoredDuringExecution: pod will not be scheduled if the node affinity cannot be found 
- prefferedDuringSchedulingIgnoredDuringExecution: pod can still be scheduled 


## Resource Requirements 

resource requests: the minimum amount of resources that are requested by a pod 
- by default, containers have no limit on a pod, this can exhaust other pods 

How to create a resource request??
- add this to your container definition

```
containers:
  - name: test
    resources:
      requests: 
        memory: "2Gi"
        cpu: 2
```

CPU: 1 cpu is equivalent to the following 
  - 1 AWS vCPU 
  - 1 Azure vCPU

Memory:
  - 1G = 1,000,000,000 bytes 
  - 1Gi = 1,073,741,824 bytes 

Resource limits: this will limit the amount of memory and CPU that a container can consume 
- system cannot use more CPU than its limit
- but a pod can consume more memory, this will result in the pod being terminated with an OOM error

```
resources:
  requests:
    memory:
    cpu:
  limits:
    memory:
    cpu: 
```

LimitRange:
 - an object created at the namespace level 
 - this limits the amount of resources that new pods can gain 

quotas:
  - namespace level object to set hard limits for requests and limits out of all pods in a cluster


## DaemonSets 
- runs one copy of your pod on each node in the cluster 

use cases:
  - monitoring agents or log collectors 
  - kube-proxy
  - networking: tools that need to be deployed on each node 

how to create a daemonset

```
apiVersion:
kind:
metadata:
spec:
  selector:
    matchLabels:
  template:
    metadata:
      labels:
    spec:
      containers:
        - name:
          image: 
```
## Static pods 
- you can only create static pods, you cannot create replicasets or deployments 
- pods managed directly by the kubelet daemon, the API server cannot see these pods 
- the kubelet watches each static pod and will restart if failed 

How does the kubelet know what pods to create? 
- you specify a directory for the staticPodManifests when configuring the kubelet 
- put the manifests in that file and they will be recognized by the kubelet 

Configuring the designated folder 
- passed into the kubelet config 
- "--pod-manifest-path="

- create a file called kubeconfig.yaml 
- start the kubelet service with the "--config=kubeconfig.yml" argument
- inside kubeconfig.yml set the following "staticPodPath: /file/path"

why would you use static pods 
- deploy the control plane components 
- ex: api-server, etcd, controller-manager 
- you dont have to download binaries, configure services and worry about them crashing
- since it's a static pod, the kubelet will restart the services (you must configure the kubelet first)
- this is how the kubeadm tool works 

How to identify static pods?
- static pod names will always be appended with the node that its placed on
ex: "core-dns-controlplane"

## multiple schedulers
- you can write your own scheduler if you need some other logic or checks in place before your pods get placed on nodes
- you can have more than one scheduler at a time 
- schedulers must have different names

deploy an additional scheduler 

```
apiVersion: v1 
kind: Pod 
metadata:
  name: mycustomScheduler
  namespace: kube-system
spec:
  containers:
    - name: kube-scheduler
      image: k8s.grc.io/kube-scheduler-amd64
      command:
        - kube-scheduler
        - --address=127.0.0.1
        - --kubeconfig=/etc/kubernetes/scheduler.conf
        - --config=/etc/kubernetes/my-scheduler-config.yaml
```

leaderElect option:
  - if youre running multiple copies of the kubescheduler at once, if youre running multiple masters for HA 

how to use a new scheduler in a pod 

```
spec:
  container:
    - name:
      image:
  schedulerName: 
```

## Configuring Scheduler Profiles 

Scheduling Queue: 
  Scheduling plugins: sorts the pods in order based on the priority 

Filtering:
  NodeResourcesFit: filters nodes that dont have sufficient resources
  NodeName
  NodeUnschedulable

Scoring: 
  NodeResourcesFit: scores each node that is available 
  ImageLocality: high score goes to the nodes that already have the pod image 

Binding: 
  DefaultBinder: provides the binding mechanism 

Extension Points:
  - each stage there is an extension point where plugins can be plugged into 
  - there are extensions before and after the filter / score / and binding phase 
  
