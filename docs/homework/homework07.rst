Homework 07
===========

**Due Date: Thursday, Apr 6, by 11:00am CST**

In The Kubernetes
-----------------

Scenario: The simple app (Flask API + Redis DB) you built in Homework 06 needs
a little bit of stability and exposure to the rest of the world. For this 
homework, we will write the necessary configurations to deploy the app onto the
class Kubernetes cluster.

This homework is the Lab exercise at the end of the 
`Unit 07 <../unit07/services.html#homework-7-deploying-our-flask-api-to-k8s>`_


.. note::

   If you want to do this homework with a different data set, e.g. the data set you
   want to work on for your final project, just get it approved with the instructors
   first.


PART 1
~~~~~~

Start a new directory for this homework (called ``homework07``). Copy over all the
assets from ``homework06`` into this new directory.

Complete the in-class exercise from the end of the
`Kubernetes section <../unit07/services.html#homework-7-deploying-our-flask-api-to-k8s>`_.
Successful completion of this exercise requires:

* A persistent volume claim for your Redis data
* A deployment for your Redis database
* A service for your Redis database
* A deployment for your Flask API
* A service for your Flask API


PART 2
~~~~~~

Update the Dockerfile and Flask API python script for your app if necessary.
Special consideration should be given to the host IP address for the Redis client
as declared in your Flask app.


PART 3
~~~~~~

All of the README instructions from Homework 06 apply to Homework 07. New instructions
should be given for deploying the software system to a Kubernetes cluster. Please note
that in order for the Flask container to get deployed to Kubernetes, it must come from 
Docker Hub. Please guide users on the steps to build their own image and push to Docker
Hub, and guide users on using your existing image from Docker Hub. You will also need
to guide users on any modifications required in the Kubernetes file or Python script,
including image name and IP address.



What to Turn In
---------------

A sample Git repository may contain the following new files after completing
homework 06:

.. code-block:: text
   :emphasize-lines: 7-12

   my-coe332-hws/
   ├── homework01
   │   └── ...
   ├── ...
   ├── homework06
   │   └── ...
   ├── homework07
   │   ├── Dockerfile
   │   ├── docker-compose.yaml
   │   ├── gene_api.py
   │   ├── README.md
   │   └── *.yml              # should be 5 yml files
   └── README.md







Additional Resources
--------------------

* `Kubernetes Lab <../unit07/services.html#homework-7-deploying-our-flask-api-to-k8s>`_
* Please find us in the class Slack channel if you have any questions!
