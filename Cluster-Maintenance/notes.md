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

