## Create flask app
First we will create a simple python3 Flask app
This will response with `"Hello World!"`   
Create a file `app.py` as:
```python
from flask import Flask

APP = Flask(__name__)


@APP.route('/')
def handle_request():
    return "Hello World!"


# create_app returns the flask app
#            can be consumed by wsgi server
def create_app():
    return APP


if __name__ == '__main__':
    APP.run(debug=True, host='0.0.0.0')
```
We can use [cherrypy](https://cherrypy.org) for a wsgi server 
Create a file `server.py` as
```python
from app import create_app
import cherrypy

if __name__ == '__main__':
    app = create_app()

    # Mount the application
    cherrypy.tree.graft(app, "/")

    # Unsubscribe the default server
    cherrypy.server.unsubscribe()

    # Instantiate a new server object
    server = cherrypy._cpserver.Server()

    # Configure the server object
    server.socket_host = "0.0.0.0"
    server.socket_port = 8080
    server.thread_pool = 30

    # Subscribe this server
    server.subscribe()

    # Start the server engine
    cherrypy.engine.start()
    cherrypy.engine.block()
```
To install cherrypy and flask we will use pip3.  
Create a file `requirments` as:
```
flask
cherrypy
```
Now we will create the dockerfile for our application   
We will use multilayer dockerfile to reduce the size of final image  
```Dockerfile
FROM python:3.7-alpine as base
FROM base as builder
# We copy just the requirements.txt first to leverage Docker cache
COPY ./requirements /install/requirements
WORKDIR /install
RUN pip install --install-option="--prefix=/install" -r requirements

FROM base
COPY --from=builder /install /usr/local
RUN cp /usr/local/bin/python /usr/bin/python
WORKDIR /app
COPY app.py    .
COPY server.py .
EXPOSE 8080
ENTRYPOINT [ "python" ]
CMD [ "server.py" ]
```
Build the docker file 
```bash
docker build -t oe-tools/dummy-flask-app .
docker run -p 8080:8080 oe-tools/dummy-flask-app
```
To test do
```bash
curl "localhost:8080"
```

## Push image to ecp repository
Set your ecp repository password as
```
export DOCKER_PASS=<password>
```
Tag, Login and Push
```bash
docker tag oe-tools/dummy-flask-app registry-qcc.quantil.com/oe-tools/dummy-flask-app:1.0.0
echo $DOCKER_PASS | docker login --username=opseng --password-stdin registry-qcc.quantil.com
docker push registry-qcc.quantil.com/oe-tools/dummy-flask-app:1.0.0
```
We successfully pushed our dummy app as `registry-qcc.quantil.com/oe-tools/dummy-flask-app:1.0.0` 
   
Now to pull the image from ECP registry to kubernets cluster, we need to provide the username and password.  
We can create a kubenerts secrets `myregistrykey` that we be used as `imagePullSecrets`. 
```bash
kubect create secret docker-registry myregistrykey \
  --docker-server="registry-qcc.quantil.com" \
  --docker-email=swarvanu.sengupta@cdnetworks.co.kr \
  --docker-username=opseng \
  --docker-password=$DOCKER_PASS
```
or
```
docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/opseng_cert.pem \
   -v $KEY:/root/.kube/opseng_key.pem \
   kubectl:2.2.2 create secret docker-registry myregistrykey \
   --docker-server="registry-qcc.quantil.com" \
   --docker-email=swarvanu.sengupta@cdnetworks.co.kr \
   --docker-username=opseng \
   --docker-password=$DOCKER_PASS
```
You can verify the secret with 
```bash
kubect get secrets
```
or
```bash
docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/opseng_cert.pem \
   -v $KEY:/root/.kube/opseng_key.pem \ 
   kubectl:2.2.2 get secrets
```

## Create a deployment and deploy in ECP

Create a deployment file for the `dummy-flask-app` as `dummy-flask-app.yml`
```yml
apiVersion: apps/v1 

kind: Deployment

metadata:
    name: dummy-flask-app
    namespace: opseng

spec:
    # Create replicated pod and manage using replica set
    # Note: Dont use replication Controller to replicate PODS, 
    #       Using replica set via deployments is the recomanded way
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
        spec:
            # the secret ECP will use to pull the image
            imagePullSecrets:
                - name: myregistrykey
            containers:
            # image in the qcc registry that we pushed earlier
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
We can deploy the deployment using 
```bash
kubectl create -f dummy-flask-app.yml
```
or 
```bash
docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/opseng_cert.pem \
   -v $KEY:/root/.kube/opseng_key.pem \
   -v $(PWD)/dummy-flask-app.yml:/dummy-flask-app.yml \
   kubectl:2.2.2 create -f dummy-flask-app.yml
```
To verify the deployment
```bash
kubectl get deployments
```
or 
```bash
docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/opseng_cert.pem \
   -v $KEY:/root/.kube/opseng_key.pem \
   kubectl:2.2.2 get deployments
```
