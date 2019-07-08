### Create flask app
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

### Push image to ecp repository
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
   
Now to pull the image from ECP registry to kubernets cluster, we need to provide the username and password
ECP uses fixed secret named `myregistrykey`
```bash
kubect create secret docker-registry myregistrykey \
  --docker-email=swarvanu.sengupta@cdnetworks.co.kr \
  --docker-username=opseng \
  --docker-password=$DOCKER_PASS
```
or
```
docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/$CERT \
   -v $KEY:/root/.kube/$KEY \
   kubectl:2.2.2 create secret docker-registry myregistrykey \
   --docker-email=swarvanu.sengupta@cdnetworks.co.kr \
   --docker-username=opseng --docker-password=$DOCKER_PASS
```
You can verify the secret with 
```bash
kubect get secrets
```
or
```bash
docker run -v $CONFIG:/root/.kube/config \
   -v $CERT:/root/.kube/$CERT \
   -v $KEY:/root/.kube/$KEY \
   kubectl:2.2.2 get secrets
```

### Create a deployment and deploy in ECP


