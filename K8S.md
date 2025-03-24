# Kubernetes:
------------------------------------------------------
It is most popular container orchestration tool.

We have so many containers in one docker engine. What if that docker engine fails? 
-Obvisiouly all the containers running in that docker engine go down.Users will not be able to access them.

How about high availability on them? 
-meaning launching multiple docker engines.

We should be doing clustering of docker engines in production environment. 
In cluster, we have one master node and multiple docker nodes.The master node will give instructions to all remaining docker nodes about containers to run.If any docker node fails,
master node automatically provision new node.
Basically master node is container orchestrator. master node distributes the containers among worker nodes or docker engines.
So all your docker nodes will be one single pool of resource which is fault tolerant.Container orchestartion is mainly done for production environment.

K8s manages the cluster of docker engines.

K8s provides:
	-service discovery and load balancing
	-storage orchestration
	-automated rollouts and rollbacks
	-automatic bin packing(it is going to replace your container on right node where it gets right resources)
	-self-healing 
	
Two main components of K8s architecture:
	-master node (manages the worker nodes)
	-worker node(where docker engines are running)
So you don't have to login into master node and tell it to run the container.you can connect it by some client.you give information to master node that I want to run so and so 
conatiners and it is going to take actions based on the requirements.So you have in master node which is also called control panel.you have following service in master node:
 1.API server
 2.Scheduler
 3.controller manager
 4.etcd

Worker node has three services:
 1.kubelet
 2.proxy
 3.docker engine

Master :Kube API server
 -Main hero! handles all the requests and enables communications across stack services
 -component on the master that exposes the K8s API
 -it is the front-end for K8s control plane
 -admins connect to it using Kubectl CLI
 -Web dashboard can be integrated with this API
 -and many more integrations
 
 
ETCD server (storage server):
 -key-value store
 -stores all the informations of K8s cluster
 -consistant and highly available key value store used as K8s backing store for all the cluster data.
 -Kube API stores and retrieves information from it.
 -should be backuped regularly
 -stores the current state of everything in the cluster
 
Kube Scheduler:
 -it schedules the container on right node.
 -watches newly created pods that have no node assigned, and selects a node for them to run on 
 -factors taken into account for scheduling decisions include
   -individual and collective resource requirements
   -hardware/software/policy constraints
   -affinity and anti-affinity specifications
   -data locality
   -inter-workload interference and deadlines
   
   
Controller manager:
  -logically each controller is a separate process
  -to reduce the complexity, they all are compiled into a single binary and run in a single process.
  -these controllers include:
      -Node controller : responsible for noticing and responding whe node goes down.
	  -replication controller: responsible for maintaining the correct number of pods for every replication controller object in the system.
	  -Endpoints controller: populates the endpoints objects(that is, joins services & pods)
	  -service account and token controllers: creates default accounts and API access tokens for new namespace.
	  
	  
Node components:

 kubelet : 
  - An agent that runs on each node in the cluster. It makes sure that containers are running in a pod.
  -this is going to listen K8s master request or commands.When the scheduler decides that this worker node is going to run the container, its going to assign the responsibility to
   kubelet so kubelet is going fetch your image and run container from it.so its going to do heavy lifting.
   
 Kube Proxy:
  - network proxy that runs on each node in your cluster.
  -you can set netwok rule like security groups.
    -rules allow network communication to your pods inside or outside of your cluster.
	
 Container runtime:
  -K8s support several container runtime.

Addons:
 -DNS
 -Web UI
 -container resource monitoring
 -cluster level logging
 
 
 
Containers enclosed in pod.
What is relation between container and pod?
-Its the same relation a VM has with the process running inside it.So let's say a Tomcat process is running in a VM so VM is going to provide all the resource to the process.Process
 just uses it.Similar way pod is going to provide all the resources to a container.The container will be running inside the pod.Container will be like process and pod will be like VM.
 

So there is no virtualisation here. only isolation.

Why does K8s uses pods? why not directly run containers?
-K8s can use diffrent runtime environement like docker, rocket, CRI.If you don't have pods,there will be no abstraction.
Now we have pod. It's a standard set of commands, standard set of configurations that we do, it doesn't matter what technology we are using behind.so pod gives us an abstraction.

Inside a pod, you can have one container or many containers.

Should we run multiple containers inside pod?
-Ideally we see one container  running inside the pod.Other container will be helper containers.

pod will be distributed accross multiple worker nodes.

K8s setup tools:
-------------------
MiniKube :
 -one node K8s cluster on your computer
 
kubeadm :
 -multi-node K8s cluster
 -can be created on any platforms vm's, ec2, physical machines etc.
 
kops :
 -multinode K8s cluster on aws
 

Now we gonna see how to setup K8s cluster using minuKube and Cops.

First lets see through MiniKube.
-open powershell as admin
-install chocolaty
-Install MiniKube with chocolaty 
choco install minikube kubernetes-cli 
-Open powershell and run 
minikube start

-when you put above command and hit enter , it is going to launch one single VM on your virtual box so you should have already oracle virtual box.thats it. cluster will be up and
 running.
 

ETCD for beginners:
------------------
objectives:
  -what is ETCD?
  -what is key-value store?
  -how to get started quickly?
  -how to operate ETCD?
  Later:
   -what is a distributed system?
   -How ETCD operates
   -RAFT protocol
   -best practices on number of nodes.
  
What is ETCD?
-It is a distributed reliable key-value store that is simple, secure and fast.
when data become more complex, you typically ending up transacting in data format like Json or YAML.Its easy to install and get started with ETCD.
1.Download the binaries
2.Extract
3.run ETCD service 

when you run ./etcd, it will start a service which listens to port 2379 by default.You can attach any client to the ETCD service to store and retrive information.
The default clinet comes with ETCD control client.ETCD control client is command line client for ETCD.

to store the key -value in ETCD command is below :
 -./etcdctl set key1 value1
 
to retrieve key value:
-./etcdctl get key1 



K8s Objects:

-pod
-Service
-Replica set
-deployment
-config map
-secret

KubeConfig File:
---------------------
When you execute your first Kubectl command, you wonder how does it connect to K8s cluster?How does it know where is my master node?
How does it get authenticated and get all information from the cluster or even create information in the cluster?

-well the answer is Kubeconfig file.
When we create cluster, by using kops or any other methods,we get a file called kubeconfig file.

Use kubeconfig files to organise information:
  1.clusters
  2.users
  3.Namespaces
  4.Authentication mechanisms
  
  
Namespaces:
how can you group or isolate your resources?
K8s cluster can have multiple namespaces. Thses namespaces will be used to set various kind of securities, quotas and few other things to your resources.
We have namespace called default, kube-system and kube-public when we create cluster.these namespaces get created automatically.



Pods:
A pod is a basic execution unit of a k8s application- the smallest and simplest unit in the K8s objects model that you create or deploy. A pod represents processes running on your
cluster.
  -pod that runs a single container:
    -one-container-per-pod model is most common K8s use case.
	-pod as a wrapper around a single container.
	-K8s manages the pods rather than the containers directly.
	
	
  -multi-container pod:
    - tightly coupled and need to share resources.
	  - one main container and other acts as a helper container (init container)
	  - Each pod is meant to run a single instance of a given application
	  - should use multiple pods to scale horizontally

 