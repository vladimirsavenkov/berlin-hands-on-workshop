## Creating and Managing Deployments

Deployments sit on top of ReplicaSets and add the ability to define how updates to Pods should be rolled out.

In this section we will combine everything we learned about Pods and Services and create a Deplyoment manifest for our hello-node application. 
* Create a deployment manifest
* Scale our Deployment / ReplicaSet
* Update our application (Rolling Update | Recreate)

----

### Explore the Deployment

```
cat deployments/deployment.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: hello-node
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: hello-node
    spec:
      containers:
      - name: hello-node
        image: muellermich/hello-node:v1
        ports:
        - containerPort: 8080
```

----

### Create the Deployment

```
kubectl create -f deployment/deployment.yaml
```

----

### View the ReplicaSet

Behind the scenes Deployments manage ReplicaSets. Each deployment is mapped to one active ReplicaSet. Use the `kubectl get replicasets` command to view the current set of replicas.
```
kubectl get rs
NAME                   DESIRED   CURRENT   READY     AGE
hello-node-364036756   1         1         1         16s
```

----

### Scaling Deployments

ReplicaSets are scaled through the Deployment or independently. Use the `kubectl scale` command to scale:

```
kubectl scale deployments hello-node --replicas=3
deployment "hello-node" scaled
```

----

### Scale down the Deployment

```
kubectl scale deployments hello-node --replicas=2
deployment "hello-node" scaled
```

Check the status of the Deployment

```
kubectl describe deployment hello-node
```
```
kubectl get pods
```

----

### Updating Deployments ( RollingUpdate )

We need to make some changes to our node.js application and create a new image with a new Version. Default update strategy is RollingUpdate and we will test that out first.

Update the text `Hello World!` to `KubeCon 2017`

Build a new Dockerimage and tag it with v2

In one terminal issue:
```
while true ; do curl -I -s <CLUSTER_IP>:30080 -o /dev/null \
  -w "%{http_code}"; sleep 1; done
```
Update the Deployment
```
kubectl set image deployment/hello-node hello-node=muellermich/hello-node:v2
```

----

### Cleanup

```
kubectl delete -f deployment/deployment.yaml
```
If there were a large number of pods, this may take a while to complete. If you want to leave the pods running instead, specify `--cascade=false`
If you try to delete the pods before deleting the Deployments, it will just replace them, as it is supposed to do.

----

### Updating Deployments ( Recreate )

We'll see how to do an update to our application using the recreate strategy. First we need to create a deploment with the Recreate strategy.
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: hello-node
spec:
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: hello-node
    spec:
      containers:
      - name: hello-node
        image: muellermich/hello-node:v1
        ports:
        - containerPort: 8080
```

----

Create the deplyoment
```
kubectl create -f deployment/deployment-v2.yaml
```
In one terminal issue:
```
while true ; do curl -I -s <CLUSTER_IP>:30080 -o /dev/null \
  -w "%{http_code}"; sleep 1; done
```
Update the Deployment
```
kubectl set image deployment/hello-node hello-node=hello-node:v2
```

----

Not all requests will be succesful, as first the pods are getting terminated and then the new ones are cerated.

Cleanup

```
kubectl delete -f deployment/deployment-v2.yaml
```