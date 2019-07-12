Kubernets [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) has two types of probes    

**Readiness**   
> readiness probes is to know when a Container is ready to start accepting traffic   

**Lifeness**   
> liveness probes is to know when to restart a Container
   
   
**Readiness and liveness probes can be used in parallel for the same container. Using both can ensure that traffic does not reach a container that is not ready for it, and that containers are restarted when they fail.**


### Define Health check
For our application we already added an API for health as `/_health`. Our implentation is rather simple, but one can add more feature to chcek application health inclusing scenario such as `database-conenction`, `deadlock`, `subsystem-availability` etc.  

To define the health check we will update `dummy-flask-app.yml` as:
```yml
apiVersion: v1 
kind: Service 
metadata:
  name: dummy-flask-app
  labels:
    app: dummy-flask-app
spec:
  ports:
  - port: 80
    name: dummy-flask-app
  clusterIP: None # ​Statefulsets​ require a ​Headless Service
  selector:
    app: dummy-flask-app

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: dummy-flask-app
  namespace: opseng
spec:
  selector:
    matchLabels:
      app: dummy-flask-app
  serviceName: "dummy-flask-app"
  replicas: 3 
  template:
    metadata:
      labels:
        app: dummy-flask-app
      annotations:
        quantil.com/ipool: dummy-flask-app-ipool
    spec:
      imagePullSecrets:
       - name: myregistrykey
      containers:
       - image: registry-qcc.quantil.com/oe-tools/dummy-flask-app:1.5.0
         name: dummy-flask-app
         livenessProbe:
           httpGet:
             path: /_health
             port: 8080
           initialDelaySeconds: 10 # Start after 10 seconds
           periodSeconds: 3 # interval is 3 seconds
           failureThreshold: 2 # if two consicutive request fails restart
         readinessProbe:
           httpGet:
             path: /_health
             port: 8080
           initialDelaySeconds: 2 # Start after 2 seconds
           periodSeconds: 5 # interval is 5 seconds
           failureThreshold: 3 # if threee consicutive request fails mark unready
         resources:
           requests: 
             memory: "64Mi" 
             cpu: "100m"
           limits:
             memory: "128Mi"
             cpu: "200m"
```
and configure with 
```bash
kubectl -f dummy-flask-app.yml
```
or 
```bash
docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/opseng_cert.pem \
   -v $KEY:/root/.kube/opseng_key.pem \
   -v $(PWD)/dummy-flask-app.yml:/dummy-flask-app.yml \
   kubectl:2.2.2 apply -f dummy-flask-app.yml
```

We can check the health check result with 
```bash
kubectl describe sts | grep health
```
or
```bash
docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/opseng_cert.pem \
   -v $KEY:/root/.kube/opseng_key.pem \
   kubectl:2.2.2 describe sts | grep health
```
