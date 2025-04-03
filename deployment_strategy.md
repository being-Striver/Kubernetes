# Understanding K8s deployment strategy(deploying new application updates):
----------------------------------------
A deployment strategy is way to change or upgrade an application.How will you deploy v2 of your application to the k8s cluster?

Let's say you have a application v1 deployed on k8s cluster and it is running. after one week, the developer has created the new version of application with lot of new features.now the question is, how will you deploy this v2 of your application to the k8s cluster so that all of the production traffic goes to this specific version.so this is what is defined within deployment strategy.There is a lot of deployment strategy. for ex- lets say you go ahead and delete all the pods associated with V1 application and then you deploy v2 of your application.
 

# Deployment strategy- `recreate` 
--------------------------------
All of the PODS get killed all at once and get replaced all at once with new ones.
Recommended for dev/test.

-kubectl deployment recreate-strategy --image=nginx --dry-run=client -o yaml
              ```yaml
				apiVersion: apps/v1
				kind: Deployment
				metadata: 
				  creationTimestamp: null
				  labels:
					app: recreate-strategy
				  name: recreate-strategy
				  
				spec:
				  replicas: 3
				  selector: 
					matchLabels: 
					  app: recreate-strategy
				  strategy: 
					type: Recreate
				  template:
					metadata: 
					 creationTimestamp: null
					 labels: 
						app: recreate-strategy
						
					spec: 
					  containers:
					  - image: nginx
						name: nginx
						resources: {}
				status:{}
             ```
-kubectl explain deployment.spec.strategy
  --you will see two deployment strategy available(recreate and RollingUpdate).Default is RollingUpdate.
  
-kubectl apply -f recreate-strategy.yml
-kubectl get deployments -- you will see deployment go created
-kubectl get pods --you will see three pods running

what if we made some changes in recreate-strategy.yaml file?
 -let's say we change image from nginx to apache.

-kubectl apply -f recreate-strategy.yaml
but this time if you execute below command,
-kubectl get deployments
 you will see deployment is not ready.
but if you do,
 -kubectl get pods
you will see all new pods are there which are not in running state.
This is the challenge with Recreate strategy deployment.first it is deleting the version one of aplication that we know was working fine. if for some reason, this v2 application 
won't work in production, it will be a total mess.that's the reason, this is not recommended in prod env.

if you run below command,
kubectl get rs(replicaset)
you will see our first replicaset where desired was changed to zero and new replicaset gets created with desired count of three(where v2 of or new application is up and running).


second deployment strategy is RollingUpdate.
-------------------------------------------
Your application is deployed to your environment one batch of instances at a time.

in this time, only change in the above yaml file is, within strategy type should be RollingUpdate.

Within this strategy,
once the newer v2 application pod is ready and healthy, old one will get deleted one by one.


# Blue green deployment strategy:
---------------------------------
Blue-green deployment is a technique that reduces downtime and risk by running two identical production environments called Blue and Green.

You have a k8s cluster. You have a blue environment running the version one of the application and currently all of the production traffic is going to the version one of application.

Now what you do is, you create one more environment with version two of your application that is already running.So in order to deployment to take place, what you do is, go ahead
and connect your k8s cluster to the v2 of your application and all the  traffic now goes to v2.
Here points to note that at this stage v1 of your aplication is not deleted.This continues to run and in case let's say that there is some issue with version two, and then it is 
very easy to disconnect with v2 and connect the cluster to v1 of your application and traffic will flow again.

Lets look at the possible implementation here:
Possible implementation method 1:
------------------------------------

                  App V1(blue)
	kplabs-service 
	             App V2(green)
				 
				
	Lets assume we have kplabs-service and this service either can be associated with blue environment which is running the v1 of your application or green environment which 
	is running your v2 of your application.
	
	so this is how service definition would really look like.
	 apiVersion: v1
	 kind: Service
	 metadata: 
	   name: application-service
	 spec: 
	   type: NodePort
	   ports:
	   - name: http
	     port: 80
		 targetPort: 80
	   selector:
	     app: demo-app-v2
		 
