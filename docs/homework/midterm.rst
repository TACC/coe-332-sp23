Midterm Project
===============

**Due Date: Friday, Mar 10, by 5:00 pm CST**

Surf Wax ISS
------------

Scenario: In Homework 05, we added more routes to our ISS tracker. Now, we will
finish off this project by adding a few final routes, automate deployment with
Docker Compose, and write up a short document describing the project.


PART 1
~~~~~~

For the Midterm Project, start a brand new git repository. Copy over the ``iss_tracker.py``
and ``Dockerfile`` that you have been working on from Homework 05. Your current iteration of
``iss_tracker.py`` should have seven functional routes as  described in
`Homework 05 <./homework05.html>`_. For this project, keep the existing
routes and add the following five new routes:

* A ``/comment`` route that returns the 'comment' list object from the ISS data
* A ``/header`` route that returns the 'header' dictionary object from the ISS data
* A ``/metadata`` route that returns the 'metadata' dictionary object from the ISS data
* A ``/epochs/<epoch>/location`` route that returns latitude, longitude, altitude, and 
  geoposition for a given ``<epoch>``. Math required! See slack for hints on this. We
  will also be using the Python GeoPy library for determining geoposition given a 
  latitude and longitude.
* A ``/now`` route that returns the same information as the 'location' route above, but
  for the up-to-date, real time position of the ISS. You will need to iterate over all
  Epochs in the data set and figure out which one is closest to the current time, then
  return location information for that Epcoh. You can confirm whether it is working by
  comparing it to the longitude and latitude of the ISS in real time using an
  `online tracker <https://www.n2yo.com/?s=90027>`_.

After completing the above, your app should have twelve total routes - all routes 
described in `Homework 05 <./homework05.html>`_ plus the following new routes:

+---------------------------------+------------+---------------------------------------------+
| **Route**                       | **Method** | **What it should do**                       |
+---------------------------------+------------+---------------------------------------------+
| ``/comment``                    | GET        | Return 'comment' list obejct from ISS data  |
+---------------------------------+------------+---------------------------------------------+
| ``/header``                     | GET        | Return 'header' dict object from ISS data   |
+---------------------------------+------------+---------------------------------------------+
| ``/metadata``                   | GET        | Return 'metadata' dict object from ISS data |
+---------------------------------+------------+---------------------------------------------+
| ``/epochs/<epoch>/location``    | GET        | Return latitude, longitude, altitude, and   |
|                                 |            | geoposition for given Epoch                 |
+---------------------------------+------------+---------------------------------------------+
| ``/now``                        | GET        | Return latitude, longitude, altidue, and    |
|                                 |            | geoposition for Epoch that is nearest in    |
|                                 |            | time                                        |
+---------------------------------+------------+---------------------------------------------+


Please use defensive programming strategies for your routes. Assume we will try to
break your routes by doing naughty things like sending non-integers into your query parameters,
or querying routes after deleting all the data. Your routes should use exception handling
to catch and handle these bad requests. In addition:

* Write appropriately formatted doc strings for your routes
* Use type annotations where appropriate
* Organize your code in the same way as the other Flask apps we have made

Sample output from the ``/now`` route may look like:

.. code-block:: console

   ubuntu@wallen-vm:~/midterm$ curl localhost:5000/now
   {
     "closest_epoch": "2023-056T20:08:00.000Z",
     "seconds_from_now": -37.382118225097656,
     "location": {
       "latitude": 40.64084133393722,
       "longitude": -255.56465621326345,
       "altitude": {
         "value": 416.8431503828533,
         "units": "km"
       },
       "geo": {
         "banner": "Alxa Right Banner",
         "region": "Alxa League",
         "state": "Inner Mongolia",
         "ISO3166-2-lvl4": "CN-NM",
         "country": "China",
         "country_code": "cn"
       }
     },
     "speed": {
       "value": 7.6671001419187075,
       "units": "km/s"
     }



PART 2
~~~~~~

**Dockerfile.** Write a Dockerfile to containerize your ``iss_tracker.py`` script. You can
start from your Homework 05 Dockerfile, but there are changes required to support the 
additional dependencies. As before, the image should
contain the same version of Python as you are using on the Jetstream VM, plus any other
dependencies (i.e. non-standard Python libraries). The default
command should launch the Flask app in a standard way. Build an image
with a name and version tag of your choosing, and push it to your 
namespace in Docker Hub.

**Docker Compose.** Write a compose file to automate the deployment of your app. The 
compose file should be configured to build your image with your chosen tag, and 
bind the appropriate port inside the container to the appropriate port on the host.


PART 3
~~~~~~

**README.** Standard README guidelines apply. The README should have a descriptive title
(not “Midterm Project”), a short summary / description of the project, short descriptions
of any very important files that the reader should know about, instructions to download
the data and build the containerized app, instructions to pull a pre-containerized copy
of the app from Docker Hub, instructions to interact with the application, and guidelines
on how to interpret the results. Be succinct, use Markdown styles, and show example commands
/ expected output.

**Write Up.** You must also turn in a written document (as a PDF) describing the project.
While the README is typically more succinct and targeted towards developers / users of
the application, the written document should be more verbose and targeted towards a non-user,
but technically savvy layperson. For example, in the README you might instruct a user: “Run
the application by performing xyz”. In the Write Up, you would instead describe what users do:
“Users of the application can run it by performing xyz”. In other words, write this document
as if you are describing to your fellow engineering students what you did in this class for
your Midterm project.

In the Write Up, include some narrative about the motivation of the project and why it is an
interesting or important application to have. You must also include a short section on “Ethical
and Professional Responsibilities in Engineering Situations”. As a suggestion, you may consider
describing the importance of citing data or the importance of verifying the quality and accuracy
of data that goes into applications such as these. We strongly encourage you to come up with other
Ethical and Professional Responsibilities that might apply here.

Please make sure to appropriately cite the data source in both the README and the written document.





What to Turn In
---------------


This Midterm project should be pushed into a standalone repo with a descriptive
name (not “coe332-midterm”). It should not be put into your existing homework repo.
A sample Git repository may contain the following after completing the Midterm:

.. code-block:: text

   Descriptive-Repo-Name/
   ├── Dockerfile               # Please stick to these same file names
   ├── docker-compose.yml     
   ├── iss_tracker.py          
   └── README.md

Send an email to wallen@tacc.utexas.edu with the written PDF summary of the project
attached plus a link to your new GitHub repository. Please include “Midterm Project”
in the subject line. We will clone all of your repos at the due date / time for evaluation.



.. warning::
  
   Do not include the raw XML data as part of your repo.



Additional Resources
--------------------

* `NASA Data Set <https://spotthestation.nasa.gov/trajectory_data.cfm>`_
* `Real Time ISS Position <https://www.n2yo.com/?s=90027>`_
* `Python GeoPy Docs <https://geopy.readthedocs.io/en/stable/#>`_
* Please find us in the class Slack channel if you have any questions!
