# Jobs: Kubernetes workloads
----------------------------------
- Jobs are workloads resources which will ensure one or more pods created and terminated successfully.
- In general we use jobs to run some batch operations like backups, Restore etc
- `workflow` : Jobs -> POD
- apiVersion for job is `batch/v1`
- job can also run multiple pod in parallel
- Once task specified in the POD container was successful, Job goes to completed status
- Job selector is optional
- PODs controlled by Job object restartPolicy should be set to only `Never` or `OnFailure`
- Types of parallel execution jobs:
   - Non-parallel jobs
   - parallel jobs with a fixed completion count
   - parallel jobs with a work queue
- It is always a best practice to enable ttlSecondsAfterFinished: 100 for job to cleanup automatically after the ttl value (`.spec.ttlSecondsAfterFinished`) which system will not put pressure on the API server
- We can suspend the Job as well and start again by `.spec.suspend` value to `true` or `false` 
  : Note: If the Job is suspended, any running POD that don't have status completed will be terminated.
  : Command: `#kubectl patch job/<job-name> --type=strategic --patch '{"spec":{"suspend": true}}'`




# Overview of batch jobs
--------------------------
A batch job is a collection or list of commands that are processed in sequence often without requiring user input or intervention.
Generally batch jobs accumulate during working hours, and then executed during the evening or another time when computer is idle.
batch jobs wait in a job queue for processing when the system has the available resources.


# creating first job in K8s
----------------------------
Job is responsible for creating one or more pods to run the instruction which have been specified.

In K8s,you have a job which is divided into two major categories:
-----------------------------------------------------------------
1. run for completion (job)
2. scheduled (cronjob)


```yaml
 apiVersion: batch/v1
 kind: Job
 metadata:
    name: kplabs-job
 spec: 
   template: 
     spec: 
	   containers: 
	   - name : kplabs-pod
	     image: busybox
		 command: ["/bin/sh"]
		 args: ["-c", "echo hello world"]
	   restartPolicy: Never
```


 - `kubectl apply -f jobs.yaml`
 - `kuebctl get pods` #job will ultimately create pod behind the scene.
 
 **Note: Job will not terminate the container.**
 kubectl get jobs
 **Note: if you delete the job, all the pods which were created, will also get deleted.**
 
 One important thing to discuss here is, restartPolicy. Once your job is completed, you no longer need pod to be up and running. Only two supported values for restartPolicy (Never and
 OnFailure).
 **In multiple-worker node cluster, if one node fails where job was running,that job will be start the pod in the available node.**
 
#K8s cronjob:
----------------
Cronjob allows us to run jobs based on a time-schedule.
CronJobs are useful for creating periodic and recurring tasks like running backups or sending emails.

```yaml
  apiVersion: batch/v1beta1
  kind: CronJob
  metadata: 
    name: kplabs-cronjob 
  spec:
    schedule: "*/1 * * * *"
	jobTemplate:
	  spec: 
	    template: 
		  spec: 
		    containers:
			- name: kplabs-pod
			  image: busybox
			  args:
			  - /bin/sh
			  - -c
			  - date; echo hello from k8s
			restartPolicy: OnFailure
```