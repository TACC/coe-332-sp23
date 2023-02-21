Homework 05
===========

**Due Date: Tuesday, Feb 28, by 11:00am CST**

Undone (The Sweater Container)
------------------------------

Scenario: The API you developed for the ISS positional and velocity data (in homework 04) 
is a great start! But, there are many improvements we could make in order
to make the API more useful. And, we can use some smart software engineering & design
strategies to make our app more **portable**.


PART 1
~~~~~~

Start a new directory for this homework (called ``homework05``) and copy the Flask app
from homework 04 into this new directory. If it isn't already, please name the file
``iss_tracker.py``. Add the following new features to the Flask app:

* Add query parameters ``offset`` and ``limit`` to the ``/epochs`` route. The ``offset``
  query parameter should offset the start point by an integer. For example, ``offset=0``
  would begin printing at the first Epoch, ``offset=1`` would begin printing at the
  second Epoch, etc. The ``limit`` query parameter controls how many results are returned.
  For example ``limit=10`` would return 10 Epochs, ``limit=100`` would return 100 Epochs,
  etc. Putting them together, the route ``/epochs?limit=20&offset=50`` would return Epochs 
  51 through 70 (20 total).
* Add a ``/help`` route that returns a human readable string to the screen. The string should
  give brief descriptions of all available routes (plus their methods) for your API. The
  returned text should be similar output to a command line tool. For example, look at the 
  output returned when you type ``git --help`` or ``docker --help``
* Add a ``/delete-data`` route that deletes everything from the in-memory dictionary of ISS
  data. Make sure to use a ``DELETE`` method for this route. After curling the
  ``/delete-data`` route, other routes (e.g ``/epochs``) should return no data.
* Add a ``/post-data`` route that restores the data to the ISS dictionary by performing a 
  new GET request using the Python requests library. Although inside this route you will
  be performing a GET request, the route itself should only accept a POST method.


After completing the above, your app should have the following routes:

+----------------------------------+------------+--------------------------------------------+
| **Route**                        | **Method** | **What it should do**                      |
+----------------------------------+------------+--------------------------------------------+
| ``/``                            | GET        | Return entire data set                     |
+----------------------------------+------------+--------------------------------------------+
| ``/epochs``                      | GET        | Return list of all Epochs in the data set  |
+----------------------------------+------------+--------------------------------------------+
| ``/epochs?limit=int&offset=int`` | GET        | Return modified list of Epochs given query |
|                                  |            | parameters                                 |
+----------------------------------+------------+--------------------------------------------+
| ``/epochs/<epoch>``              | GET        | Return state vectors for a specific Epoch  |
|                                  |            | from the data set                          |
+----------------------------------+------------+--------------------------------------------+
| ``/epochs/<epoch>/speed``        | GET        | Return instantaneous speed for a specific  |
|                                  |            | Epoch in the data set (math required!)     |
+----------------------------------+------------+--------------------------------------------+
| ``/help``                        | GET        | Return help text (as a string) that        |
|                                  |            | briefly describes each route               |
+----------------------------------+------------+--------------------------------------------+
| ``/delete-data``                 | DELETE     | Delete all data from the dictionary object |
+----------------------------------+------------+--------------------------------------------+
| ``/post-data``                   | POST       | Reload the dictionary object with data     |
|                                  |            | from the web                               |
+----------------------------------+------------+--------------------------------------------+

Please use defensive programming strategies for your routes. Assume we will try to
break your routes by doing naughty things like sending non-integers into your query parameters,
or querying routes after deleting all the data. Your routes should use exception handling
to catch and handle these bad requests.



PART 2
~~~~~~

Write a Dockerfile to containerize your ``iss_tracker.py`` script. The image should
contain the same version of Python as you are using on the Jetream VM, plus any other
dependencies (i.e. non-standard Python libraries). The default
command should launch the Flask app in a standard way. Build an image
with the version tag similar to ``username/iss_tracker:hw05`` and push it to your 
namespace in Docker Hub.



PART 3
~~~~~~

You've written the code, you've containerized it, now you need to tell other
people how to use it. The problem is that you are not quite sure *how* people will
want to use this code. In the real world, developers and end users will find your
repo, and some of them will want to use your pre-built Docker image to run the
code, while others will want to start from your Dockerfile and build their own
image. The challenge is to write a README that caters to all audiences.

Write a README with the standard sections from previous homeworks: there should
be a descriptive title, there should be a high level description of the project,
there should be concise descriptions of the main files within, and you should
be using Markdown styles and formatting to your advantage.

The section of the README with run instructions will be a little more challenging.
Please make sure to include instructions to (at a minimum):

* Pull and use your existing image from Docker Hub
* Build a new image from your Dockerfile
* Run the containerized Flask app being careful to map ports correctly
* Give example API query commands and expected outputs in code blocks

Finally, your README should also have a section to describe the ISS data. Please
give enough information for others to understand what data they are seeing and
what it means (including units). Please cite the data appropriately as well.



What to Turn In
---------------

A sample Git repository may contain the following new files after completing
homework 05:

.. code-block:: text
   :emphasize-lines: 7-10

   my-coe332-hws/
   ├── homework01
   │   └── ...
   ├── ...
   ├── homework04
   │   └── ...
   ├── homework05
   │   ├── Dockerfile
   │   ├── iss_tracker.py       # please use this name for your file
   │   └── README.md
   └── README.md

Additional Resources
--------------------

* `NASA Data Set <https://spotthestation.nasa.gov/trajectory_data.cfm>`_
* Please find us in the class Slack channel if you have any questions!
