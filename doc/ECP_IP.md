By default in kubernetes, every pod gets assigned an lcoal IP address.  
And every container in the pod gets assigned that same IP address. 
> The applications in a pod all use the same network namespace (same IP and port space), and can thus “find” each other and communicate using localhost. Because of this, applications in a pod must coordinate their usage of ports. Each pod has an IP address in a flat shared networking space that has full communication with other physical computers and pods across the network.

In ECP though you can request and assign an public IP address to your PODs. For that ECP provides a custom object `IPool`

### Create Reserved IP Pool
**You need permission to create IPool object in ECP**    

To Create a reserved IP pool define a `IPool` object in `ip-pool.yml` as:
```yml
apiVersion: v1 
kind: IPool 
metadata:
   name: dummy-flask-app-ipool
   namespace: opseng 
spec:
   enableExternalIP: true # Optional. Set to “true” if static public IPV4 address is required 
   enableIpv6ExternalIP: false #   Optional. Set to “true” if static public IPV6 address is required
   counts: 3 # The count of static addresses needed. IPV4 and IPV6 addresses are counted separately.
```
To create the `IPool` 
```bash
kubectl create -f ip-pool.yml
```
or 
```bash
docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/opseng_cert.pem \
   -v $KEY:/root/.kube/opseng_key.pem \
   -v $(PWD)/ip-pool.yml:/ip-pool.yml \
   kubectl:2.2.2 create -f ip-pool.yml
```
To List the `IPool` objects
```bash
kubectl get ipools
```
or
```bash
docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/opseng_cert.pem \
   -v $KEY:/root/.kube/opseng_key.pem \
   kubectl:2.2.2 get ipools
```
Once a `IPool` created you will get 3 reserved IP's for your applcation. To verify
```bash
kubectl describe ipool ipool-example
```
or
```bash
docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/opseng_cert.pem \
   -v $KEY:/root/.kube/opseng_key.pem \
   kubectl:2.2.2 describe ipool dummy-flask-app-ipool
```


> * The IP's that are reserved for you will not change when you scale up/down. Those IP's gets released when the IPool object is deleted.   
> * A ipool object is always mapped to only one POD template. If your replication factor is more that the IP counts in reserved pool, it will cause an error.  


### Assign IP Pool to Pods

To assign the `IPool` to the POD's we will update the POD template by adding a `annotations`:
```yaml
            annotations:
                quantil.com/ipool: dummy-flask-app-ipool
                
```
Update the `dummy-flask-app.yml` as:
```yaml
apiVersion: apps/v1 

kind: Deployment

metadata:
    name: dummy-flask-app
    namespace: opseng

spec:
    # Create replicated pod and manage using replication controller
    replicas: 3

    # defines how the Deployment finds which Pods to manage
    selector:
        matchLabels:
            app: dummy-flask-app

    # Template for the pods
    template:
        # set the metadata for pods, which labels to use
        # same labels is used by selector in deployment
        metadata:
            labels:
                app: dummy-flask-app
            annotations:
                quantil.com/ipool: dummy-flask-app-ipool
        spec:
            # the secret ECP will use to pull the image
            imagePullSecrets:
                - name: myregistrykey
            containers:
            - image: registry-qcc.quantil.com/oe-tools/dummy-flask-app:1.0.0
              name: dummy-flask-app
              resources:
                  requests: 
                      memory: "64Mi" 
                      cpu: "100m"
                  limits:
                      memory: "128Mi"
                      cpu: "200m"
```
To update the deployment 
```bash
kubectl apply -f dummy-flask-app.yml
```
or
```bash
docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/opseng_cert.pem \
   -v $KEY:/root/.kube/opseng_key.pem \
   -v $(PWD)/dummy-flask-app.yml:/dummy-flask-app.yml \
   kubectl:2.2.2 apply -f dummy-flask-app.yml
```
Once updated you can check the assigned External IPs to corresponding pods in `Annotations`
```bash
kubectl describe pods --selector="app=dummy-flask-app" | grep externalIP
```
or
```bash
docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/opseng_cert.pem \
   -v $KEY:/root/.kube/opseng_key.pem \
   kubectl:2.2.2 describe pods --selector="app=dummy-flask-app" | grep externalIP
```
For pod you can get the public IP from the annotion `quantil.com/externalIP` 
To avail the `External IP` to your container you can pass it by adding environment in Pod Template
```yml
        env:
        - name: PUBLIC_IP
          valueFrom:
            fieldRef:
              fieldPath: metadata.annotations['quantil.com/externalIP']
```

### Access your APP
Each of your PODs gets a different external IP's. Now you can use one of these EXTERNAL_IP to call your application 
```
export dummyflaskapp=<EXTERNAL_IP>
curl "$dummyflaskapp:8080"
```
