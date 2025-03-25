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


