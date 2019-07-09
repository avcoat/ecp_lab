Now that we have our initial app deployed, we will try to dig more about the internals of QCC and K8 itself.  
Along with that we will learn about few handy kubectl commands

## Dig your deployment

We already have created a deployment as `dummy-flask-app`. To know more about the deployment 
```bash
kubectl describe deployment dummy-flask-app
```
or 
```bash
docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/opseng_cert.pem \
   -v $KEY:/root/.kube/opseng_key.pem \
   kubectl:2.2.2 describe deployment dummy-flask-app
```
Kubernets creates replica set for each deployments. In the deployment desc it has 
```yaml
OldReplicaSets:  
NewReplicaSet:
```
Replica sets are for maintaining deployments and scaling. 
Version upgrade is done by rollout of new replica-sets where Kubenets keeps the reference to `OldReplicaSets` to rollback  

To list the replica sets for current deployment, use the label as selector
```bash
kubectl get rs --selector="app=dummy-flask-app"
```
or
```bash
docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/opseng_cert.pem \
   -v $KEY:/root/.kube/opseng_key.pem \
   kubectl:2.2.2 get rs --selector="app=dummy-flask-app"
```

## Dig your replica-set

To get to know about the replicaset
```bash
kubectl describe rs dummy-flask-app-6cc9fcbb8f
```
or 
```bash
docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/opseng_cert.pem \
   -v $KEY:/root/.kube/opseng_key.pem \
   kubectl:2.2.2 describe rs dummy-flask-app-6cc9fcbb8f
```
* Note: We will no more about `rollback` and `rollout` in the operation section

Check the Pod status summary 
```yaml
Pods Status:    2 Running / 1 Waiting / 0 Succeeded / 0 Failed
```
If your pods status is not Succeeded or has Failed, you can debug by looking at the Pods

## Dig your pods

To list the pods created by the replicaset, use the selector of rs
```bash
kubectl get pods --selector="app=dummy-flask-app"
```
or
```bash
docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/opseng_cert.pem \
   -v $KEY:/root/.kube/opseng_key.pem \
   kubectl:2.2.2 get pods --selector="app=dummy-flask-app"
```
Now to look in details of each pods perform
```bash
kubectl describe pods --selector="app=dummy-flask-app"
```
or
```bash
docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/opseng_cert.pem \
   -v $KEY:/root/.kube/opseng_key.pem \
   kubectl:2.2.2 describe pods --selector="app=dummy-flask-app"
```

To debug any issues you can see the `Events` section   
or get the logs from all pods 
```bash
kubectl get logs --selector="app=dummy-flask-app"
```
or 
```bash
docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/opseng_cert.pem \
   -v $KEY:/root/.kube/opseng_key.pem \
   kubectl:2.2.2 logs --selector="app=dummy-flask-app"
```

