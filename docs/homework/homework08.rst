Homework 08
===========

**Due Date: Thursday, Apr 13, by 11:00am CST**


Holidapp
--------

Scenario: Your **app** (Flask API + Redis DB) has been deployed in Kubernetes,
and now we are going to continue to build out some more features. First,
we will write some code to dynamically assign the Redis client host IP.
Second, we will add some functionality to write images to a Redis database
and read images back out.


.. note::

   Please continue to work on the HGNC data set from Homework 06. If you want
   to do this homework with a different data set, e.g. the data set you
   want to work on for your final project, just get it approved with the instructors
   first.



PART 1
~~~~~~

Start a new directory for this homework (called ``homework08``). Copy over all the
assets from ``homework06`` / ``homework07`` into this new directory. You should have:

* A Flask python script
* A Dockerfile
* A docker-compose.yml file
* 5 Kubernetes yml files

At this point we assume you can orchestrate your Flask and Redis services together 
using docker-compose in the **development environment** (Jetstream cluster, ``[user-vm]$``
). We assume that your latest Docker image for your Flask app has been pushed to 
Docker Hub. And we assume that you can orchestrate your Flask and Redis services together
in the **deployment environment** (Kubernetes cluster, ``[kube]$``). Please make sure 
this is all working before proceeding with the rest of this homework.


PART 2
~~~~~~

We will use an **Environment variable** to dynamically set the Redis host IP. Environment
variables are variables that get set in the shell and are available for programs. In 
python, the ``os.environ`` dictionary contains a key for every variable.
Implement some code like the following into your Flask app (or wherever you create the python
Redis client):

.. code-block:: python

   import os

   redis_ip = os.environ.get('REDIS_IP')
   if not redis_ip:
       raise Exception()
   rd=redis.StrictRedis(host=redis_ip, port=6379, db=0)

Then, in your docker-compose.yml file, add the following attribute to your Flask 
service:

.. code-block:: yaml

   environment:
     - REDIS_IP=<what value goes here?>

Figure out what value should be set in ``<what value goes here?>``, and make sure your
services still can be orchestrated together with docker-compose in the **development environment**.
When it is working, don't forget to push your modified Docker image to Docker Hub.

Next, add the following attribute to your Flask deployment spec:

.. code-block:: yaml

   env:
     - name: REDIS_IP
       value: <what value goes here?>

Figure out what value should be set in ``<what value goes here?>``, and make sure your
services still can be orchestrated together with kubernetes in the **deployment environment**.


`Hint 1 <https://docs.docker.com/compose/environment-variables/set-environment-variables/>`_ , 
`Hint 2 <https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/>`_ 


PART 3
~~~~~~

Create a second Redis client in your Flask app that points to a new "tab" in the Redis
database. In other words, if your main redis client ``rd`` is pointing to ``db=0``, create 
a new client with a new name pointing to ``db=1``. 

Create a new Flask route called ``/image`` which accepts POST, GET, or DELETE requests.

* A POST request to ``/image`` should read some portion of data out of the database (``db=0``), 
  run some matplotlib code to create a simple plot of that data, then write the resulting
  plot back into the database (``db=1``).
* A GET request to ``/image`` should return the image to the user, if present in the database.
* A DELETE request to ``/image`` should delete the image from the database.

Please use defensive programming strategies when writing this route. For example,
what should happen if we POST to the ``/image`` route, but have not yet loaded the 
raw data into the database? What should happen if we GET or DELETE to the ``/image``
route but there is no image in the database yet?

.. note::

   Please note that this part of the code will eventually be moved to a new
   python script (the "Worker") for the final project.


PART 4
~~~~~~

All of the README instructions from Homework 06 and Homework 07 apply to Homework 08.
New instructions should be given to use the ``/image`` route, and new instructions
should be given on how to account for the environment variables in the docker-compose yaml
file and in the Flask deployment file. In other words, do the users need to modify
anything in place of ``<what value goes here?>``, or can you hardcode that in your repo?
Please focus on the Kubernetes deployment of the app - we are looking for step-by-step instructions
on how a user could clone your repo and launch your app on a Kubernetes cluster.




What to Turn In
---------------

A sample Git repository may contain the following new files after completing
homework 06:

.. code-block:: text
   :emphasize-lines: 7-13

   my-coe332-hws/
   ├── homework01/
   │   └── ...
   ├── ...
   ├── homework07/
   │   └── ...
   ├── homework08/
   │   ├── Dockerfile
   │   ├── README.md
   │   ├── docker-compose.yml
   │   ├── gene-api.py
   │   └── kubernetes/          # put kubernetes files in a folder to organize
   │       └── x.yml
   └── README.md



Additional Resources
--------------------

* `Environment Variables in Docker-compose <https://docs.docker.com/compose/environment-variables/set-environment-variables/>`_ 
* `Environment Variables in Kubernetes <https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/>`_ 
* Please find us in the class Slack channel if you have any questions!
