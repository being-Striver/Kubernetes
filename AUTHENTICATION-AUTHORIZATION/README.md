# Object creation workflow in api server
------------------------------------------


`API request` --> `Authentication & Authorization` -> `Mutating admission` -> `Object schema validation` -> `Validating admission` -> `Persisted to ETCD`

Authentication means It will verify who you are.
Authorization means what you can access.


# Introduction
-------------------
Let's begin with host that form the cluster itself.
-----------------------------------------------------
Offcourse, all access to these host must be secured(like root access disabled, password based authentication disabled, only SSH key based authentication available).Here our focus is more on k8s security.
 
What are the risks and what measures do you take to secure the cluster?
---------------------------------------------------------------------------
As we know, kube-apiserver is at the center of all operations within k8s. We interact with it through the kube control utility or by accessing the api directly.Through that you can perform any operations on the cluster.So that's the first line of defense(controlling access to the API server).
  
We need to make two types of decisions:
   - who can access the cluster?
   - what can they do?
   
Who can access the cluster, is defined by the authentication mechanism.There are diffrent ways you can authenticate to the API server, starting with user IDs and passwords stored in static files or tokens, certificates or even authentication with external authentication providers like LDAP.
Finally for machines, we create service account.

Once they gain access to the cluster, what they can do?
it is defined by authorisation mechanism. Authorisation is implemented using role based access(RBAC) control where users are associated to groups with specific permssion.In addition there are other authorisation modules like ABAC(attribute based access control), node authorisation, webhook mode.
   
All communication with the cluster between various components such as ETCD cluster, the kube-control api, manage scheduler, API server as well as those running on the worker nodes such as the kubelet and kube-proxy is secured using the TLS encryption.


Authentication:
-------------------
1. cert
2. Tokens
3. username/password

Authorization modes:
---------------------
1. RBAC
2. ABAC
3. node
4. Webhook

# K8s api-server authentication strategies
---------------------------------------------
- k8s supports two categories of users:
  - service account managed by k8s
  - normal users
- Normal users have different methods to authenticate like certs, passwords and token bases
- service account only provides token based authentication
- from v1.24, creating serviceAccount will not create secret token.
- we can create token by executing command # `kubectl create token` command.


# Understanding kuebconfig file
---------------------------------------------
command : `kubectl config view`



