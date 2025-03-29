# What is pod?
----------------------
POD creates a `logical layer` to group one or more containers to have `common network` and `shared storage`.

POD has unique ip address assigned from `--pod-network-cidr=10.244.0.0/16` .
Containers inside the POD talk to each other localhost/127.0.0.1, since container shares same network stack.
Containers share data inside pod.
In general, we need to create containers inside POD which are dependent, not different application.


**NOTE**
-----------
- A pod always runs on a Node.
- A node is a worker machine in K8s.
- Each node is managed by the cluster.
- A node can have multiple pods.


# POD Lifecycle
-----------------------------
PODs have a defined lifecycle, from creation to termination. Understanding this lifecycle is crucial for managing applications effectively in Kubernetes. The lifecycle includes various phases, such as:

    - Pending: The POD has been accepted by the Kubernetes system, but one or more of the containers have not been created. This includes time before being scheduled as well as time spent downloading images over the network.
    - Running: The POD has been bound to a node, and all of the containers have been created. At least one container is running, or is in the process of starting or restarting.
    - Succeeded: All containers in the POD have terminated successfully and will not be restarted.
    - Failed: All containers in the POD have terminated, and at least one container has terminated in failure. That is, the container exited with a non-zero exit code or was terminated by the system.
    - Unknown: The state of the POD could not be determined for some reason, typically due to an error in communicating with the node where the POD ran.
  
Multi-Container PODs:
--------------------------
Multi-container PODs are useful when you have tightly coupled containers that need to share resources and communicate frequently. A common use case is the sidecar pattern, where a helper container runs alongside the main application container to provide supporting functionality, such as logging, monitoring, or proxying.

Example: A web application container and a logging agent container running in the same POD. The logging agent collects logs from the web application container and forwards them to a central logging system.


Use Cases for Multi-Container Pods:
----------------------------------------
- Sidecar Pattern: A sidecar container provides supporting functionality to the main application container, such as logging, monitoring, or proxying.
- Adapter Pattern: An adapter container transforms the output of the main application container into a format that can be consumed by other applications.
- Ambassador Pattern: An ambassador container acts as a proxy between the main application container and external services.

Best example of multi-container pod: data processing pipeline

Single container pod:
--------------------------
This is the simplest and most common scenario, where a POD contains a single container running your application.



Key Characteristics of Pods:
--------------------------------
- Co-location: Pods schedule containers together on the same node, enabling them to communicate efficiently.
- Shared Resources: Containers within a Pod share the same network namespace, IPC namespace, and volumes.
- Atomic Unit: Kubernetes manages Pods as a single unit. Scaling, replication, and updates are performed at the Pod level.
- Ephemeral Nature: Pods are designed to be ephemeral. They can be created, destroyed, and replaced as needed.

   +---------------------------------------------------+
   | NAME     |   READY |  STATUS  |  RESTARTS |  AGE  |
   -----------------------------------------------------
   | nginx-pod |   1/1  |   Running |  0       |   10s |
   +---------------------------------------------------+

1. NAME: The name of the POD (nginx-pod).
2. READY: Indicates how many containers in the POD are ready. 1/1 means that one out of one container is ready.
3. STATUS: The current status of the POD. Running means that the POD is running successfully. Other possible statuses include Pending, Succeeded, and Failed. We will discuss these in detail in a later module.
4. RESTARTS: The number of times the containers in the POD have been restarted.
5. AGE: The amount of time the POD has been running.

# create a pod
-----------------
- `kubectl apply -f nginx.yaml`

# delete a pod
----------------------
- `kubectl delete -f nginx.yaml`

# apiVersion
-----------------
The `apiVersion` field specifies the Kubernetes API version that is being used to create the object. Kubernetes is constantly evolving, and different API versions may offer different features or have different requirements. It's crucial to specify the correct apiVersion to ensure that your POD definition is interpreted correctly by the Kubernetes cluster.

# kind
-----------------
The `kind` field specifies the type of Kubernetes object that you are defining. In our case, we are defining a `POD`, so the kind will be set to Pod. Other possible values for kind include `Service`, `Deployment`, `ReplicaSet`, and `Namespace`.

# metadata
-----------------
The `metadata` field contains information about the POD, such as its `name`, `labels`, and `annotations`. This information is used to identify and manage the POD within the Kubernetes cluster.


# name
-----------------
The `name` is a unique identifier for the POD within its namespace. It's how you'll refer to the POD when using `kubectl` or other Kubernetes tools.

# labels
--------------------
`labels` are key-value pairs that are attached to the POD. They are used to organize and select PODs based on various criteria, such as `application name`, `environment`, or `version`.

# spec
------------------
The `spec` field defines the desired state of the POD. This includes the containers that will run within the POD, the volumes that will be mounted, and other configuration options. This is where you define the core functionality of your POD.

# containers
--------------------
The `containers` field is a list of container definitions that will run within the POD. Each container definition specifies the image to use, the ports to expose, and other configuration options.


# How to check logs of a pod
------------------------------------------
- `kubectl logs <pod-name>`

# how to check container logs if you have multiple container within pod
---------------------------------------------------------------------------
- `kubectl logs <pod-name> -c <container-name>`

# how to get detailed pod info
--------------------------------
- `kubectl get pod -o wide`

# Pod object creation workflow
--------------------------------
When you pass `kubectl apply -f <pod-yaml-file>` , this request goes to API server. Then API server will check the ETCD 