-here demo-app-v2 is v2 of your application curently service is serving.
lets say for some reason you want to serve traffic back to v1 of your application, all you need to do is, you need to change selector app name with demo-app-v1 and traffic will
route back to version 1 of your application.


Example of blue-green deployment:
------------------------------------
1. Create 2 Pods

kubectl run blue-pod --image=nginx
kubectl run green-pod --image=httpd
2. Add Labels to both the Pods

kubectl label pod blue-pod app=demo-app-v1
kubectl label pod green-pod app=demo-app-v2
3. Verify POD labels

kubectl get pods --show-labels

4. Create Service

blue-green.yaml

apiVersion: v1
kind: Service
metadata:
 name: application-service
spec:
 type: NodePort
 ports:
 - name: http
   port: 80
   targetPort: 80
 selector:
   app: demo-app-v1
kubectl apply blue-green.yaml

5. Verify Service:

kubectl get service
kubectl describe service application-service
6. Fetch the Public Endpoint IP of the nodes

kubectl get nodes
kubectl describe node <node-name>
7. Change from Blue to Green Environment:

Modify the selector to demo-app-v2 in the service manifest file and re-deploy it.

kubectl apply blue-green.yaml

Second Way to Edit Service (Optional)

kubectl edit service application-service

Modify the selector to demo-app-v2



# Canary deploymment:
-----------------------
Canary deployment is a process where we deploy a new feature and shift some % of traffic to new feature to perform some analysis to see if feature is sucessful.
Suppose we have two deployment where one deployment is running version 1 of your application and second deployment is running version 2 of your application.Now  what you can do is
90% of your traffic you can send to v1  of your application and 10% of your trafffic you can send it to v2.
Canary deployment is not directly supported within k8s natively.However there are various external components that you will have to use in case if you want to implement this in 
a right way.
However, just to understand the approach, lets quickly look into the simplest way possible.

	1. Create Deployment Manifests:

	kubectl create deployment v1-app --image=nginx --replicas 3 --dry-run=client -o yaml
	kubectl create deployment v2-app --image=httpd --replicas 1 --dry-run=client -o yaml

	2. Store these manifests in v1-canary.yaml and v2-canary.yaml

	3. Add a common label of deptype: canary to both of these manifest files for the pod template section

	4. Create deployment in K8s

	kubectl apply -f v1-canary.yaml
	kubectl apply -f v1-canary.yaml
	5. Verify if PODS are created with appropriate labels:

	kubectl get pods --show-labels

	6. Create a Canary Service

	canary-svc.yaml

	apiVersion: v1
	kind: Service
	metadata:
	 name: canary-deployment
	spec:
	 type: NodePort
	 ports:
	 - name: http
	   port: 80
	   targetPort: 80
	 selector:
	   deptype: canary
	kubectl apply -f canary-svc.yaml

	7. Verify if Service have 4 Endpoint IPs:

	kubectl describe svc canary-deployment

	8. Make CURL request to see if requests are distributed among v1 and v2 apps

	curl PUBLIC-IP:NODEPORT


So canary deployment can be used in production with the help of service mesh and in aws with the help of route53.With the help of istio, you can use weight.

			kubectl apply -f - <<EOF
			apiVersion: networking.istio.io/v1alpha3
			kind: VirtualService
			metadata:
			  name: helloworld
			spec:
			  hosts:
				- helloworld
			  http:
			  - route:
				- destination:
					host: helloworld
					subset: v1
				  weight: 90
				- destination:
					host: helloworld
					subset: v2
				  weight: 10
			---
			apiVersion: networking.istio.io/v1alpha3
			kind: DestinationRule
			metadata:
			  name: helloworld
			spec:
			  host: helloworld
			  subsets:
			  - name: v1
				labels:
				  version: v1
			  - name: v2
				labels:
				  version: v2
			EOF
