## Storage in k8s:

Let's understand the storage in docker:
------------------------------------------
will talk about docker storage drivers and file systems.
Let's see how docker store file system on the local file system.

 -When you install docker on your system, it creates the folder structure at /var/lib/docker.
   - `/var/lib/docker`
      - `aufs`
      - `containers`
      - `image`
      - `volumes`
       
This is pretty much folder architecture and where docker stores all its data by default. When I say data means files related to images and containers running on the docker host.All files related to containers are stored under the containers folder. Lets understand how docker stores these files and in what format?

How does docker actually stores the file of image and containers?
-----------------------------------------------------------------
To understand that we need to understand `docker's layered architecture`.When docker builds images, it build these in a layered architecture. Each line of instructions in the dockerfile creates a new layer in the docker image with just the changes from the previous layer.Since each layer stores the changes from previous layer, it reflects in size as well. once the build is complete,you cannot modify the contents of these layers and so they are read only as you can only modify them by initiating a new build.When you run container based off this image using the docker run command, docker creates a container based off of these layers creates a new writable layer on top of image layer.The writable layer is used to store data created by the container such as log file written by the applications, any temporary files generated by the container or any file just modified by the user.
The life of this layer though is only as long as the container is alive. When the container is destroyed, this layer and all of the changes stored in it are also destroyed.
Remember same image layer is shared by all containers created using this image.

- `container layer` -- `read write layer` 
- `image layer`  -- `read only` (app.py)(files in this layer will not be modified in the image itself.)

what if you want to modify source code of image in the containers? is it possible?
-----------------------------------------------------------------------------------
Yes it is possible.When I saved the modified file, docker automatically creates a copy of file in the read-write layer and then i will be modifying a different version of the file in the read-write layer.All future modifications will be happen in this copy of file in the read-write layer.This is called copy on write mechanism.The image layer being read only just means that the files in these layers will not be modified in the image itself.so the image will remain the same all the time until you rebuild image using the docker build command.
   
What happens when we get rid of containers?
---------------------------------------------
All the data will stored in the containers layer also get deleted.The change in app.py and the new temp file we created will also get deleted.

So what if we wish to persist this data?
----------------------------------------
We could add a persistent volume to the container.To do this, first create a volume using docker volume command.
- `docker create volume data_volume` (it will create folder within volume folder)

- `docker run -v data_volume:/var/lib/mysql mysql` 
	Here,
	 `/var/lib/mysql` is the location inside my container which is the default location where my mysql stores the data(/var/lib/mysql) &
	 mysql is the image name.above command will create a new container and mount the data volume we created into /var/lib/mysql folder inside the container so all the data written by the database is in fact stored on the volume created on the docker host.Even if container is destroyed, data is still active.
	 
	 
What if our data is at another location?
----------------------------------------
lets say we have some external storage on the docker host at `/data` and we would like to store database data on that volume and not in the default `/var/lib/docker` volume folder.
 in this case we run below command providing complete path.
  - docker run -v /data/mysql:/var/lib/mysql mysql 
    It will create a container and mount the folder to the container.This is called bind mounting.
	
 Note: `-v` is pretty much old method. New way is to use --mount option and we pass each parameter in key-value format.
 for example, previous command can be written as:
   -- docker run --mount type=bind, source=/data/mysql, target=/var/lib/mysql mysql 
   
So who is resposible for doing all these operations?
-----------------------------------------------------------
maintaining the layered architecture, creating a writeable layer, moving files across layers to enable copy and write etc. Its the storage drivers.Docker uses storage drivers to enable layered architecture.
Some of the common storage drivers are:
     - AUFS
	 - ZFS
	 - BTRFS
	 - Device Mapper
	 - Overlay
	 - Overlay2
	 
The selection of storage driver is based on the OS being used underlying.for ubuntu, default storage driver is AUFS.Storage drivers help manage storage on images and containers.
Remember volumes are not handled by storage drivers.volumes are handled by volume driver plugins.The default volume driver plugins is local.
	
When you run a docker container, you can choose to use a specific volume driver such as `REX-ray EBS` to provision a volume from amazon EBS.
- `docker run -it \
	    --name mysql
		--volume-driver rexray/ebs
		--mount src=ebs-vol, target=/var/lib/mysql
		mysql`
		
