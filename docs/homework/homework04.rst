Homework 04
===========

**Due Date: Tuesday, Feb 21, by 11:00am CST**

Buddy Flask
-----------

Scenario: You have found an abundance of interesting positional and velocity
data for the International Space Station (ISS). It is a challenge, however, to
sift through the data manually to find what you are looking for. Your objective
is to build a Flask application for querying and returning interesting
information from the ISS data set.


PART 1
~~~~~~

First, navigate to the 
`ISS Trajectory Data <https://spotthestation.nasa.gov/trajectory_data.cfm>`_
website. 

There are two downloads available, and they contain the same data. It is the 
'Orbital Ephemeris Message (OEM)' data in either plain text or XML format.
Download and examine both files to get a feel for what they contain. Closely 
read the ~3 paragraphs of accompanying text on the website as well.

The most important thing to understand is that the data contains ISS state
vectors over an ~15 day period. The range of days may change depending on
when you download the data set. 'State vectors' are the Cartesion vectors
for both position ``{X, Y, Z}`` and velocity ``{X_DOT, Y_DOT, Z_DOT}`` that,
along with a time stamp ``(EPOCH)``, describe the complete state of the system
(the ISS).  The frame of reference for these vectors is Earth, based on something
called the J2000 reference frame.

.. tip::

   The XML data is probably a little bit easier than the raw text data to
   ingest into dictionary format inside a python script. See the 
   `Unit on XML <../unit02/xml.html>`_ for tips.



PART 2
~~~~~~

Write a Flask application for querying the ISS position and velocity. The
application should load in the data from the link above, and expose
that data to an end user with carefully crafted routes. At a minimum, write
routes so the user can return:

+---------------------------+----------------------------------------+
| **Route**                 | **What it should return**              |
+---------------------------+----------------------------------------+
| ``/``                     | The entire data set                    |
+---------------------------+----------------------------------------+
| ``/epochs``               | A list of all Epochs in the data set   |
+---------------------------+----------------------------------------+
| ``/epochs/<epoch>``       | State vectors for a specific Epoch     |
|                           | from the data set                      |
+---------------------------+----------------------------------------+
| ``/epochs/<epoch>/speed`` | Instantaneous speed for a specific     |
|                           | Epoch in the data set (math required!) |
+---------------------------+----------------------------------------+


All of the normal Python3 script best practices apply:

* Write appropriately formatted doc strings for your routes
* Use type annotations where appropriate
* Organize your code in the same way as the other Flask apps we have made


.. tip::

   For the final route, you will need an equation to calculate speed from
   Cartesian velocity vectors: **speed = sqrt(x_dot^2 + y_dot^2 + z_dot^2)**



PART 3
~~~~~~

The homework must also include a README file. The README should be descriptive,
use proper grammar, and contain enough instructions so anyone else could clone
the repository and figure out how to run your app and interact with it. 
Guidelines to follow for the README are:

* Descriptive title
* High-level description of the folder contents / project objective. I.e. why
  does this exist and why is it important? (2-3 sentences)
* Instructions on how to access and description of the data set from the original source
  (do not include the data itself in this repository!)
* Specific description of the Flask app (1-2 sentences)
* Instructions to run the app
* Example queries to the running API and expected results / how to interpret the results
  (at least one example per route; some routes will return very long results, so please 
  truncate the results when you put them in your README)
* Try to use markdown styles to your advantage, give the sections headers, use
  code blocks where appropriate, etc.

Remember, the README is your chance to document for yourself and explain to others
why the project is important, what the code is, and how to use it / interpret
the outputs / etc. This is a *software engineering and design* class, so we are
not just checking to see if your code works. We are also evaluating the design of
the overall submission, including how well the project is described in the README.




What to Turn In
---------------

A sample Git repository may contain the following new files after completing
homework 04:

.. code-block:: text
   :emphasize-lines: 8-10

   my-coe332-hws/
   ├── homework01
   │   └── ...
   ├── homework02
   │   └── ...
   ├── homework03
   │   └── ...
   ├── homework04
   │   ├── iss_tracker.py       # your flask app name may vary
   │   └── README.md
   └── README.md

**Do not include the raw XML data as part of your repo.**

There is no need to email the link to your homework repo again, as we should have
it on file from the first homework. We will re-clone the same repo as before at the
due date / time for evaluation.





Additional Resources
--------------------

* `NASA Data Set <https://spotthestation.nasa.gov/trajectory_data.cfm>`_
* `Info on State Vectors <https://en.wikipedia.org/wiki/Orbital_state_vectors>`_
* `Info on Reference Frame <https://en.wikipedia.org/wiki/Earth-centered_inertial>`_
* `Unit on XML <../unit02/xml.html>`_
* Please find us in the class Slack channel if you have any questions!

