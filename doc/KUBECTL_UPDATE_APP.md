We have used deployments to rollout our initial application. To revisit from official document
> A Deployment controller provides declarative updates for Pods and ReplicaSets.
You describe a desired state in a Deployment, and the Deployment controller changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.

### Update your app
We have created a dummy flask app in our earlier lab. Now we will update the app by adding a health check api as `_health`   
To do that we will update the `app.py` as
```python
from flask import Flask
  
APP = Flask(__name__)


@APP.route('/')
def handle_request():
    return "Hello World!"

@APP.route('/_health')
def check_health():
    return "OK"

# create_app returns the flask app
#            can be consumed by wsgi server
def create_app():
    return APP


if __name__ == '__main__':
    APP.run(debug=True, host='0.0.0.0')                                        
```
As we have updated the app we need to recreate the docker image and push   
We will update the tag to `1.5.0` from `1.0.0` 
```bash
docker build -t oe-tools/dummy-flask-app .
docker tag oe-tools/dummy-flask-app registry-qcc.quantil.com/oe-tools/dummy-flask-app:1.5.0
docker push registry-qcc.quantil.com/oe-tools/dummy-flask-app:1.5.0
```
Once the image has been pushed successfully we can update the deployment by changing the `dummy-flask-app.yml`  
We will update the image tag for container from `1.0.0` to `1.5.0`
```yml
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
            - image: registry-qcc.quantil.com/oe-tools/dummy-flask-app:1.5.0
              name: dummy-flask-app
              env:
              - name: PUBLIC_IP # make public IP available to your container
                valueFrom:
                    fieldRef:
                        fieldPath: metadata.annotations['quantil.com/externalIP']
              resources:
                  requests: 
                      memory: "64Mi" 
                      cpu: "100m"
                  limits:
                      memory: "128Mi"
                      cpu: "200m"
```

### Dealing With Reserve IP's in Rollouts

ECP IPool IPs are mapped to a POD until the pod dies. **Which makes rollout complecated**. Cause or the new pod template a new replica Set will be created. It will create the PODs first and try to assign the IP from IPool objects.   
Now as all IP’s are already reserved we will get errors like below:
```bash
 Warning  FailedSchedulingIP  6s (x20 over 4m)  default-scheduler  Can't allocate ip from ipool dummy-flask-app-ipool.Error: Operation cannot be fulfilled on ipools "dummy-flask-app-ipool": ipool dummy-flask-app-ipool have not freeip
```

There are different ways to deal with this problem

#### 1. Consider having one extra IP

If we have a replica-count `n` then the reserved IP count should be `n+1`. In this way we always have an additional IP to assign a new pod. Although this might still cause scheduler to try starting a POD before killing one and causing the same problem. Although the scheduler will to retry again and eventually the IP's will be freed. 

> This will cause one dangling IP    

To increase the reserve IP count update `ip-pool.yml` as
```yml
apiVersion: v1
kind: IPool
metadata:
   name: dummy-flask-app-ipool
   namespace: opseng
spec:
   enableExternalIP: true # Optional. Set to “true” if static public IPV4 address is required 
   enableIpv6ExternalIP: false #   Optional. Set to “true” if static public IPV6 address is required
   counts: 4 # The count of static addresses needed. IPV4 and IPV6 addresses are counted separately.
```
Then apply the ipool object as
```bash
docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/opseng_cert.pem \
   -v $KEY:/root/.kube/opseng_key.pem \
   -v $(PWD)/ip-pool.yml:/ip-pool.yml \                         
   kubectl:2.2.2 apply -f ip-pool.yml
```
To update the deployment perform 
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
Now for the update the kubernets will update each pods by initiating a new replication set   
To check the rollout status 
```bash
kubectl rollout status deployment.v1.apps/dummy-flask-app
```
or
```bash
docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/opseng_cert.pem \
   -v $KEY:/root/.kube/opseng_key.pem \
   kubectl:2.2.2 rollout status deployment.v1.apps/dummy-flask-app
```

#### 2. Consider having intermidiate RS
#### 3. Stateful Set