This will create a container and attach a volume from the aws cloud.	
	
	
Container storage interface(CSI):
------------------------------------
The container storage interface is a standard that defines how an orchestration solution like k8s would communicate with container runtime like docker.so in the future if any new container runtime interface is developed, they can simply follow CRI standards and that new container runtime would work with k8s without having relly work with k8s developers or touch the source code.

Similary to extent support different networking solutions, container networking interface (CNI) was introduced.Any new networking vendors could simply develop their plugin on the CNI standards.
CSI was developed to support multiple storage solutions. With CSI, you can write your own drivers for your own storage to work with k8s.

Volumes in K8s:
---------------
The pods created in k8s are transient in nature.When a pod is created to process a data and then deleted, the data processed by it gets deleted as well.For this we attached a volume to the pod.The data generated by the pod is now stored in the volume and even after the pod is deleted, the data remains.
lets look at the simple implementation of volumes.

We have a single node k8s cluster.We create a simple pod which generates a random number.
syntax:
-------------
```yaml
   apiversion: v1
   kind: Pod
   metadata: 
     name: random-name-generation
   spec:
     containers:
     - image: alpine
       name: alpine
       command: ["/bin/sh", "-c"]
       args: ["shuf -i 0-100 -n 1 >>/opt/output.txt;"]
	   volumeMounts:
	   - mountPath: /opt
	     name: data-volume 
	   
	 volumes:
	 - name: data-volume
	   hostpath: 
	     path: /data
		 type: Directory
```		 

To retain the random number generated by pod, we create a volume and a volume needs a storage.When you create a volume, you can choose to configure its storage in diffrent ways.
For now we simply configure it to use a directory on the host.In this case, we have specified the path "/data" on the host.

```yaml
     volumes:
	 - name: data-volume
	   hostpath: 
	     path: /data
		 type: Directory
```

This way, any files created in the volume would be stored in the directory data on the node. Once the volume is created, to access it from container, we mount the volume to a directory inside the container.We use the volume mounts field in each container to mount data volume to the directory `"/opt"` within the container. 
```yaml
      volumeMounts:
	   - mountPath: /opt
	     name: data-volume 
```

We just use the host path option to configure it directly on the host as storage space for volume.Now that works on single node. However it is not recommended for use in multi-node cluster.This is because the pods would use the "/data" directory  on all the nodes and expect all of them to be same and have the same data. Infact they are not same since they are on diffrent servers unless you configure some kind of external replicated cluster storage solution. 


Persistant volume:
----------------------
In the last we have learned about volumes. When we create volumes, we configure volumes within the pod defintion file so every confguration information required to configure storage for the volume goes within the pod definiton file.Now we have a large environment with lot of users deploying a lots of pods, the users would have to configure storage everytime for each pod.Whatever storage solutions is being used, user who deploys the pods would have to configure that on all pod defintion file in his environment. Instead of that, you would like to manage storage more centrally.You would like it to be configured in a way that an adminstrator can create a large pool of storage and then have users carve out pieces from it as required.That is where persistent volumes can help us. 
`A persistent volume is a cluster wide pool of storage volumes configured by administrator to be used by users deploying applications on the cluster.The users now can select storage from this pool using persistent volumes claims(PVC)`.

Let's now create a persistent volume.
-------------------------------------------
pv-defintion.yaml

```yaml
   apiversion: v1
   kind: PersistentVolume
   metadata: 
      name: pv-vol1
   spec: 
      accessModes:
        - ReadWriteOnce
	  capacity:
	     storage: 1Gi
	  hostPath:
	     path: /tmp/data 
``` 		 


Here accessModes defines how a volume should be mounted on the hosts, whether in a read-only mode or read-write mode etc.The supported values are:
   - ReadOnlyMany
   - ReadWriteOnce
   - ReadWriteMany
   
Next is `capacity`. Capacity specify the amount of storage to be reserved for this persistent volumes which is set to be 1gb.Next comes is `volumeType`.We specify the `hostPath` option that uses the storage from the node's local directory.Remember above option in yaml file is not to be used in production environment.
 
To create the volume, run the below command.
 - `kubectl create -f pv-defintion.yaml`
 
