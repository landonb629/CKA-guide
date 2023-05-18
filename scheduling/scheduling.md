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