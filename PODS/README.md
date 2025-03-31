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
When you pass `kubectl apply -f <pod-yaml-file>` , this request goes to API server. Then API server will check the ETCD whether this resource already exists or not. If it is already available, then it will response saying that this resource is already available.

Now let's say this resource doesn't exist, that information will store into the ETCD. Now it will go to scheduler to some prechecks like which node is suitable for this, after taking decision, it will send response to API server and that particular info will be stored in ETCD. Scheduler cannot talk to ETCD and ETCD cannot talk to scheduler. Once the creation info will pass to kubelet, kubelet will pass this info to containerd/ dockerd for container creation. once the container creation is done, this info will be passed to API server, and API server will pass the info to ETCD to store the information.


# Pod resource allocation CPU and memory
-----------------------------------------
volume path in pod: `/var/lib/kubelet/pods/<pod-uuid>/volumes/kubernetes.io~empty-dir/nginx-data`



# Pod multi-container with shared volume
----------------------------------------------

  #What is emptyDir volume?
  ---------------------------
  An `emptyDir` volume is a temporary directory created when a Pod is assigned to a node. As the name suggests, it's initially empty. All containers within the Pod can read and write to the emptyDir volume. The volume's lifetime is tied to the Pod's lifetime. When the Pod is removed from the node, the data in the emptyDir is permanently deleted.


  key characteristics:
  ---------------------
  1. Temporary Storage: Data persists only as long as the Pod exists on the node.
  2. Shared Access: All containers within the Pod can access the emptyDir volume.
  3. Node-Specific: The volume is local to the node where the Pod is running.
  4. Automatic Creation and Deletion: Kubernetes manages the creation and deletion of the volume.
  5. Storage Medium: By default, emptyDir volumes are backed by the node's disk. However, you can configure them to use the node's RAM for faster access

  use cases of emptyDir:
  -----------------------
  1. Scratch space: Providing temporary space for computations, sorting, or caching.
  2. Sharing Data Between Containers: Enabling containers within a Pod to exchange data.
  3. Checkpointing: Periodically saving the state of a long-running process.
 

 Storage medium: Disk vs Memory:
 --------------------------------
 By default, `emptyDir` volumes are backed by the node's disk. This is suitable for most use cases. However, for applications that require very low latency, you can configure an emptyDir to be backed by the node's RAM (memory).

 To specify that an emptyDir should use memory(node's RAM), you can set the `medium` field to `"Memory"` in the Pod's specification.

 Size limit:
 --------------
 It can be ignored in emptyDir case. It is of no use.


 Limitation:
 --------------------
 emptyDir is ephemeral in nature.
 No resource limit by default unless you specify using `size-limit`
 Storage medium is node-dependent.


# Access pod outside the cluster (hostPort)
-------------------------------------------------
In Kubernetes, a `hostPort` is a configuration option that allows you to expose a specific port on a node (the host) to a pod running in a Kubernetes cluster. Essentially, it binds a port on the node to a port on the container inside the pod, allowing external traffic to reach the pod via that node's port.

With hostPort you can expose container port to the external network at the address `<hostIP>:<hostPort>`, where the hostIP is the IP address of the Kubernetes node where the container is running and the hostPort is the port requested by the user. 
 

# Pod init containers
-------------------------
1. initContainers will run/create before app container defined under containers section.
2. initContainers will not be in running state, they will be completed state.
3. It works similar to actual containers, but each initContainer must complete successfully before next one starts.
4. if initContainer fail, kubelet restarts init container until it is succeeded.
5. initContainer donot support probes and lifecycle.
6. use cases:
    - to do some initial setup before actual applications come  up
    - to run some code/script as part of initialization before app gets created
    - to do some kernel changes for application specifics
  

# Pod restart policy
------------------------------
Pod restart policy will be defined on POD level not on container level.

restart policy:
 - Never
 - Always (Default)
 - OnFailure

# Static pod(controlled by kubelet)
------------------------------------
Application deployment using static pod:
 1. static POD will be managed by kubelet daemon on specific node
 2. static POD will not be controlled by kube-apiServer (kubectl or API calls)
 3. deleting static pod through API or kubectl will not effect the applications since kubelet will recreate it
 4. static POD names will be added with suffix of node name
 5. static POD manifest can be defined through FileSystem-hosted or web-hosted.
 6. kubelet can read default directory `/etc/Kubernetes/manifests`, if we place manifests file(YAML or JSON) to create static POD for filesystem-hosted approach.
 7. kubelet can download manifest file from website by passing `--manifest-url=<URL>` argument for web-hosted approach.

# Challenges in standalone pod
-------------------------------
1. POD is basic entity to run application on k8s cluster
2. POD works as standalone application in k8s cluster
3. POD are not bind to any specific node(unless we use nodeName, nodeSelector, nodeAffinity)
4. POD lifecycle is not managed by K8s controller
5. For application to be deployed with high availability, POD object is not a right solution.
6. In realtime environments of k8s cluster, we don't deploy application as standalone POD
7. We should always define containers in POD are application specific
8. Failed POd should be deleted by us through kubectl or API
9. If a Node dies, the pods scheduled to that node are scheduled for deletion after a timeout period.




