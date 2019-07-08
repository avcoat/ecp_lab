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
