# Kubernetes Security

### Security Primitives 

Secure Hosts:
 - you must secure the underlying hosts to your kubernetes cluster 

kube-apiserver:
  - you can perform almost any operation on the kube-apiserver, this is the first line of defense
  - you need to make sure to control access to the kube-apiserver with authentication / authorization

authentication mechanisms:
- useranme / password files 
- username / token files
- Certificates 
- LDAP providers 
- service accounts ( for machines )

Authorization mechanisms:
- RBAC authorization
- ABAC authorization 
- node authorization
- webhooks 


TLS certificats:
- used for authentication between the different kubernetes architecture components 

Network policies:
- used for securing communication between the workloads running on kubernetes


### Authentication 

Users - people who need access to the kubernetes cluster 
Service Accounts - internal bots or workloads that need to perform actions on the cluster or get information from the cluster

You cannot create users in Kubernetes but you can create service accounts


Users:
- all user requests managed through the kube-apiserver

  what are the method for authenticating to the kube-apiserver?
  - static password file 
  - static token file 
  - certificates 
  - identity services 

Static password / token files 
- you can create a csv file with three columns (password, username, userid) you can add a 4th column of group, then pass this as an argument to the kube-apiserver configuration 

``` --basic-auth-file=user-details.csv ``` 
- you must restart the kube-apiserver afterwards

Making a request to the kube-apiserver with basic authentication

``` curl -v -k https://master-node-ip:6443/api/v1/pods -u "user1:password1" ```  


### Basics of TLS certificates 

Certificate: ensure trust between two parties during communication 
  - certificates provide encryption that allows us to be safe from man-in-the-middle attacks 
  - asymmetric encryption: pair of keys is used to encrypt the data 

        private key: you keep this to yourself, this is the only thing that can decrypt things encrypted with your public key

        public key: anyone can access this key, used to encrypt the data that only you can decrypt with the private key

Certificate Authority: an issuer that validates your certificate to be valid. 

How does this process work? (public infrastructure)
 - generate a CSR ( certificate signing request )
 ex: ``` openssl req -new -key test.key -out test.csr -subj "/C=US/O=test.org" ``` 
 - the CA makes sure this is legit and they sign your certificate 
 - CA's have public and private key pairs, they use their private keys to sign, the public keys are built into the browsers, so they will be able to validate 

  
### TLS Certificates in Kubernetes

Public keys: usually .crt or .pem 
Private keys: usually .key -key.pem

- K8s clusters consist of master and worker nodes, the commmunication between them needs to be encrypted and secured 
- server certificates are used for inter-cluster communication between the different components 
- client certificates are used for users / external processes talking to the cluster

Server certificates for servers
- **kube-apiserver**: we need to generate a certificate for this componenet 
    ex: apiserver.crt and apiserver.key 
- **etcd-server**: stores all the information about a cluster 
    ex: ectdserver.crt and etcdserver.key
- **kubelet services**: these expose an HTTPS endpoint on the worker nodes that the kube-apiserver talks to 
    ex: kubelet.crt and kubelet.key 

Client Certificates for clients
- **admin**: authenticate to the kube-apiserver for administration
    ex: admin.crt and admin.key 
- **kube-scheduler**: talks to the kube-apiserver to look for pods that require scheduling, this is a client that accesses the kube-apiserver
    ex: scheduler.crt and scheduler.key
- **kube-controller-manager**: daemon that acts as a continuous loop in a kubernetes cluster. monitors the current state of the cluster via calls made to the API server 
- **kube-proxy**: network proxy that runs on each node in your cluster, this allows network communication to your pods from network sessions inside or outside your cluster

outlier:
- **kube-apiserver**: this component will talk with ETCD server, so ETCD views this as a client, you can generate new certificates for this communication, or use the current certificates 

We will need a certificate authority for our cluster to sign all of these certficiates, we will have one CA 
ex: ca.cert and ca.key

### Certificate Generation 

we are going to use OPENSSL, now we are going to generate our keys 

``` openssl genrsa -out ca.key 2048 ``` 

certificate signing request 

``` openssl req -new ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr ``` 

sign certificates 

``` openssl x509 -req -in ca.csr -signkey ca.key -out ca.cert ``` 


### Generate client certificates for the admin 

Generate the key

