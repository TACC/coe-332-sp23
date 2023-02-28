Docker Compose
==============

Using the ``docker run`` command to start containers is OK for simple commands, but as 
we started to see in the previous material, the commands can get long pretty quickly. It can be
hard to remember all of the flags and options that we want to use when starting our
containers. 

Moreover, so far we have been looking at single-container applications. 
But what if we want to do something more complex involving multiple containers? In this course, 
our goal is to ultimately develop and orchestrate a multi-container
application consisting of, e.g., a Flask app, a database, a message queue, an
authentication service, and more.

**Docker compose** is tool for managing multi-container applications that allows you to document 
all of the options you want to use for starting the containers in a single file. A YAML
file is used to define all of the application containers, called *services*, and a few simple commands
can be used to spin up or tear down all of the services.

In this module, we will get a first look at Docker compose. Later in this course
we will do a deeper dive into advanced container orchestration. After going
through this module, students should be able to:

* Translate Docker run commands into YAML files for Docker compose
* Run commands inside *ad hoc* containers using Docker compose
* Manage small software systems composed of more than one script, and more than
  one container
* Copy data into and out of containers as needed
* **Design Principles:** Docker compose will further enable our use of containers 
  to support the portability of our software projects.


Write a Compose File
--------------------

Docker compose works by interpreting rules declared in a YAML file (typically
called ``docker-compose.yml``). The rules we will write will replace the
``docker run`` commands we have been using, and which have been growing quite
complex. Recall from the past lecture that the command we were using to start our Flask 
application container looked like the following:

.. code-block:: console

   [user-vm]$ docker run -p 5000:5000 -v /home/ubuntu/coe332/docker2/config.yaml:/config.yaml --rm jstubbs/degrees_api

The above ``docker run`` command can be translated into a YAML file.
Navigate to the folder that contains your Python scripts and Dockerfiles, then
create a new empty file called ``docker-compose.yml``:

.. code-block:: console

   [user-vm]$ pwd
   /home/ubuntu/coe-332/docker
   [user-vm]$ touch docker-compose.yml
   [user-vm]$ ls
   Dockerfile  config.yaml  degrees_api.py  docker-compose.yml


Next, open up ``docker-compose.yml`` with your favorite text editor and type /
paste in the following text:

.. code-block:: yaml
   :linenos:
   :emphasize-lines: 9

   ---
   version: "3"

   services:
       flask-app:
           build:
               context: ./
               dockerfile: ./Dockerfile
           image: username/degrees_api:1.0
           volumes:
               - ./config.yaml:/config.yaml

.. note::

   Be suer to update the highlighted line above with your username.


The ``version`` key must be included and simply denotes that we are using
version 3 of Docker compose.

The ``services`` section defines the configuration of individual container
instances that we want to orchestrate. In our case, we define just one container
called ``flask-app``. We can use any allowable name for the services we defined, but each
name should be unique within the docker-compose.yml file. 

The ``flask-app`` service is configured with its own Docker image, including A
reference to a Dockerfile to be used to ``build`` the image, 
and a list of mounted volumes (equivalent to the ``-v`` option for ``docker run``), in 
this case, just one volume, to mount the configuration file. 

.. note::

   The top-level ``services`` keyword shown above is just one important part of
   Docker compose. Later in this course we will look at named volumes and
   networks which can be configured and created with Docker compose.


Running Docker Compose
----------------------

The Docker compose command line too follows the same syntax as other Docker
commands:

.. code-block:: console

   docker-compose <verb> <parameters>

Just like Docker, you can pass the ``--help`` flag to ``docker-compose`` or to
any of the verbs to get additional usage information. To get started on the
command line tools, try issuing the following two commands:

.. code-block:: console

   [user-vm]$ docker-compose version
   [user-vm]$ docker-compose config

The first command prints the version of Docker compose installed, and the second
searches your current directory for ``docker-compose.yml`` and checks that it
contains only valid syntax.

To run our Flask application container, we simply use the ``docker-compose up`` verb, which will start up all 
containers defined in the file. Alternatively, we could use ``docker-compose run``
and pass the name of a service to run, in this case, ``flask-app``:

.. code-block:: console

   [user-vm]$ docker-compose up 

   Creating docker_flask-app_1 ... done
   Attaching to docker_flask-app_1
   flask-app_1  |  * Serving Flask app 'degrees_api'
   flask-app_1  |  * Debug mode: off
   flask-app_1  | WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
   flask-app_1  |  * Running on all addresses (0.0.0.0)
   flask-app_1  |  * Running on http://127.0.0.1:5000
   flask-app_1  |  * Running on http://172.18.0.2:5000
   flask-app_1  | Press CTRL+C to quit

Note that ``docker-compose`` starts the container in the foreground and takes over our terminal. If we use 
``Ctrl+C`` we will stop the container. We can see confirm that the container is stopped using the
``docker ps -a`` command:

.. code-block:: console

   [user-vm] docker ps -a 
   CONTAINER ID   IMAGE                      COMMAND                  CREATED              STATUS                       PORTS     NAMES
   cd3e3df2cb84   username/degrees_api:1.0   "python degrees_api.…"   About a minute ago   Exited (137) 5 seconds ago             docker_flask-app_1



Essential Docker Compose Command Summary
----------------------------------------

+------------------------+------------------------------------------------+
| Command                | Usage                                          |
+========================+================================================+
| docker-compose version | Print version information                      |
+------------------------+------------------------------------------------+
| docker-compose config  | Validate docker-compose.yml syntax             |
+------------------------+------------------------------------------------+
| docker-compose up      | Spin up all services                           |
+------------------------+------------------------------------------------+
| docker-compose down    | Tear down all services                         |
+------------------------+------------------------------------------------+
| docker-compose build   | Build the images listed in the YAML file       |
+------------------------+------------------------------------------------+
| docker-compose run     | Run a container as defined in the YAML file    |
+------------------------+------------------------------------------------+


Additional Resources
--------------------

* `Docker Compose Docs <https://docs.docker.com/compose/>`_
