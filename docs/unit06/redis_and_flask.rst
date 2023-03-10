Redis and Flask
===============

Now that we have gained some experience with starting a containerized Redis 
service and interacting with that service from a simple Python script, it is
time to integrate it into a larger software project. We will next learn how to
mount a host volume inside the Redis container in order to persist the database,
incorporate the Redis service into our Docker compose files, and interact with
the Redis service from our Flask apps. After going through this module, students
should be able to:

  * Mount a host volume inside a Redis container to save data between restarts of Redis itself.
  * Pass an argument into the Redis container to influence the saving behavior.
  * Connect to a Redis container from within a Flask application.
  * Incorporate a Redis service into a Docker compose file.
  * **Design Principles:** Separating the database service from the 
    application container is a great example illustrating the *modularity* design
    principle.


Recap: Start a Redis Container
------------------------------

Recall from last time we started a Redis container and mapped port ``6379``
inside the container to port ``6379`` on the host. This allows us to interact
with the Redis server on that port.


.. code-block:: console

   # start the Redis server on the command line:
   [user-vm]$ docker run -p 6379:6379 redis:7
   1:C 01 Mar 2023 22:01:28.798 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
   1:C 01 Mar 2023 22:01:28.798 # Redis version=7.0.9, bits=64, commit=00000000, modified=0, pid=1, just started
   1:C 01 Mar 2023 22:01:28.798 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
   1:M 01 Mar 2023 22:01:28.799 * monotonic clock: POSIX clock_gettime
   1:M 01 Mar 2023 22:01:28.799 * Running mode=standalone, port=6379.
   1:M 01 Mar 2023 22:01:28.799 # Server initialized
   1:M 01 Mar 2023 22:01:28.799 # WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition. Being disabled, it can can also cause failures without low memory condition, see https://github.com/jemalloc/jemalloc/issues/1328. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
   1:M 01 Mar 2023 22:01:28.800 * Ready to accept connections



The above command will start the Redis container in the foreground which can be
helpful for debugging, seeing logs, etc. However, it may also be useful to
start it in the background (detached or daemon mode). You can do that by adding the
``-d`` flag:

.. code-block:: console

   [user-vm]$ docker run -d -p 6379:6379 redis:7
   3a28cb265d5e09747c64a87f904f8184bd8105270b8a765e1e82f0fe0db82a9e

At this point, Redis is running and available from your host. Double check this is 
true by executing ``docker ps -a``. Let's explore the persistence of the data in Redis.

We can open a Python shell, connect to our Redis container, and add some data:

.. code-block:: python3

    >>> import redis
    >>> rd = redis.Redis(host='127.0.0.1', port=6379, db=0)

    # create some data
    >>> rd.set('k', 'v')
    True

    # check that we can read the data
    >>> rd.get('k')
    b'v'

If we exit our Python shell and then go into a new Python shell and connect to Redis,
what do we see?

.. code-block:: python3

    >>> import redis
    >>> rd = redis.Redis(host='127.0.0.1', port=6379, db=0)

    # previous data is still there
    >>> rd.get('k')
    b'v'

Great! Redis has persisted our data across Python sessions. That was one of our goals.
But what happens if we shut down the Redis container itself? Let's find out by killing
our Redis container and starting a new one.

.. code-block:: console

   # shut down the existing redis container using the name you gave it
   [user-vm]$ docker rm -f <redis container id>

   # start a new redis container
   [user-vm]$ docker run -d -p 6379:6379 redis:7


Now go back into the Python shell and connect to Redis:

.. code-block:: python3

    >>> import redis
    >>> rd = redis.Redis(host='127.0.0.1', port=6379, db=0)

    # previous data is gone!
    >>> rd.get('k')

    # no keys at all!
    >>> rd.keys()
    []

All the data that was in Redis is gone. The problem is we are not permanently persisting
the Redis data across different Redis containers. But wasn't that the whole point of using a
database? Are we just back to where we started?

Actually, we only need a few small changes to the way we are running the Redis container to make the
Redis data persist across container executions.

Modifications Required to Support Data Persistence
--------------------------------------------------

Container Bind Mounts
~~~~~~~~~~~~~~~~~~~~~

A container bind mount (or just "mount" for short) is a way of replacing a file or directory in a
container image with a file or directory on the host file system in a running container.

Bind mounts are specified with the ``-v`` flag to the ``docker run`` statement. The full syntax
is

.. code-block:: console

    [user-vm]$ docker run -v <host_path>:<container_path>:<mode> ...*additional docker run args*...

where:

  * ``<host_path>`` and ``<container_path>`` are absolute paths in the host and container file
    systems, and
  * ``<mode>`` can take the value of ``ro`` for a read-only mount and ``rw`` for a read-write mount.

.. note::
  
   Note that ``mode`` is optional and defaults to read-write.

