Introduction to Containers
==========================

Containers are an important common currency for app development, web services,
scientific computing, and more. Containers allow you to package an application
along with all of its dependencies, isolate it from other applications and
services, and deploy it consistently and reproducibly and *platform-agnostically*.
In this introductory module, we will learn about containers and their uses, in
particular the containerization platform **Docker**.

After going through this module, students should be able to:

* Describe what a container is
* Use essential docker commands
* Find and pull existing containers from Docker Hub
* Run containers interactively and non-interactively



What is a Container?
--------------------

* A container is a standard unit of software that packages up code and all its
  dependencies so the application runs quickly and reliably from one computing
  environment to another.
* Containers allow a developer to package up an application with all of the
  parts it needs, such as libraries and other dependencies, and ship it all out
  as one package.
* Multiple containers can run on the same machine and share the OS kernel with
  other containers, each running as isolated processes in user space, hence are
  *lightweight* and have *low overhead*.
* Containers ensure *portability* and *reproducibility* by isolating the
  application from environment.


How is a Container Different from a VM?
---------------------------------------

Virtual machines enable application and resource isolation, run on top of a
hypervisor (high overhead). Multiple VMs can run on the same physical
infrastructure - from a few to dozens depending on resources. VMs take up more
disk space and have long start up times (~minutes).

.. figure:: images/arch_vm.png
   :width: 400
   :align: center

   Applications isolated by VMs.

Containers enable application and resource isolation, run on top of the host
operating system. Many containers can run on the same physical infrastructure -
up to 1,000s depending on resources. Containers take up less disk space than VMs
and have very short start up times (~100s of ms).

.. figure:: images/arch_container.png
   :width: 400
   :align: center

   Applications isolated by containers.



Docker
------

