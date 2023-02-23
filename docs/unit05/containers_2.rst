Advanced Containers
===================

In the first part, we pulled and ran existing container images from Docker Hub.
In this section, we will build an image from scratch for running some of our own
Python3 code. Then, we will push that image back up to Docker Hub so others may
find and use it. After going through this module, students should be able to:

* Install and test code in a container interactively
* Write a Dockerfile from scratch
* Build a Docker image from a Dockerfile
* Push a Docker image to Docker Hub
* **Design Principles:** As with the previous lecture, we will see how containers contribute 
  to the portability of software projects


Getting Set Up
--------------

*Scenario:* You are a developer who has written some code the provides a JSON API to some
data. You now want to distribute that
code for others to use in what you know to be a stable production environment
(including OS and dependency versions). End users may want to use this application
on their local workstations, in the cloud, or on an HPC cluster.


The first step in a typical container development workflow entails installing
and testing an application interactively within a running Docker container.

.. note::

   We recommend doing this on your own student VM. But, one of the most
   important features of Docker is that it is platform agnostic. These steps
   could be done anywhere Docker is installed.


To begin, make a new folder for this work and prepare to gather some important
files.


.. code-block:: console

   [user-vm]$ cd ~/coe-332/
   [user-vm]$ mkdir docker/
   [user-vm]$ cd docker/
   [user-vm]$ pwd
   /home/ubuntu/coe332/docker

We're going to containerize the ``degrees_api`` Flask application from the end of Unit 4. 
Specifically, you need your ``degrees_api.py`` script that we wrote together in class. To simplify
our work today, we will just copy the file into our docker directory:

.. code-block:: console

   [user-vm]$ cp ~/coe-332/degrees_api.py .

.. note::
   You may need to update the path above to point to your ``degrees_api.py`` file.


.. warning::

   It is important to carefully consider what files and folders are in the same
   ``PATH`` as a Dockerfile (known as the 'build context'). The ``docker build``
   process will index and send all files and folders in the same directory as
   the Dockerfile to the Docker daemon, so take care not to ``docker build`` at
   a root level.


Containerize Code Interactively
-------------------------------

There are several questions you must ask yourself when preparing to containerize
code for the first time:

1. What is an appropriate base image?
2. What dependencies are required for my program?
3. What is the install process for my program?
4. What environment variables may be important?

We can work through these questions by performing an **interactive installation**
of our Python script. The official Python container image that worked with last lecture
is a an excellent choice for Python projects, as it already contains a well-maintained
installation of Python, and we can easily pick the version of Python that we want by selecting
different tags. The Python we have been using on the VM is Python 3.8.10, so we'll stick 
with that one to avoid any Python-specific issues that could arise from chaging the version. 

To get that exact version of the Python offisial image, use ``docker pull python:3.8.10``.
We can also run a container from the image to 


.. code-block:: console

   [user-vm]$ docker pull python:3.8.10
   [user-vm]$ docker run --rm -it python:3.8.10 /bin/bash
   [root@7ad568453e0b /]#

Here is an explanation of the options:

.. code-block:: text

   docker run       # run a container
   --rm             # remove the container on exit
   -it              # interactively attach terminal to inside of container
   python:3.8.10    # image and tag from Docker Hub
   /bin/bash        # shell to start inside container


The command prompt will change, signaling you are now 'inside' the container. Let's check that 
we can run ``python`` from the shell:

.. code-block:: console

  [root@7ad568453e0b /]$ python
  Python 3.8.10 (default, Jun 23 2021, 15:19:53) 
  [GCC 8.3.0] on linux
  Type "help", "copyright", "credits" or "license" for more information.


What about Flask? What happens if try to import it from the Python repl?

.. code-block:: console

  [root@7ad568453e0b /]$ python
  Python 3.8.10 (default, Jun 23 2021, 15:19:53) 
  [GCC 8.3.0] on linux
  Type "help", "copyright", "credits" or "license" for more information.
  >>> import flask


The base Python images have the Python interpreter and standard library, but they do not include 
any third-party packages. If we try to ``import flask``, we'll get a ``ModuleNotFoundError`` exception.

.. code-block:: console

   >>> import flask
   Traceback (most recent call last):
   File "<stdin>", line 1, in <module>
   ModuleNotFoundError: No module named 'flask'

To use Flask, we'll need to install it in the container image ourselves.

Install Required Packages
~~~~~~~~~~~~~~~~~~~~~~~~~

For our Python Flask program to work, we need to install the ``Flask`` package. 
How do we typically install Python packages? We use the ``pip`` package manager. Does
our Python container image have ``pip`` installed? Let's check:

.. code-block:: console

   [root@7ad568453e0b /]# pip -h
   Usage:   
   pip <command> [options]

   Commands:
   install                     Install packages.
   ...