It is important to keep the following in mind when using bind mounts:

  * If the container image originally contained a file or directory at the ``<container_path>``
    these will be replaced entirely by the contents of ``<host_path>``.
  * If the container image did not contain contents at ``<container_path>`` the mount will still
    succeed and simply create a new file/directory at the path.
  * If the ``<mode>`` is read-write (the default), any changes made by the running container will be
    reflected on the host file system. Note that the process running in the container still must have
    permission to write to the path.
  * If ``<host_path>`` does not exist on the host, Docker will create a **directory** at the path
    and mount it into the container. **This may or may not be what you want.**


Save Data to File Periodically
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We can use bind mounts to persist Redis data across container executions: the key point is that Redis
can be started in a mode so that it periodically writes all of its data to the host.

From the `Redis documentation <https://redis.io/docs/management/persistence/>`_, we see that we need to
set the ``--save`` flag when starting Redis so that it writes its dataset to the file system periodically.
The full syntax is:

.. code-block:: console

  --save <frequency> <number_of_backups>

where ``<frequency>`` is an integer, in seconds. We'll instruct Redis to write its data to the file
system every second, and we'll keep just one backup.


Entrypoints and Commands in Docker Containers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Let's take a moment to revisit the difference between an entrypoint and a
command in a Docker container image. When executing a container from an image, Docker uses both
an ``entrypoint`` and an (optional) ``command`` to start the container. It combines the two using
concatenation, with ``entrypoint`` first, followed by ``command``.

When we use ``docker run`` to create and start a container from an existing image, we can choose to
override either the command or the entrypoint that may have been specified in the image. Any string
``<string>`` passed after the ``<image>`` in the statement:

.. code-block:: console

   [user-vm]$ docker run <options> <image> <string>

will override the ``command`` specified in the image, but the original entrypoint set for the image
will still be used.

A common pattern when building Docker images is to set the ``entrypoint`` to the primary program,
and set the ``command`` to a default set of options or parameters to the program.

Consider the following simple example of a Dockerfile:

.. code-block:: Dockerfile

  FROM ubuntu
  ENTRYPOINT ["ls"]
  CMD ["-l"]

If we build and tag this image as ``test/ls``, then:

.. code-block:: console

  # run with the default command, equivalent to "ls -l"
  [user-vm]$ docker run --rm -it test/ls
  total 48
  lrwxrwxrwx   1 root root    7 Jan  5 16:47 bin -> usr/bin
  drwxr-xr-x   2 root root 4096 Apr 15  2020 boot
  drwxr-xr-x   5 root root  360 Mar 23 18:37 dev
  drwxr-xr-x   1 root root 4096 Mar 23 18:37 etc
  drwxr-xr-x   2 root root 4096 Apr 15  2020 home
  . . .

  # override the command, but keep the entrypoint; equivalent to running "ls -a" (note the lack of "-l")
  [user-vm]$ docker run --rm -it test/ls -a
  .   .dockerenv	boot  etc   lib    lib64   media  opt	root  sbin  sys  usr
  ..  bin		dev   home  lib32  libx32  mnt	  proc	run   srv   tmp  var


  # override the command, specifying a different directory
  [user-vm]$ docker run --rm -it test/ls -la /root 
  total 16
  drwx------ 2 root root 4096 Jan  5 16:50 .
  drwxr-xr-x 1 root root 4096 Mar 23 18:38 ..
  -rw-r--r-- 1 root root 3106 Dec  5  2019 .bashrc
  -rw-r--r-- 1 root root  161 Dec  5  2019 .profile


Modifying the Command in the Redis Container
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The official `redis <https://hub.docker.com/_/redis>`_ container image provides an entrypoint which
starts the redis server (check out the `Dockerfile <https://github.com/docker-library/redis/blob/15ed0a0c1cb60c5193db45d8b59a8707507be307/7.0-rc/Dockerfile>`_
if you are interested.).

Since the ``save`` option is a parameter, we can set it when running the redis server container
by simply appending it to the end of the ``docker run`` command; that is,

.. code-block:: console

  [user-vm]$ docker run <options> redis:7 --save <options>


Bring it All Together for a Complete Solution
---------------------------------------------

With ``save``, we can instruct Redis to write the data to the file system, but we still need to
save the files across container executions. That's where the bind mount comes in. But how do we know
which directory to mount into Redis? Fortunately, the Redis documentation tells us what we need to know:
Redis writes data to the ``/data`` directory in the container.

Putting all of this together, we can update the way we run our Redis container as follows:


.. code-block:: console

   [user-vm]$ docker run -d -p 6379:6379 -v </path/on/host>:/data redis:7 --save 1 1

.. tip::

  You can use the ``$(pwd)`` shortcut for the present working directory.

For example, I might use:

.. code-block:: console

  [user-vm]$ docker run -d -p 6379:6379 -v $(pwd)/data:/data:rw redis:7 --save 1 1

Now, Redis should periodically write all of its state to the ``data`` directory. You should see a
file called ``dump.rdb`` in the directory because we are using the default persistence mechanism
for Redis. This will suffice for our purposes, but Redis has other options for persistence which
you can read about `here <https://redis.io/docs/management/persistence/>`_ if interested.


EXERCISE 1
~~~~~~~~~~

