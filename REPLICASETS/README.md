# ReplicaSet(rs): K8s workload resource
----------------------------------------
- It makes sure that specified number of PODs(workload) are running at any given point of time on the cluster.
- Deployment object uses ReplicaSet to manage POD replicas
- apiVersion for ReplicaSet object is `apps/v1`.
- `Workflow:` ReplicaSet -> POD
- It's recommended to use Deployment instead of ReplicaSet(unless we need to manually upgrade the application)
- RS will use POD template to create POD with same specifications based on the replica count(.spec.replicas=1)
- Make sure standalone PODs do not match labels with ReplicaSet or ReplicationController as well.
- In the replicaSet, .spec.template.metadata.labels must match .spec.selector or it will be rejected by API.
- In order to delete POD managed under RS, delete RS not pod.
- We can delete RS without deleting PODs managed by RS(`#kubectl delete --cascade=orphan rs <rs-name>`)
- Scalein and scaleout of replicas are done using `#kubectl scale --replicas=<count> rs/<rs-name>`
- RS selector supports two parameters to define labels -> matchLabels and matchExpressions
- matchExpressions has key, operator and values parameters.

*Note: Whatever label that you give within the selector should match the label which you assign within the template section under metadata field.*

# Let's understand the ReplicaSet Selector:matchExpressions
----------------------------------------------------------------
key     |    operator        |      values
----------------------------------------------------------------------
key_name  |   [In, NotIn, Exists, DoesNotExists]  | [value1, value2]  |
----------------------------------------------------------------------
for In operator, we can have one value or multiple values. (like OR operator, you can have one match or multiple match)
for NotIn operator, we can provide list of values as well.

the operators `Exists` and `DoesNotExist` are used to select objects based on the presence or absence of a `label key`, regardless of its value. 


NOTE: If you pass both `matchLabels` and `matchExpressions`, it should be like `AND` condition. both should pass in order to create replicaset.


for the `Exists`, you don't have to define values. It will see in template.metadata labels.