It does! That's great. So we should be able to use pip to install a particular version of Flask. 
Which version do we want to install? We can check which version we were using on our VM and use the
same one in the container. That way, we won't hit any issues due to version changes with the 
package. 

Back out in the VM, we can get a list of packages pip knows about using ``pip freeze``:

.. code-block:: console

  [user-vm]$ pip freeze
   appdirs==1.4.3
   apturl==0.5.2
   asttokens==2.2.1
   attrs==19.3.0
   Automat==0.8.0
   backcall==0.2.0
   blinker==1.4
   Brlapi==0.7.0
   cached-property==1.5.1
   ceph==1.0.0
   cephfs==2.0.0
   certifi==2019.11.28
   . . . 

Wow, that's a long list! We really just want to know the version of the Flask package, so we can 
pipe the output of ``pip freeze`` to ``grep`` to just select lines with ``Flask`` in them (note the capital
``F``):

.. code-block:: console

  [user-vm]$ pip freeze | grep Flask
  Flask==2.2.2

Great, so we need Flask version 2.2.2. Back in the container, we can try to install Flask using ``pip`` 

.. code-block:: console

   [root@7ad568453e0b /]$ pip install Flask==2.2.2

   Collecting Flask==2.2.2
   Downloading Flask-2.2.2-py3-none-any.whl (101 kB)
      |████████████████████████████████| 101 kB 3.3 MB/s 
   Collecting itsdangerous>=2.0
   . . .
   Successfully installed Flask-2.2.2 Jinja2-3.1.2 MarkupSafe-2.1.2 Werkzeug-2.2.3 click-8.1.3 importlib-metadata-6.0.0 itsdangerous-2.1.2 zipp-3.14.0

That worked! Note that when pip installed Flask 2.2.2 it also installed its dependencies for us.

.. warning::

   An important question to ask is: Does the versions of Python and other
   dependencies match the versions you are developing with in your local
   environment? If not, make sure to install the correct version of Python.




Assemble a Dockerfile
---------------------

After going through the build process interactively, we can translate our build
steps into a Dockerfile using the directives described below. A Dockerfile is 
just a text file that contains commands for building a new image. We'll 
cover a few of the different Dockerfile instructions below. 

Create a new file called ``Dockerfile`` and open it with a text editor.


The FROM Instruction
~~~~~~~~~~~~~~~~~~~~

We can use the FROM instruction to start our new image from a known base image.
This should be the first line of our Dockerfile. In our scenario, we want to
use an official Python image that contains the same version of Python that we 
have been using on the VM:

.. code-block:: dockerfile

   FROM python:3.8.10

At this point, our new image just has the Python 3.8.10 official image in it.


The RUN Instruction
~~~~~~~~~~~~~~~~~~~

We can install updates, install new software, or download code to our image by
running commands with the RUN instruction. The RUN instruction works by literally
running the command line provided after ``RUN`` in the existing container image.
Any files created, modified or deleted by the command line will be correspondingly 
changes in the image.

In our case, our only dependency was the Flask library which we can install with ``pip``. 
We will use a RUN instruction to execute the ``pip`` command to install it. 

.. code-block:: dockerfile

   RUN pip install Flask==2.2.2

Each RUN instruction creates an intermediate image (called a 'layer'). Too many
layers makes the Docker image less performant, and makes building less
efficient. We can minimize the number of layers by combining RUN instructions.
Dependencies that are more likely to change over time (e.g. Python3 libraries)
still might be better off in in their own RUN instruction in order to save time
building later on.




The COPY Instruction
~~~~~~~~~~~~~~~~~~~~

Now we need to add our flask application. 
There are a couple different ways to get your source code inside the image.
When you are developing, the most practical methods is usually to copy code in
from the Docker build context using the COPY instruction. For example, we can
copy our script to the root-level `/` directory with the following
instructions:

.. code-block:: dockerfile

   COPY degrees_api.py /degrees_api.py


The CMD Instruction
~~~~~~~~~~~~~~~~~~~

Another useful instruction is the ``CMD`` instruction. This sets a default command line
to run in the container when none is provided to a ``docker run`` command that makes use 
of the image. To run our flask application, we can simply execute the file with python:

.. code-block:: console

  [user-vm]$ python degrees_api.py


To provide the command line to ``CMD`` instruction, separate each part of the command line 
into a list of strings. 

.. code-block:: dockerfile

   CMD ["python", "degrees_api.py"]



Putting It All Together
~~~~~~~~~~~~~~~~~~~~~~~

The contents of the final Dockerfile should look like:

.. code-block:: dockerfile
   :linenos:

   FROM python:3.8.10

   RUN pip install Flask==2.2.2

   COPY degrees_api.py /degrees_api.py

   CMD ["python", "degrees_api.py"]