Test out persistence of your Redis data across Redis container restarts by starting a new Redis container
using the method above, saving some data to it in a Python shell, shutting down the Redis container
and starting a new one, and verifying back in the Python shell that the original data is still there.


Using Redis in Flask
--------------------
Using Redis in our Flask apps is identical to using it in the Python shells that we have been using
to explore with. We simply create a Python Redis client object using the ``redis.Redis()`` constructor.
Since we might want to use Redis from different parts of the code, we'll create a function for
generating the client:

.. code-block:: python3
   :linenos:

    def get_redis_client():
         return redis.Redis(host='127.0.0.1', port=6379, db=0)
      
    rd = get_redis_client()

Then, within the function definitions for your routes, use the ``rd`` client to set or get data
as appropriate, e.g.:

.. code-block:: python3
   :linenos:

   @app.route('/get-keys', methods=['GET'])
   def get_keys():
       return rd.keys()



EXERCISE 2
~~~~~~~~~~

In the last module, we wrote some code to put the Meteorite Landings data (i.e., the
``Meteorite_Landings.json`` file from Unit 2/3) into Redis. In this exercise, let's turn this
function into a Flask API with one route that handles POST, GET, and DELETE requests.

* Use ``/data`` as the URL path for the one route.
* A POST request to ``/data`` should load the Meteorite Landings data into Redis.
* A GET request to ``/data`` should read the data out of Redis and return it as a JSON list.
* A DELETE request to ``/data`` should delete the Meteorite Landings data from Redis.

Access the Meteorite Landings data `at this link1 <https://raw.githubusercontent.com/TACC/coe-332-sp23/main/docs/unit06/scripts/Meteorite_Landings.json>`_

A sample template for the Flask app can be found `at this link2 <https://raw.githubusercontent.com/TACC/coe-332-sp23/main/docs/unit06/scripts/flask_app_template.py>`_




Docker Compose
--------------

We now have two services to consider: Flask and Redis. We need to launch one container for Flask,
and a separate container for Redis. Each service requires a long list of options to the ``docker run``
command,  and it would be convenient to launch them as a unit. This is where the concept of container
orchestration really starts to become useful.

Consider the following update to your Docker compose file from Unit 5:


.. code-block:: yaml

    ---
    version: "3"

    services:
        redis-db:
            image: redis:7
            ports:
                - 6379:6379
            volumes:
                - ./data:/data
            user: "1000:1000"
        flask-app:
            build:
                context: ./
                dockerfile: ./Dockerfile
            depends_on:
                - redis-db
            image: username/ml_flask_app:1.0
            ports:
                - 5000:5000
            volumes:
                - ./config.yaml:/config.yaml


One thing to note is a new specification under the ``flask-app`` service called '``depends_on``'. 
This ensures that the ``redis-db`` service is up before starting the ``flask-app`` service. 
Why might this be important? 

Another question to ask yourself is why does the ``redis-db`` service omit the ``build``
specification? Should we provide a context and Dockerfile for Redis?

One slight modification to the Flask app is required to enable container-to-container
communication over the Docker bridge network. Instead of using an IP address like '127.0.0.1'
or an alias like 'localhost', use the docker-compose alias for the Redis service,
``redis-db``. Specifically, in your containerized Flask app, establish a Redis client like:

.. code-block:: python3

    rd = redis.Redis(host='redis-db', port=6379, db=0)


Given the above, try launching both services using the following command:

.. code-block:: console

    [user-vm]$ docker-compose up -d
    Creating network "redis_default" with the default driver                                                                               
    Creating redis_flask-app_1 ... done                                                                                                    
    Creating redis_redis-db_1  ... done     
    [user-vm]$ docker ps -a
    CONTAINER ID   IMAGE                      COMMAND                  CREATED         STATUS         PORTS                                       NAMES
    d266fbd99e4c   redis:7                    "docker-entrypoint.s…"   3 seconds ago   Up 2 seconds   0.0.0.0:6379->6379/tcp, :::6379->6379/tcp   redis_redis-db_1
    193f057687a8   wjallen/ml_flask_app:1.0   "python3 /ml_flask_a…"   3 seconds ago   Up 2 seconds   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   redis_flask-app_1

You should see the containers running. When you are ready to kill the services:

.. code-block:: console

   [user-vm]$ docker-compose down
   Stopping redis_redis-db_1  ... done
   Stopping redis_flask-app_1 ... done
   Removing redis_redis-db_1  ... done             
   Removing redis_flask-app_1 ... done                                                                                                    
   Removing network redis_default         
   [user-vm]$ docker ps -a


EXERCISE 3
~~~~~~~~~~

* Launch both services using Docker compose
* Query the appropriate route to POST the ML data, and ensure that it is in the database
* Take all services back down and ensure both containers have stopped running
* Launch both services again using Docker compose
* Query the appropriate route to see if the data is still in the database



Additional Resources
--------------------

* `Redis Docs <https://redis.io/documentation>`_
* `Redis Python Library <https://redis-py.readthedocs.io/en/stable/>`_
* `Try Redis in a Browser <https://try.redis.io/>`_
* `Semantic Versioning <https://semver.org/>`_
