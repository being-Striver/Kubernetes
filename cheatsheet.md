# Commands
------------------------
- To describe a pod
  : `kubectl describe pod <pod-name>`

- To delete the pod
  : `kubectl delete pod <pod-name>` 
  : You can specify `-f` option for force delete.

- To create pod from yaml file
  : `kubectl create -f <file-name.yaml>`

- To create replicaset
  : `kubectl create -f <replicaset-defintion.yaml>`

- To get replicaset
  : `kubectl get replicaset`

- To delete replica set (It will also delete underlying all pods)
  : `kubectl delete replicaset <replicaset-name>` 

- `kubectl replace -f replicaset-defintion` 

- `kubectl edit replicaset <replicaset-name>` -- it will automatically open the vi editor for you to edit the replicaset. once you edit and save it. make sure you have to delete all pods which was created before editing. once you delete old pod, new pod will be created automatically with new edit replicaset-defintion file.
	 
- `kubectl scale replicaset <replicaset-name> --replicas==5`  -- command for scaling up replicas(also for scaling down)

- `kubectl create namespace <namespace-name>` ---- way to create namespace

- manual way to create namespace:
   apiVersion: v1
   kind: Namespace
   metadata:
     name: dev
	 
	 
what if we want to switch dev namespace permanently so that we don't have to specify namespace anymore?
- `kubectl config set-context $(kubectl config current-context) --namespace=dev` 
then you simply run "kubectl get pods" command without specifying namespace to list pods in dev namespace.


to view pods in all namespace:
 - `kubectl get pods --all-namespaces`
 
- `kubectl config set-context $(kubectl config current-context) --namespace=dev`
 here:
   - `$(kubectl config current-context)` -- this will give current context.
   - and then sets the namespace to the desired one for that current-context.
   - Well, context are used to manage multiple clusters and multiple enviroment from the same management system.
   - To limit a resource in a namespace, create a resource quota.
   
   
   
#Resources within a namespace, simply refer to each other by their names.
  mysql.connect("db-service")  --if its in a same namespace
  if required, it can connect to other services in diffrent namespace by
    -mysql.connect("db-service.dev.svc.cluster.local")
	you are able to do this because when the service is created, a DNS entry is added automatically in this format.
	looking closely at the DNS name service, the last part(cluster.local), is the default domain name of the k8s cluster.
	svc is the subdomain of the service, followed by namespace and then the name of the service itself.
	
	
# The imperative commands:
-------------------------
`--dry-run=client` =>This will not create resource instead it will tell you whether the resource can be created and if your command is right.
`-o-yaml` => this will output the resource in yaml format.

**NOTE**: *Use the above two in combination along with Linux output redirection to generate a resource definition file quickly, that you can then modify and create resources as required, instead of creating the files from scratch.*
 
 - `kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml`
 
 Generate pod manifest YAML file(-o yaml). don't create it(--dry-run)
 - `kubectl run nginx --image=nginx --dry-run=client -o yaml`
 
 Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379:
 -kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml(this will automtically use pod's label as selector)
 -kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml(This will not use the pods' labels as selectors; instead it will assume selectors as app=redis)
 
 Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:
 -kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml



   Generate Deployment with 4 Replicas

  -kubectl create deployment nginx --image=nginx --replicas=4
  
  
 kubectl describe nodes --- it will give much sense about node.
 
 kubectl cluster-info --it will tell you where controlplane is running and where is coreDNS is running.
 kubectl cluster-info dump --to see further investigation at cluster level
 
 kubectl replace --force  -f nginx.yaml  --first it will delete the pod and replace with new one. (when you edit the pod definition file, this command can be used)
 kubectl get pods -o wide --- it will show more details
 kubectl get pods --watch --
 
 
 command to list every object within prod environment(including pods, replicasets, deployements, services etc)
 --kubectl get all --selector env=prod
 
 to apply multple filter condiitons:
  --kubectl get pods --selector env=prod --selector bu=finance --selector tier=frontend
  
  
  to apply taint on node01:
  --kubectl taint node <node-name> key=value:<taint-effect>
  
  How do we untaint a node?
  -kubectl taint node <node-name> key=value:<taint-effect>-  (adding a hyphen at the end will remove taint)
  
  
  kubectl taint node controlplane node-role.kubernetes.io/control-plane:NoSchedule-   => to untaint the controlplane node
  
  
  ERROR:
  json: cannot unmarshal object into Go struct field PodSpec.spec.tolerations of type []v1.Toleration
  
    --It happens when yaml file is not defined correctly. Here in this case, parameters should be passed in array under tolerations while i passing as mapping.
	
	
 apply a label on node:
 -kubectl edit node <node-name> --then edit the label 
 
 
 if you have a key defined on particular node, you can apply node affinity using operator "Exists" by specifying key only in affinity paramter like below:
    affinity:
	  nodeAffinity:
	    requiredDuringSchedulingIgnoredDuringExecution:
		  nodeSelectorTerms:
		    - matchExpressions:
			   - key: node-role.kubernetes.io/control-plane
			     operator: Exists
				 
	
  #The application stores the log at /log/app.log. view the logs.
   -kubectl exec <pod-name> -- cat /log/app.log