``` openssl genrsa -out admin.key 2048 ``` 

generate the CSR 

``` openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr ``` 

sign certificates 

``` openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt ``` 


We follow the same pattern for the other components

Client certificates:

Scheduler: certificate names must be prefixed with the word "system"
controller-manager: must be prefixed with keyword "system"
kube-proxy: must be prefixed with keyword "system"

Server certificates: 
- etcd: generate a certificate called etcdserver.crt, you must also generate another peer certificates when deploying ETCD in an HA setup across multiple nodes in a cluster
    - when enabling etcd, you must provide the following configuration options

    ```
    --key-file
    --cert-file
    --peer-cert-file
    --peer-client-cert
    --peer-key-file
    --peer-trusted-ca-file
    --trusted-ca-file #this is the certificate authority file 
    ```

- kube-apiserver: we need to generate a certificate called apiserver.crt and apiserver.key, all operations go through the kube-apiserver, this can be called kubernetes, kubernetes.defualt, or kubernetes.defualt.svc.cluster.local, so all these names need to be present in the certificate file 

in the csr, you need to create an openssl.cnf file that holds all of these different names and IP addresses

example: 

``` openssl -req -new -key apiserver.key -subj "/CN=kube-apiserver" -out apiserver.csr -config openssl.cnf ```

``` 
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation,
subjectAltName = @alt_names
[alt_names]
DNS.1 = kubernetes 
DNS.2 = kubernetes.default 
DNS.3 = kubernetes.default.svc 
DNS.4 = kubernetes.default.svc.cluster.local
IP.1 = 10.96.0.1
IP.2 = 172.17.0.87 
```

``` openssl x509 -req -in apiserver.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out apiserver.crt -extensions v3_req -extfile openssl.cnf -days 1000 ``` 

files that are needed when configuring the kube-apiserver service

```
--etcd-cafile
--etcd-certfile
--etcd-keyfile
--kubelet-certificate-authority
--kubelet-client-certificate
--kubelet-client-key
--client-ca-file=
--tls-cert-file
--tls-private-key
```

- kubelet server: installed on worker nodes, this is where the kube-apiserver talks with the node, you need a keypair for each node in the cluster (node01.crt / node01.key), kubelet also needs client certificate to authenticate to the kube-apiserver (system:node:node01) and the nodes need to be in a group called system:nodes



How do you use these certificates? 
- move them all into a kubeconfig file 
- use them when making API requests 


#### How to view certs in an existing cluster?
- first, it's important to now how the cluster was configured 

"the hard way"
- all the kubernetes components are configured as services


kubeadm
- all the kubernetes cluster components are configured as 

1. start by gathering the location and name of all the certficiates 
2. look inside each certificate 
    ``` openssl x509 -in /cert/path/apiserver.crt -text -noout ``` 
3. find the subject names, alternative names, and the organization, and expiration
    - the certificate requirements are specified in the kubernetes documentation
4. if using kubeadm you can read through the logs with docker logs 



#### Certificate API 
- kubernetes has a builtin certificates API, you can send a CSR directly to kubernetes 
- all CSR's can be seen by all admins in the cluster, you use kubernetes commands to approve 


How does it work?

1. user generates a key 

``` openssl genrsa -out test.key 2048 ``` 

2. user generates a certificate signing request 

``` openssl req -new -key test.key -subj "/CN=user" -out test.csr ``` 

3. admin creates a csr object using a manifest file in kubernetes 

``` 
apiVersion: certificates.k8s.io/v1beta1
kind: CertficiateSigningRequest
metadata:
  name: user
spec: 
  groups:
    - system:authenticated
  usages:
    - digital signature
    - key encipherment 
    - server auth 
  request: 
    // cat test.csr | base64 | tr -d "\n" 

    base64 encoded values 
``` 

4. approve the cert 

``` kubectl certificate approve user ``` 

5. share the generated cert with the user 

``` kubectl get csr user -o yaml ``` 

- copy the certificate 

``` echo "cert" | base64 --decode ```


#### Kubeconfig

what makes up the kubeconfig file?
- clusters: could be dev, production, or a cloud provider
- contexts: define which user accesses which cluster
- users: defines what users are going to be used 


kubeconfig file is in a yaml format 

```
apiVersion: v1 
kind: Config

clusters:

contexts:

users:

```

