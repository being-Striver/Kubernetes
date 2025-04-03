# Deployment: K8s workloads resource
-----------------------------------------------------
- Deployment object has same feature that replicaSet has provided.
- In addition to that, deployment can rollout and rollback applications.
- `Workflow`: Deloyment -> ReplicaSet -> POD
- Features: ScaleIn and ScaleOut, Rollout and Rollback, pause rollout/rollback, remove older RS
- By default, replicaSet will not get deleted by your deployment whenever you rollout a rollback
- apiVersion for deployment: `apps/v1`
- When deployment manages replicaset, donot modify pod-template-hash label.
- Rollout and Rollback can be performed through imperative(recommended) or (Declarative) approach.
- We can check the number of rollouts done for a deployment by executing command.
  - `kubectl rollout history deploy/<deployment-name>`
  - `kubectl rollout undo deploy/<deployment-name> [--to-revision=<revision-number>]`
- All the revisions will have unqiue POD template.
- Deployment supports two strategies: `RollingUpdate`(defualt) and `Recreate`(.spec.strategy.type) 
- deployment revision history limit can be set under `.spec.revisionHistoryLimit`(default: 10).


# Command to see all three objects of deployment at the same time
---------------------------------------------------------------------
- `kubectl get deployment, rs, pod -o wide`

# Challenges with ReplicaSet
--------------------------------------
Before we go into deployments, lets see the challenges with replicaSets.
ReplicaSets works well in basic functionality like managing pods, scaling pods and similar.However if you want certain important functionality related to rolling out changes, rolling back changes and so on, then you need to go with deployments.

# Understanding deployments:
-------------------------------------
Deployments provide replication functionality with the help of replicasets, along with various additional capability like rolling out of changes, rollback changes if required.

# Benefits of deployments- rollout & rollback changes
----------------------------------------
We can easily rollout new updates to our application using deployments.Deployments will perform update in rollout manner to ensure that your app is not down. means It will spin up new pod with rollout changes and once it is running, the older pods will get self-deleted.

  #How to check deployment
  - `kubectl get deployments`

  #Lets look at the rollout history associated with the deployment.
  - `kubectl rollout history deployment.v1.apps/kplabs-deployment`


Lets create our first deployment:
---------------------------------
```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: kplabs-deployment
    
  spec:
    # modify replicas according to your case
    replicas: 3
    selector:
      matchLabels:
        tier: frontend
    template:
      metadata:
        labels:
          tier: frontend
      spec:
        containers:
        - name: php-redis
          image: gcr.io/google_samples/gb-frontend:v3
```
# create deployment from yaml file
------------------------------------------
- `kubectl apply -f deployments.yaml`

- `kubectl get deployments` 
- `kubectl get replicaset`  #This command will show you replicaSet created and managed under deployment which is managed by deployment.
- `kubectl get pods` #this will show you 3 pods running which is created and mananged  by replicaset.

- Is there a way we can change the scaling of replicaSets?
Yes.There is something called RollingUpdateStrategy when you type `kubectl describe deployment <deployement-name>`.
So you can change the way in which scaling up and scaling down of replica would happen.


When you run `kubectl rollout history deployment.v1.apps/kplabs-deployment`, It basically maintains the revision associated with your changes.

`kubectl rollout history deployment.v1.apps/kplabs-deployment --revision 1` #it will give you details about revision 1.

**NOTE: Revision is a great functionality**.

# Important pointers:
---------------------
1. Deployment ensures that the only a certain number of pods are down while they are being updated.
2. By default, it ensures that at least 25% of the desired number of pods are up(25% max unavailable)
3. Deployments keep the history of revision which has been made.



# Rolling back deployments:
-----------------------------
`kubectl rollout undo deployment.v1.apps/kplabs-deployment --to-revision=1` # we have specified the revision number to which we want to rollback.


# Generate deployment via CLI
------------------------------
challenge with deployment Manifest:
-------------------------------------
To create a simple deployment, we have to search for the entire deployment manifest file from the documentation and edit it based on our requirements.


Creating deployments via CLI:
We can run `kubectl create deployment` set of commands to create a deployment from CLI.

- `kubectl create deployment my-dep --image=nginx`   #creates a new deployment 
- `kubectl create deployment my-dep --image=nginx --replicas 3`   #deployment with three replicas 
- `kubectl create deployment my-dep --image=nginx --replicas 3 --dry-run -o yaml` #this will give us manifest and it will not create the deployment.
- `kubectl create deployment --help` # very useful command 
  
# deployment related options (maxSurge and maxUnavailable)
------------------------------------------------------------
- Overview of deployment configuration:
 while performing a rolling update, there are two important configuration to know.
 configuration parameter                              Description
 `maxSurge`                             maximum number of PODs that can be scheduled above original number of pods.
 `maxUnavailable`                       maximum number of pods that can be unavailable during the update 
 
 
# Example use case:
--------------------------
 Total pods in deployment 8.
  maxSurge -25%
  maxUnavailable- 25%
  
  maxSurge=2 pods 
  maxUnavailable=2 pods
  At most of 10 pods(8 current pods +2 maxSurge pods)
  at most of 6 pods (6 current pods +2 maxUnavailable pods)
  
 **NOTE: you can define maxSurge and maxUnavailable in both numeric and percentage terms.**
 **Note: It might be a scenario where your worker node cannot handle more than 3 pods. In such case , define these parameters accordingly.**
 
 Scenario:
 maxSurge =0
 maxUnavailable=25%
 means, deployment rollout will happen since you have specified maxSurge=0, you won't see any new pod created on top of existing ones.
 Instead of creating new pod, it will delete existing one with new one.In such kind of process, only few pods are running while remaing under update process,those running are serving the traffic.
 
# 4 important pointers for deployments:
----------------------------------------
1. You should know how to set a new image to deployment as part of rolling update.
2. You should know the importance  of --record instruction.
3. You should know how to rollback a deployment.
4. You should be able to scale up the deployment.


- `kubectl set image deployment/my-deployment nginx=nginx:1.91 --record` #set image
-`kubectl scale deployment/my-deployment --replicas 20` #scale deployment
- `kubectl rollout undo deployment/my-deployment` #rollout undo 


Whenever you run below command, you won't get much details.
- `kubectl rollout history deployment kplabs-deployment`

In order to have some information while performing rollback, also provide record instruction.

- `kubectl set image deployment/my-deployment nginx=httpd --record` now when you run `kuebctl rollout history deployment my-deployment` now you will see exact command which was used in specific revision.




