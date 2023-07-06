# CKA - Storage 

Volume driver plugins
- default is local
- other ones are Azure file storage, clusterFS, portworx, vmware, convoy, rexray
- you can choose a specific volume driver when creating a container

#### Container storage interface

Container storage interface
- used to interact with multiple types of underlying storage 
- EX: allowing your kubernetes container runtime interface to work with underlying storage like AWS EBS


#### Volumes in kubernetes 
- helps to persist data inside of a container 
- allows containers in pods to be removed / recreated, and the data will still persist in a volume


#### Persistent Volumes 
- large pool of underlying storage that is cluster wide
- users can use the persistent volumes for applications with persistent volume claims 

VolumeBindingMode:
- WaitForFirstConsumer: the PVC will be associated to a PV once consumed by a pod

Access modes: defines how a volume is mounted on the host
- ReadOnlyMany
- ReadWriteOnce
- ReadWriteMany

capacity: the amount of storage for the persistent volume 

volume type:
- ex: hostPath

``` 
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
```


#### PersistentVolumeClaims
- admins create the PV which is the underlying storage 
- users use the PVC to claim the PV for their workload 
- there is a 1 to 1 mapping between a volume and a claim, so if you bind a claim to a PV that is way bigger than the claim, the rest of the storage is wasted
- if there are no PVs for claims, they stay in the pending state

Kubernetes will use the properties of a PV and a claim to find the best match for the claim.
ex: sufficient capacity, access modes, volume modes, and storage class

you can specify a binding using labels

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

What happens when the claim is deleted? 
- you can set the behavior that happens, by default its Retain (stays there until manually deleted)
- you can change it to Delete, and that will free up storage automatically
- recycle will make it available by scrubbing the previous data


#### StorageClasses 
- storage classes give administrators a way to define the type of storage to be used. storage classes also allow you to dynamically provision underlying storage for persistentvolumeclaims. a dynamic provisioner will automatically create the persistent volume without having to manually specify that. 

ex: you can create three storage classes which are silver, gold, and platinum. each of these can specify different storage types, but all the users need to know is that platinum will provision the highest performing storage 

Static provisioning vs dynamic provisioning 

- static: you must provision the underlying storage before you can start creating PV / PVC
- dynamic: using a provisioner, we can automatically provision the underlying storage, this is what storage classes are for. we no longer need the PV to be defined manually, in the PVC definition we select the storage class we are using, which would be something like AWS, GCP, or Azure 