- viewing the kubeconfig information 

``` kubectl config view ``` 

- how to update the context you're using 

``` kubectl config use-context prod-user@production ``` 

- how to use a context in a different config file 

``` kubectl config use-context test --kubeconfig=/path/to/file ``` 

how to configure a context to switch to a namespace
- specify a namespace in the kubeconfig file 

Certificates in kubeConfig 
- you can use the cert authority data field and encode the base64 values 

#### API groups 

There are different API endpoints that serve different purposes in k8s: 
- /api 
- /apis
- /logs  
- /version  
- /healthz 
- /metrics 


Two categories of API 
- core: core consists of all functionality for the cluster exists, configmaps, secrets, replicaSets, etc.
- named: more organized, breaks the API functionality into smaller groups 
    - /apps 
      - /v1
         - /deployments
         - /replicasets
         - /statefulsets
    - /networking.k8s.io
      - /v1
        - /networkpolicies 
    - /storage.k8s.io
    - /certificates.k8s.io
      - v1
        - /certificatesigningrequests

NOTE: 
 - if you're going to use curl for accessing your cluster, you need to pass in the --key, --cert, and --cacert 
 ``` curl http://localhost:6443 -k --key admin.key --cert admin.crt --cacert ca.crt ``` 

 - you can also start a proxy with kubectl 
 ``` kubectl proxy ``` 
 - this will open up a port that will use your kubeconfig credentials, so you can curl like so 
 ``` curl http://localhost:8001 -k ``` 


#### Authorization 

Authorization Mechanisms 
- Node Authorizer: makes sure that nodes have the correct permissions to report the data related to pod. etc.
- ABAC: attribute based authorization, difficult to manage, you associate a user or group directly with permissions and must restart the kube-apiserver everytime  
- RBAC: role based access control, you create a role with the required permissions, and associate the users/groups to the role, MORE STANDARD APPROACH 
- Webhook: outsource all authorization through external provider, using something like OPA, offloading authorization to third party 

Authorization mode:
- the mode is set on the kube-apiserver, default it's always allow 
- when you set multiple modes, the request is passed in the order that you specified, ex: you set Node, RBAC, and Webhook to be the authorization modes on the kube-apiserver
  - your request is first handled by the node authorizer, so it denies the request and forwards it to the next one in the chain, which is the RBAC authorizer


#### RBAC
- creating roles is done through the kubernetes API 
- you then can create a roleBinding that will set who gets which Role 

How do you view rbac?
- k get roles 
- k get rolebindings
- k describe role $rolename
- k describe rolebinding $rolebindingname 

Check access 
- k auth can-i create deployments
- k auth can-i delete nodes 
- k auth can-i create deployments --as dev-user
- k auth can-i create pods --as dev-user


``` 
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get", "watch"]
```

```
apiVersion: rbac.authorization.k8s.io/vi
kind: RoleBinding
metadata:
  name: dev-user-binding
subjects: -> who the role is going to be applied on 
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef: -> the role that we are going to apply to the subject
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```


#### Cluster Roles

resources can either be namespaced or cluster scoped
- namespaced: resources are bound to a specific namespace 
- cluster scoped: you cannot bind this resource to one namespace, they apply to the entire cluster, here are a few cluster scoped resources:  nodes, PV, clusterroles, clusterrolebindings, certificatesigningrequests, namespaces

these operating very similar to roles, you also use a cluster role binding 


#### Service Accounts 

Service accounts in kubernetes can be used for applications, or third partys to connect to the kubernetes API 
- you can generate tokens for your kubernetes API objects using the tokenRequestAPI, these tokens are audience bound, time bound, and object bound
- creating a service account doesn't automatically create a secret, or a token


creating a service account 

``` kubectl create serviceaccount dev-account ``` 


What if you are running an application that needs to authenticate to kubernetes, in kubernetes?
- you can mount the secret token and make it available to the application as a volume, inside the pod
- the token will already be placed inside the pod and read 

the default service account:
- automatically created in each namespace 
- the default service account and token are automatically placed in all pods 


#### Image Security 

when working with private registries, how do we authenticate kubernetes to pull images?
- create a secret object with the information to authenticate
- specify the imagePullSecrets: under the containers definition in the manifest