Replace the host path with supported storage solutions.
```yml
  awsElasticBlockStore:
    volumeID: <volume-id>
	fsType: ex 
```

Persistant volume claims:
---------------------------
Now we will try to create PVC to make storage available to a node.
persistent volumes and persistent volume claims are two separate objects in the k8s namespace.
An administrator creates a set of persistent volumes and users create peristent volume claims to use storage.
Once the persistent volume claims are created, k8s binds the persistent volumes to claims based on the request and properties set on the volume.Every persistent volume claim 
is bound to a single persistent volume.During the binding process, k8s tries to find a persistent volume that has suffiecient capacity as requested by the claims and any other 
requests property such as accessModes, volumeModes, storage Class etc  

However if there are multiple possible matches for a single claim, and you would like to specifically use a particular volume, you could still use label and selectors to bind 
to right volumes.
Finally note that a smaller claim may get bound to a larger volume if all other criteria matches, and there are no better options. There is one-to-one relationships between 
claims and volume.so no other claims can utilize the remaining capacity in the volume.If there are no volumes are available, the persistent volume claim will remain in pending state
until newer volumes are made available to cluster.once the newer volumes are availabe, claims automatically bound to the newly available volumes.

let us now create a persistent volume claims.

  pvc-definiton.yaml
   apiVersion: v1
   kind: PersistantVolumeClaim
   metadata:
     name: my-claim
   spec:
     accessModes:
         - ReadWriteOnce

     resources:
         requests: 
           storage: 500Mi

To create the claim, use below kubectl command
-kubectl create -f pvc-definiton.yaml

To get pvc
-kubectl get persistentvolumeclaim	

When the claim is created, k8s looks at the volumes created previously.k8s looks at the accessMode and requests. Since storage is not matched but volumes storage is higher than claim
it can bound to claim since no other volume is available.

-kubectl get persistentvolumeclaim
  NAME    STATUS      VOLUME     CAPACITY      ACCESS MODES       STORAGECLASS     AGE
 my-claim  Bound     pv-vol1     1Gi            RWO                                 43s


Delete pvc:
----------------
-kubectl delete persistentvolumeclaim my-claim

but what happens to the volume under pvc?
You can choose what happens to the volume.
 persistentVolumeReclaimPolicy: Retain - by deault
 
It means persistent volumes retains until it is deleted manually.

 persistentVolumeReclaimPolicy: delete (means as soon as claim is deleted, volume will get also deleted).
 
 

Storage class:
-------------------
 in the last lecture, we have learned how to create volumes(pvs) and  claims(pvc) to claim that storage.then use the pvc in the pod definition file  as volume.

   
   apiversion: v1
   kind: Pod
   metadata: 
     name: random-name-generation
   spec:
     containers:
     - image: alpine
       name: alpine
       command: ["/bin/sh", "-c"]
       args: ["shuf -i 0-100 -n 1 >>/opt/output.txt;"]
	   volumeMounts:
	   - mountPath: /opt
	     name: data-volume 
	   
	 volumes:
	 - name: data-volume
	   persistentvolumeclaim:
	       claimName: my-claim
		   

Till now we have learned about the static provisioning of volumes.
It would be nice if the volume gets provisioned automatically when the application requires it and thats where storage classes comes in.

With Storage classes, you can define a provisioner such as google storage that can automatically provision storage on google cloud and attach that to pods when a claim is made. 
That is called dynamic provisioning of volumes.you do that by creating a storage class object with apiVersion set to storage.k8.io/v1.

sc-defintion.yaml 
--------------------
```yaml
  apiversion: storage.k8.io/v1
  kind: StorageClass
  metadata:
    name: google-storage
  provisioner: kubernetes.io/gce-pd
``` 
Since now we have storage class, we no longer need PV defintion file because PV and any associated storage is going to be created automatically when the storage class is created.
now we have to specify storage class name in pvc definition  file.

    apiVersion: v1
   kind: PersistantVolumeClaim
   metadata:
     name: my-claim
   spec:
     accessModes:
         - ReadWriteOnce
     storageClassName: google-storage 
     resources:
         requests: 
           storage: 500Mi
		   
Storage class still creates a PV but now you don't have to create it manually.It creates automatically by storage class.