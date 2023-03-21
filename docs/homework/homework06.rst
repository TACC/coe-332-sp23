Homework 06
===========

**Due Date: Tuesday, Mar 28, by 11:00am CDT**

Say It Ain't Genes
------------------

Scenario: We are going to turn our attention to a brand new dataset. The Human Genome
Organization (`HUGO <https://en.wikipedia.org/wiki/Human_Genome_Organisation>`_) is a 
non-profit which oversees the HUGO Gene Nomenclature Committee
(`HGNC <https://en.wikipedia.org/wiki/HUGO_Gene_Nomenclature_Committee>`_). The HGNC 
*"approves a unique and meaningful name for every gene"*. For this homework, we will
download the complete set of HGNC data and inject it into a Redis database through
a Flask interface.


This homework will essentially be a re-hash of Exercise 2 and Exercise 3 from the last
`Redis section <../unit06/redis_and_flask.html>`_.


.. note::

   If you want to do this homework with a different data set, e.g. the data set you
   want to work on for your final project, just get it approved with the instructors
   first.


PART 1
~~~~~~

You can find the HGNC data on this page: https://www.genenames.org/download/archive/

Scroll to the bottom and look for the link that says "Current tab separated hgnc_complete_set
file" or "Current JSON format hgnc_complete_set file". Please spend some time reading this
page to help you understand the data. I recommend opening up the tsv file in a spreadsheet
tool (like Excel) to get a better feel for what it contains. Notice there are a lot more 
fields than we are used to (54), but it is the same *list-of-dictionaries* format that
we see over and over again in this class. Another thing to notice is that this dataset
is *sparse*, meaning not every cell has data in it.


PART 2
~~~~~~

Start a new directory for this homework (called ``homework06``). Write a Flask app that
has a ``/data`` route and a ``/gene`` route. The routes should do the following:

* A POST request to ``/data`` should load the HGNC data to a Redis database. Use the Python
  ``requests`` library to get the data directly from the web.
* A GET request to ``/data`` should read all data out of Redis and return it as a JSON list.
* A DELETE request to ``/data`` should delete all data from Redis.
* The ``/genes`` route should return a json-formatted list of all ``hgnc_id`` fields. 
  (This is the first field in the data set, and a unique identifier for each gene).
* The ``/genes/<hgnc_id>`` route should return all data associated with a given ``<hgnc_id>``.
  Be careful to handle the case where something other than a valid gene ID is provided by the user,
  and be careful to handle the sparesely populated data it returns. An example query to this route
  might look like:

.. code-block:: console

   [user-vm]$ curl localhost:5000/genes/HGNC:5
   {
    "hgnc_id": "HGNC:5",
    "symbol": "A1BG",
    "name": "alpha-1-B glycoprotein",
    "locus_group": "protein-coding gene",
    "locus_type": "gene with protein product",
    "status": "Approved",
    "location": "19q13.43",
    "location_sortable": "19q13.43",
    "alias_symbol": "",
    "alias_name": "",
    "prev_symbol": "",
    "prev_name": "",
    "gene_group": "Immunoglobulin like domain containing",
    "gene_group_id": 594,
    "date_approved_reserved": "6/30/89",
    "date_symbol_changed": "",
    "date_name_changed": "",
    "date_modified": "1/20/23",
    "entrez_id": 1,
    "ensembl_gene_id": "ENSG00000121410",
    "vega_id": "OTTHUMG00000183507",
    "ucsc_id": "uc002qsd.5",
    "ena": "",
    "refseq_accession": "NM_130786",
    "ccds_id": "CCDS12976",
    "uniprot_ids": "P04217",
    "pubmed_id": 2591067,
    "mgd_id": "MGI:2152878",
    "rgd_id": "RGD:69417",
    "lsdb": "",
    "cosmic": "",
    "omim_id": 138670,
    "mirbase": "",
    "homeodb": "",
    "snornabase": "",
    "bioparadigms_slc": "",
    "orphanet": "",
    "pseudogene.org": "",
    "horde_id": "",
    "merops": "I43.950",
    "imgt": "",
    "iuphar": "",
    "kznf_gene_catalog": "",
    "mamit-trnadb": "",
    "cd": "",
    "lncrnadb": "",
    "enzyme_id": "",
    "intermediate_filament_db": "",
    "rna_central_ids": "",
    "lncipedia": "",
    "gtrnadb": "",
    "agr": "HGNC:5",
    "mane_select": "ENST00000263100.8|NM_130786.4",
    "gencc": ""
   }


After completing the above, your app should have the following routes:

+-------------------------+------------+--------------------------------------------+
| **Route**               | **Method** | **What it should do**                      |
+-------------------------+------------+--------------------------------------------+
| ``/data``               | POST       | Put data into Redis                        |
+-------------------------+------------+--------------------------------------------+
| ``/data``               | GET        | Return all data from Redis                 |
+-------------------------+------------+--------------------------------------------+
| ``/data``               | DELETE     | Delete data in Redis                       |
+-------------------------+------------+--------------------------------------------+
| ``/genes``              | GET        | Return json-formatted list of all hgnc_ids |
+-------------------------+------------+--------------------------------------------+
| ``/genes/<hgnc_id>``    | GET        | Return all data associated with <hgnc_id>  |
+-------------------------+------------+--------------------------------------------+


Please use defensive programming strategies for your routes with exception handling, and
use doc strings / type annotations as appropriate.



PART 3
~~~~~~

The application should be containerized and orchestrated along side a Redis container.
Write a Dockerfile for containerizing your Flask app, and write a Docker-compose yaml
file for orchestrating the services together. Read very closely the 
`Docker Compose <../unit06/redis_and_flask.html#docker-compose>`_
section of Unit 06 for detailed instructions on how to do this part. Please also push a 
copy of your containerized application to Docker Hub.



PART 4
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
* Launch the containerized app and Redis using docker-compose
* Give example API query commands and expected outputs in code blocks

Finally, your README should also have a section to describe the data itself. Please
give enough information for others to understand what data they are seeing and
what it means (not every field must be described, just a general overview).
Please cite the data appropriately as well.



What to Turn In
---------------

A sample Git repository may contain the following new files after completing
homework 06:

.. code-block:: text
   :emphasize-lines: 7-11

   my-coe332-hws/
   ├── homework01
   │   └── ...
   ├── ...
   ├── homework05
   │   └── ...
   ├── homework06
   │   ├── Dockerfile
   │   ├── docker-compose.yaml
   │   ├── gene_api.py             # please use this name for your file
   │   └── README.md
   └── README.md

Additional Resources
--------------------

* `HGNC Data Set <https://www.genenames.org/download/archive/>`_
* `Unit on Docker Compose <../unit06/redis_and_flask.html#docker-compose>`_
* Please find us in the class Slack channel if you have any questions!
