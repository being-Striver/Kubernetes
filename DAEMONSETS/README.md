# Daemonsets(ds): Kubernetes workload resource
------------------------------------------------
- DaemonSets object makes sure that one POD will be running on each node( control and compute node).
- It doesn't support replica concept, for which scalein and scaleout feature is not available
- `Workflow` -> DaemonSet -> POD
- apiVersion for DaemonSet object is `apps/v1`
- since by default control plane doesn't launch workloads, we need to define tolerations under template
- `DaemonSet` object supports two selector parameters (matchLabels and matchExpressions)
- In order to manage PODs controller by DS, `.spec.selector` should match with `.spec.template.metadata.labels`
- we can still control the DS to launch POD on specific node with the help of nodeSelector and nodeAffinity
- DaemonSet supports rollout and rollback of PODs
- DaemonSet has two update strategies:
    - RollingUpdate(default): MaxUnavailable and MaxSurge
    - OnDelete 
- For DaemonSet, each revision is stored in a resource names ControllerRevision
   : `kubectl get controllerrevision -l ds-key=ds-value`



# Introduction
-----------------------
DaemonSets are like replicaSets as it helps you deploy multiple instances of pods. But it runs one copy of your pod on each node in your cluster.Whenever a new nodde is added to cluster, a replica of the pod is automatically added to that node. And when a node is removed, the pod is automatically removed. The DaemonSets ensure that one copy of the pod is always present in all nodes in the cluster.So what are some use cases of DaemonSets?
Say you want to deploy a monitoring agent or log collector on each of your nodes in the cluster,so you can monitor your cluster better.A DeamonSet is perfect for that as it deploy your monitoring agent in the form of pod in all the nodes in your cluster.Then you don't have to worry about adding or removing monitoring agents from these nodes when there are any chnages in your cluster as DaemonSet will take care of that for you.

We have seen that `kube-proxy` is k8s component which needs to be available in every worker node. So we can use concept of DaemonSet to deploy kube-proxy on every worker node.

Creating DaemonSet is similar to replicaSet creation process.It has nested pod specification under template section and selectors to link the DaemonSet to the pods.

- daemon-set-definition-file.yaml
```yaml
   apiVersion: apps/v1
   kind: DaemonSet
   metadata: 
      name: monitoring-daemon
   spec: 
     selector:
	   matchLabels:
	     app: monitoring-agent
	 template:
	   metadata: 
	     Labels:
		   app: monitoring-agent
	   spec:
	     containers:
		 - name: monitoring-agent
		   image: monitoring-agent
```		   
# to create a daemonSet using kubectl.
- `kubectl create -f daemon-set-definition-file.yaml` 

# to view the daemonSet
- `kubectl get daemonsets` 
  
# to view more details
- `kubectl describe daemonsets monitoring-daemon`

# to check status of rollout
- `kubectl rollout status ds <ds-name>`
  
  
# How does a daemonSet works? How does it schedule a pod on each node and how does it ensure that every node has one pod?
If you were ask to schedule a pod on every worker node in the cluster, how would you do it?
till v1.12, daemonset uses the NodeAffinity and default scheduler to schedule pods on nodes.