Docker is a containerization platform that uses OS-level virtualization to
package software and dependencies in deliverable units called containers. It is
by far the most common containerization platform today, and most other container
platforms are compatible with Docker. (E.g. Singularity and Shifter are two
containerization platforms you'll find in HPC environments).

We can find existing containers at:

1. `Docker Hub <https://hub.docker.com/>`_
2. `Quay.io <https://quay.io/>`_
3. `BioContainers <https://biocontainers.pro/#/>`_


Some Quick Definitions
----------------------

Container
~~~~~~~~~

A container is a standard unit of software that packages up code and all its
dependencies so the application runs quickly and reliably from one computing
environment to another. Containers includes everything from the operating
system, user-added files, metadata.

Image
~~~~~

A Docker images is a read-only file used to produce Docker containers. It is
comprised of layers of other images, and any changes made to an image can only
be saved and propagated on by adding new layers. The "base image" is the
bottom-most layer that does not depend on any other layer and typically defines,
e.g., the operating system for the container. Running a Docker image creates an
instance of a Docker container.

Dockerfile
~~~~~~~~~~

The Dockerfile is a recipe for creating a Docker image. They are simple, usually
short plain text files that contain a sequential set of commands (*a recipe*)
for installing and configuring your application and all of its dependencies. The
Docker command line interface is used to "build" an image from a Dockerfile.

Image Registry
~~~~~~~~~~~~~~

The Docker images you build can be stored in online image registries, such as
`Docker Hub <https://hub.docker.com/>`_. (It is similar to the way we store
Git repositories on GitHub.) Image registries support the notion of tags on
images to identify specific versions of images. It is mostly public, and many
"official" images can be found.

Summing Up
----------

If you are developing an app or web service, you will almost certainly want to
work with containers. First you must either **build** an image from a
Dockerfile, or **pull** an image from a public registry. Then, you **run** (or
deploy) an instance of your image into a container. The container represents
your app or web service, running in the wild, isolated from other apps and
services.

.. figure:: images/docker_workflow.png
   :width: 600
   :align: center

   Simple Docker workflow.



Getting Started With Docker
---------------------------

Much like the ``git`` command line tools, the ``docker`` command line tools
follow the syntax: ``docker <verb> <parameters>``. Discover all the verbs
available by typing ``docker --help``, and discover help for each verb by typing
``docker <verb> --help``. Open up your favorite terminal, log in to your student
server, and try running the following:

.. code-block:: console

   [user-vm]$ docker version
   Client:
      Version:           20.10.12
      API version:       1.41
      Go version:        go1.16.2
      Git commit:        20.10.12-0ubuntu2~20.04.1
      Built:             Wed Apr  6 02:14:38 2022
      OS/Arch:           linux/amd64
      Context:           default
      Experimental:      true

   Server:
     Engine:
      Version:          20.10.12
      API version:      1.41 (minimum version 1.12)
      Go version:       go1.16.2
      Git commit:       20.10.12-0ubuntu2~20.04.1
      Built:            Thu Feb 10 15:03:35 2022
      OS/Arch:          linux/amd64
      Experimental:     false
      containerd:
   Version:          1.5.9-0ubuntu1~20.04.6
      GitCommit:        
      runc:
      Version:          1.1.0-0ubuntu1~20.04.2
   GitCommit:        
      docker-init:
      Version:          0.19.0
      GitCommit:        


.. warning::

   Please let the instructors know if you get any errors on issuing the above
   command.

EXERCISE
~~~~~~~~

Take a few minutes to run ``docker --help`` and a few examples of
``docker <verb> --help`` to make sure you can find and read the help text.


Working with Images from Docker Hub
-----------------------------------

To introduce ourselves to some of the most essential Docker commands, we will go
through the process of listing images that are currently available on your student
server, we will pull a 'hello-world' image from Docker Hub, then we will run the
'hello-world' image to see what it says.

List images on your server with the ``docker images`` command. This peaks
into the Docker daemon, to see which images are available, when they were created, 
and how large they are:

.. code-block:: console

   [user-vm]$ docker images
   REPOSITORY            TAG       IMAGE ID       CREATED        SIZE
   guacamole/guacamole   <none>    a385e28f9fd6   8 months ago   642MB
   guacamole/guacd-dev   <none>    315a12ba560b   9 months ago   228MB


Pull an image from Docker hub with the ``docker pull`` command. This looks
through the Docker Hub registry and downloads the 'latest' version of that
image:

.. code-block:: console

   [user-vm]$ docker pull hello-world
   Using default tag: latest
   latest: Pulling from library/hello-world
   0e03bdcc26d7: Pull complete
   Digest: sha256:31b9c7d48790f0d8c50ab433d9c3b7e17666d6993084c002c2ff1ca09b96391d
   Status: Downloaded newer image for hello-world:latest
   docker.io/library/hello-world:latest


Run the image we just pulled with the ``docker run`` command. In this case,
running the container will execute a simple shell script inside the container
that has been configured as the 'default command' when the image was built:

.. code-block:: console

   [user-vm]$ docker run hello-world

   Hello from Docker!
   This message shows that your installation appears to be working correctly.

   To generate this message, Docker took the following steps:
    1. The Docker client contacted the Docker daemon.
    2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
       (amd64)
    3. The Docker daemon created a new container from that image which runs the
       executable that produces the output you are currently reading.
    4. The Docker daemon streamed that output to the Docker client, which sent it
       to your terminal.

   To try something more ambitious, you can run an Ubuntu container with:
    $ docker run -it ubuntu bash

   Share images, automate workflows, and more with a free Docker ID:
    https://hub.docker.com/

   For more examples and ideas, visit:
    https://docs.docker.com/get-started/


Check to see if any containers are still running using ``docker ps``:

.. code-block:: console

   [user-vm]$ docker ps
   CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES


**EXERCISE**

The command ``docker ps`` shows only currently running containers. Pull up the
help text for that command and figure out how to show all containers, not just
currently running containers.


Pull An Official Image
----------------------

One powerful aspect of developing with containers and the Docker ecosystem is the 
large collection of container images freely available. There are 10s of millions of
images on Docker Hub alone, but beware: using an image that you don't know anything 
about comes with the same risks involved with running any software.

.. warning::
   Be careful running container images that you are not familiar with. Some could contain 
   security vulnerabilities or, even worse, malicious code like viruses or ransomware. 

To combat this, Docker Hub provides `"Official Images" <https://docs.docker.com/docker-hub/official_images/>`_,
a well-maintained set of container images providing high-quality installations of operating
systems, programming language environments and more.

We can seearch through the official images on Docker Hub `here <https://hub.docker.com/search?image_filter=official&q=&type=image>`_.

Scroll down to find the Python official image called ``python``, then 
click on that `image <https://hub.docker.com/_/python>`_.

We see a lot of information about how to use the image, including information about the different 
"tags" available. We see tags such as ``3.12-rc``, ``3.11.2``, ``3.11``, ``3``, etc.
We'll discuss tags in detail later, but for now, does anyone have a guess as to what
the Python tags refer to? 

We can pull the official Python image using command, then check to make sure it is
available locally:

.. code-block:: console

   [isp02]$ docker pull python
   ...
   [isp02]$ docker images
   ...
   [isp02]$ docker inspect python
   ...

.. tip::

   Use ``docker inspect`` to find some metadata available for each image.



Start an Interactive Shell Inside a Container
---------------------------------------------

Using an interactive shell is a great way to poke around inside a container and
see what is in there. Imagine you are ssh-ing to a different Linux server, have
root access, and can see what files, commands, environment, etc., is available.

Before starting an interactive shell inside the container, execute the following
commands on the ISP server (we will see why in a minute):

.. code-block:: console

   [user-vm]$ whoami
   ubuntu
   [user-vm]$ pwd
   /home/ubuntu
   [user-vm]$ cat /etc/os-release
   NAME="Ubuntu"
   VERSION="20.04.5 LTS (Focal Fossa)"
   ID=ubuntu
   ID_LIKE=debian
   PRETTY_NAME="Ubuntu 20.04.5 LTS"
   VERSION_ID="20.04"
   HOME_URL="https://www.ubuntu.com/"
   SUPPORT_URL="https://help.ubuntu.com/"
   BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
   PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
   VERSION_CODENAME=focal
   UBUNTU_CODENAME=focal

Now start the interactive shell inside a Python container:

.. code-block:: console

   [user-vm]$ docker run --rm -it python /bin/bash
   root@fc5b620c5a88:/#

Here is an explanation of the command options:

.. code-block:: text

  docker run       # run a container
  --rm             # remove the container when we exit
  -it              # interactively attach terminal to inside of container
  python           # use the official python image 
  /bin/bash        # execute the bash shell program inside container

Try the following commands - the same commands you did above before staring the
interactive shell in the container - and note what has changed:

.. code-block:: console

   root@fc5b620c5a88:/# whoami
   root
   root@fc5b620c5a88:/# pwd
   /
   root@fc5b620c5a88:/# cat /etc/os-release
   PRETTY_NAME="Debian GNU/Linux 11 (bullseye)"
   NAME="Debian GNU/Linux"
   VERSION_ID="11"
   VERSION="11 (bullseye)"
   VERSION_CODENAME=bullseye
   ID=debian
   HOME_URL="https://www.debian.org/"
   SUPPORT_URL="https://www.debian.org/support"
   BUG_REPORT_URL="https://bugs.debian.org/"

Now you are the ``root`` user on a different operating system inside a running
Linux container! You can type ``exit`` to escape the container.

EXERCISE
~~~~~~~~

Before you exit the container, try running the command ``python``. What happens?
Compare that with running the command ``python`` directly on your student VM. 


Run a Command Inside a Container
--------------------------------

Back out on you student VM, we now know we have a container image called
``python`` that has a particular version of Python (3.11.2) that is 
otherwise not available on your student server. The 3.11.2 Python interpreter,  
it's standard library, and all of the dependencies of those are included in the 
container image and 
are *isolated* from everything else. This image (``python``) is portable
and will run the exact same way on any OS that Docker supports.

In practice, though, we don't want to start interactive shells each time we need
to use a software application inside an image. Docker allows you to spin up an
*ad hoc* container to run applications from outside. For example, try:


.. code-block:: console

   [user-vm]$ docker run --rm python whoami
   root
   [user-vm]$ docker run --rm python pwd
   /
   [user-vm]$ docker run --rm python cat /etc/os-release
   PRETTY_NAME="Debian GNU/Linux 11 (bullseye)"
   NAME="Debian GNU/Linux"
   VERSION_ID="11"
   VERSION="11 (bullseye)"
   VERSION_CODENAME=bullseye
   ID=debian
   HOME_URL="https://www.debian.org/"
   SUPPORT_URL="https://www.debian.org/support"
   BUG_REPORT_URL="https://bugs.debian.org/"
   [user-vm] docker run -it --rm python
   Python 3.11.2 (main, Feb 11 2023, 02:24:27) [GCC 10.2.1 20210110] on linux
   Type "help", "copyright", "credits" or "license" for more information.
   >>> # type exit() to escape


In the first three commands above omitted the ``-it`` flags because they did not
require an interactive terminal to run. On each of these commands, Docker finds
the image the command refers to, spins up a new container based on that image,
executes the given command inside, prints the result, and exits and removes the
container.

The last command, which did not specify a command to run inside the container, uses the container's 
default command. We don't know ahead of time what (if any) default command is provided for 
any given image, but what default command was provided for the ``python`` image? 

Yes, it was the ``python`` command itself, and that requires an interactivity to use, 
so we provide the ``-it`` flags.


Essential Docker Command Summary
--------------------------------

+----------------+------------------------------------------------+
| Command        | Usage                                          |
+================+================================================+
| docker login   | Authenticate to Docker Hub using username and  |
|                | password                                       |
+----------------+------------------------------------------------+
| docker images  | List images on the local machine               |
+----------------+------------------------------------------------+
| docker ps      | List containers on the local machine           |
+----------------+------------------------------------------------+
| docker pull    | Download an image from Docker Hub              |
+----------------+------------------------------------------------+
| docker run     | Run an instance of an image (a container)      |
+----------------+------------------------------------------------+
| docker inspect | Provide detailed information on Docker objects |
+----------------+------------------------------------------------+
| docker rmi     | Delete an image                                |
+----------------+------------------------------------------------+
| docker rm      | Delete a container                             |
+----------------+------------------------------------------------+
| docker stop    | Stop a container                               |
+----------------+------------------------------------------------+
| docker build   | Build a docker image from a Dockerfile in the  |
|                | current working directory                      |
+----------------+------------------------------------------------+
| docker tag     | Add a new tag to an image                      |
+----------------+------------------------------------------------+
| docker push    | Upload an image to Docker Hub                  |
+----------------+------------------------------------------------+

If all else fails, display the help text:

.. code-block:: console

   [isp02]$ docker --help
   shows all docker options and summaries


.. code-block:: console

   [isp02]$ docker COMMAND --help
   shows options and summaries for a particular command

Additional Resources
--------------------

* `Docker Docs <https://docs.docker.com/>`_
* `Best practices for writing Dockerfiles <https://docs.docker.com/develop/develop-images/dockerfile_best-practices/>`_
* `Docker Hub <https://hub.docker.com/>`_
* `Docker for Beginners <https://training.play-with-docker.com/beginner-linux/>`_
* `Play with Docker <https://labs.play-with-docker.com/>`_