Build the Image
---------------

Once the Dockerfile is written and we are satisfied that we have minimized the
number of layers, the next step is to build an image. Building a Docker image
generally takes the form:

.. code-block:: console

   [user-vm]$ docker build -t <dockerhubusername>/<code>:<version> .

The ``-t`` flag is used to name or 'tag' the image with a descriptive name and
version. Optionally, you can preface the tag with your **Docker Hub username**.
Adding that namespace allows you to push your image to a public registry and
share it with others. The trailing dot '``.``' in the line above simply
indicates the location of the Dockerfile (a single '``.``' means 'the current
directory').

To build the image, use:

.. code-block:: console

   [user-vm]$ docker build -t username/degrees_api:1.0 .

.. note::

   Don't forget to replace 'username' with your Docker Hub username.


Use ``docker images`` to ensure you see a copy of your image has been built. You can
also use `docker inspect` to find out more information about the image.

.. code-block:: console

   [user-vm]$ docker images
   REPOSITORY                 TAG        IMAGE ID       CREATED              SIZE
   jstubbs/degrees_api        1.0        2883079fad18   About a minute ago   928MB
   ...

.. code-block:: console

   [user-vm]$ docker inspect username/degrees_api:1.0


If you need to rename your image, you can either re-tag it with ``docker tag``, or
you can remove it with ``docker rmi`` and build it again. Issue each of the
commands on an empty command line to find out usage information.



Test the Image
--------------

We can now test our newly-built image! Let's start a container from the image
using the ``docker run`` command. Execute the following in your VM:

.. code-block:: console

   [user-vm]$ docker run -it --rm -p 5000:5000 username/degrees_api:1.0

   * Serving Flask app 'degrees_api'
   * Debug mode: on
   WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
   * Running on all addresses (0.0.0.0)
   * Running on http://127.0.0.1:5000
   * Running on http://172.17.0.2:5000
   Press CTRL+C to quit
   * Restarting with stat
   * Debugger is active!
   * Debugger PIN: 127-571-634

Note the use of the ``-p`` flag to bind a port on the container to a port on the VM. We'll talk more about
container networking later, but for now, understand that every container gets a complete set of "private" 
ports that, by default, are not connected to the host ports. Since our Flask application listens on port 
5000, if we do not connect the container's port 5000 to the host's port 5000, then we won't be able to communicate
with our Flask program on the VM. 

The ``-p`` flag takes the form ``<host port>:<container port>`` and connects the two. 

With our ``degrees_api`` container running, let's use curl in another window to interact with our 
program:

.. code-block:: console

   [user-vm]$ curl 127.0.0.1:5000/degrees
   [
      {
         "degrees": 5818,
         "id": 0,
         "year": 1990
      },
      {
         "degrees": 5725,
         "id": 1,
         "year": 1991
      },
      {
         "degrees": 6005,
         "id": 2,
         "year": 1992
      },
      {
         "degrees": 6123,
         "id": 3,
         "year": 1993
      },
      {
         "degrees": 6096,
         "id": 4,
         "year": 1994
      }
   ]

It worked!


Share Your Docker Image
-----------------------

Now that you have containerized, tested, and tagged your code in a Docker image,
the next step is to disseminate it so others can use it.

Docker Hub is the *de facto* place to share an image you built. Remember, the
image must be name-spaced with either your Docker Hub username or a Docker Hub
organization where you have write privileges in order to push it:

.. code-block:: console

   [user-vm]$ docker login
   ...
   [user-vm]$ docker push username/degrees_api:1.0


You and others will now be able to pull a copy of your container with:

.. code-block:: console

   [user-vm]$ docker pull username/degrees_api:1.0


As a matter of best practice, it is highly recommended that you store your
Dockerfiles somewhere safe. A great place to do this is alongside the code
in, e.g., GitHub. GitHub also has integrations to automatically update your
image in the public container registry every time you commit new code. (More on
this later in the semester).

For example, see: `Publishing Docker Images <https://docs.github.com/en/actions/publishing-packages/publishing-docker-images/>`_


Application Configuration Files and Container Volume Mounts
-----------------------------------------------------------
Some times we need to provide additional files to a container that we do not want to include in the public 
image available on Docker Hub. For example, if our application needs to use passwords or other secret information 
we don't want to include that data in the public image because then anyone who downloaded the image could see 
the secrets. 

Moreover, it is often convenient to allow applications to be configured to run in different ways. One way 
to allow them to be configured is with a configuration file. The configuration file contains settings and other 
values that the software reads to know how it should run.

First, we'll modify our Flask application to use a configuration file. We'll use YAML because it is easy to read and
write, it can include comments, and it will give us some practice for when we write Kubernetes YAML files later
in the semester. 

In order to work with yaml, we'll use the ``pyyaml`` library. The ``pyyaml`` library must be installed with ``pip``,
for example:

.. code-block:: console

   [user-vm] pip install pyyaml==6.0

For the very first version of our configuration file, we'll accept a single configuration called ``debug`` that 
takes a ``bool`` value (true or false). If ``debug`` is set to ``true`` in the configuration file, we'll start our 
server in debug mode; otherwise, we'll start our server in normal mode.

Our strategy will be to look for a file called ``config.yaml`` in the current working directory; that is, 
in the same directory as our ``degress_api.py`` file. We'll use the following rules for configuring our application:

* If the file doesn't exist or if the file is not valid YAML we'll simply ignore the configuration file and start 
  the server in debug mode.
* If the file exists and is valid YAML, we will read the file, check for a config variable called ``debug``, and use
  the value, if it exists.
* If the configuration file exists and is valid YAML but ``debug`` is not set in it, we will ignore it and start the 
  server in debug mode.


Reading the Configuration File
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Our first step is to read the configuration file. For that, we need to import the ``yaml`` package:

.. code-block:: python
   :linenos:
   :emphasize-lines: 3

   from flask import Flask, request
   import json
   import yaml

To read the file, we'll use the ``yaml.safe_load()`` function. We'll create a function called ``get_config``. 
Our first attempt might look something like this:

.. code-block:: python

   def get_config():
      with open('config.yaml', 'r') as f:
          return yaml.safe_load(f)

However, we'll have a problem if either the file does not exist or is not valid YAML. How can we fix it?

.. note:: 
   
   We can use exceptions to handle different kinds of errors at run time.

Let's use a ``try.. except`` block to handle the different errors:

.. code-block:: python

    def get_config():
        default_config = {"debug": True}
        try:
            with open('config.yaml', 'r') as f:
                return yaml.safe_load(f)
        except Exception as e:
            print(f"Couldn't load the config file; details: {e}")
        # if we couldn't load the config file, return the default config
        return default_config


Use the Configuration to Start Flask
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
We need to read the configuration file before we start the flask server so we can know
whether to use the ``debug`` mode. We can add a call to ``get_config()`` in the main block:

.. code-block:: python
   :linenos:
   :emphasize-lines: 2-3,5

   if __name__ == '__main__':
       config = get_config()
       if config.get('debug', True):
           app.run(debug=True, host='0.0.0.0')
       else:
           app.run(host='0.0.0.0')


If we start up our Docker container again as before, then there will not be a configuration file, so we 
should see the default behavior of debug mode. We should also see a message indicating that it could not
find the configuration file:

.. code-block:: console

   [user-vm]  docker run -p 5000:5000 --rm -it jstubbs/c23-flask2
    Couldn't load the config file; details: [Errno 2] No such file or directory: 'config.yaml'
      * Serving Flask app 'degrees_api'
      * Debug mode: on


Using Volume Mounts to Add a Configuration File
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now, let's test our new configuration feature by providing a configuration file.
We won't add the configuration file to the image. Instead, we'll allow operators to write their own and 
add it directly to the container. How will we do that?

We'll use Docker volume mount. In Docker, a volume mount allows you to add files and directories from the host 
computer to a container. To add a volume mount, use the ``-v </path/on/host:/path/in/container>`` format.


.. note::

   The host path and the container path should be provided as absolute paths.

.. note::

   If you mount a path on the host to a path in the container that already exists in the image, the contents 
   in the image will be replaced by those on the host path. 

Let's first create a new file called ``config.yaml`` in the current directory and place the following contents
into it:

.. code-block:: yaml

   debug: false


Now let's modify our ``docker run`` statement to include a volume mount that adds the configuration file 
into our running container:

.. code-block:: console

  docker run -p 5000:5000 -v /home/ubuntu/coe332/docker2/config.yaml:/config.yaml --rm jstubbs/degrees_api

Here is an explanation of the options used above:

.. code-block:: text

   docker run                                   # run a container
   -p 5000:5000                                 # map the host port 5000 to the container port 5000
   -v  /home/ubuntu/coe332/docker2/config.yaml  # mount the config.yaml file on the host into the container.
       :/config.yaml                             
   --rm                                         # remove the container after the program exits
   jstubbs/degrees_api                          # image to use for the container 

Now you should see output similar to:

.. code-block:: console

    * Serving Flask app 'degrees_api'
    * Debug mode: off

Great! It read out config file and turned off debug mode.


Additional Resources
--------------------

* `Docker for Beginners <https://training.play-with-docker.com/beginner-linux/>`_
* `Play with Docker <https://labs.play-with-docker.com/>`_
