# Virtualization with hypervisor:
------------------------------------
Hypervisor is a piece of software which creates virtual servers on top of physical servers.


# Container Orchestration
--------------------------------------
So far we have learned that you can run a single instance of the application with a simple docker run command. But it is just one instance of your app on one docker host. what happens when the number of users increase and that instance is no longer able to handle the load, you deploy additional instance of your app by running the docker run command multiple times. that is something you have to keep a close watch on the load and performance of your application and deploy additional instances yourself. Also you have to check health on your instances.
What about the host if its become inaccessible? The containers hosted on that host become inaccessible too.So what do you do in order to solve these issues?
You can build your own scripts and that will help you tackle these issues to some extent. Container orchestration is solution for that.
It is a solution that consists of a set of tools and scripts that can help host containers in a production environment. 

Typically container orchestration solution consist of multiple Docker hosts that can host containers.

Container orchestration solution allows you to deploy hundreds or thousands of instances of your application with a single command. This is a command used for docker swarm.


with kubernetes cli, which is kubectl(kube control), you can run thousands of instances of same application.
- `kubectl run --replicas=1000 my-web-server`

K8s can upgrade these 1000 instances of the application in rolling upgrade fashion one at a time with single command.
- `kubectl rolling-update my-web-server --image=web-server:2`

If something goes wrong, it can help you roll back these images with single command.
- `kubectl rolling-update my-web-server --rollback`

**NOTE**: K8s can help you test new features of your application by only upgrading a percentage of these instances through A/B testing methods.

# What is the relation between docker and k8s?
-----------------------------------------------
K8s uses docker hosts to host application in the form of docker containers. K8s clusters consists of set of nodes.
Node is a physical/virtual machine on which a K8s software a set of tools installed. A node is a worker machine and that is where container is launched by K8s.But what if the node on which the application is running fails?
well obvisouly our application  goes down so you need to have more than one nodes.A cluster is a set of nodes grouped together. This way even if one node fails, you have your aplication still accessible from other nodes.
Now we have a cluster, who is responsible for managing these clusters?
where is information about members of the cluster is stored?
How the nodes monitored when the node fails?
How do you move the workload of failed node to another worker node?
Thats where the master comes in.
The master is a node with K8s control panel component installed.The master watches over the nodes in the cluster and is responsible for the actual orchestration of  containers on 
worker nodes.



# K8s API primitive
-------------------------
Whenever you run a command in cli, cli will make an appropriate call to API depending upon what you want to do.if you want to create a pod, it will make a specific API call to create a pod. Depending on the operations you intend to do, there are various APIs which are available. So kubectl actually does the translation. As a user, we don't have to worry about it.

`kubectl proxy --port 8080`
>It will allows you to connect API server on the port 8080 on your localhost.

