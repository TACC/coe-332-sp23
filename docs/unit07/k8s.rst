Kubernetes - Overview and Introduction to Pods
==============================================

In this section we give an overview of the Kubernetes system and introduce the first major Kubernetes 
abstraction, the Pod.

After going through this module, the student should be able to:

* Connect to the class Kubernetes cluster and issue basic commands using ``kubectl``.
* Describe a Kubernetes pod in a yaml file and use ``kubectl`` to deploy the pod to the cluster.
* Retrieve details about the pod including status and logs. 
* Use labels to organize pods deployed to the Kubernetes cluster.
* **Design Principles**. Kubernetes improves our software portability, particularly 
  for large distributed applications that need to run across multiple machines.


Kubernetes Overview
~~~~~~~~~~~~~~~~~~~
Kubernetes (k8s) is itself a distributed system of software components that run a cluster of one or more machines (physical
computers or virtual machines). Each machine in a k8s cluster is either a "manager" or a "worker" node.

Users communicate with k8s by making requests to its API. The following steps outline how Kubernetes works at a high level:

 1) Requests to k8s API describe the user's *desired state* on the cluster; for example, the desire that 3 containers of
    a certain image are running.
 2) The k8s API schedules new containers to run on one or more worker nodes.
 3) After the containers are started, the Kubernetes deployment controller, installed on each worker node, monitors the
    containers on the node.
 4) The k8s components, including the API and the deployment controllers, maintain both the *desired state* and the
    *actual state* in a distributed database. The components continuously coordinate together to make the actual state
    converge to the desired state.

.. figure:: ./images/k8s_overview.png
    :width: 1000px
    :align: center


.. note::

  It is important to note that most of the time, the k8s API as well as the worker nodes are running on separate machines
  from the machine we use to interact with k8s (i.e., make API requests to it). The machine we use to interact with k8s
  only needs to have the k8s client tools installed, and in fact, as the k8s API is available over HTTP, we don't strictly
  speaking require the tools -- we could use curl or some other http client -- but the tools make 
  interacting with the API much easier.


Connecting to the TACC Kubernetes Instance
------------------------------------------
In this class, we will use a Kubernetes cluster running at TACC that we have created for use in this class 
for deploying our applications. To simplify the process
of accessing the Kubernetes cluster, we have enabled connectivity to it from the ``login-coe332`` host. 
Therefore, any time you want to work with k8s, simply SSH to ``login-coe332`` with your TACC username as you have throughout the 
semester and then issue the command ``ssh kube-access``:

.. code-block:: console 

 [laptop] $ ssh <tacc_username>@login-coe332.tacc.utexas.edu
 [coe332] $ ssh kube-access
 [kube]   $ # ...do kubernetes work...

Note that the result of issuing the command ``ssh kube-access`` is that you are now in a new shell on a 
separate machine. This machine has Kubernetes tools installed on it and has *access to the Kubernetes API*
for the cluster, but you should be aware that this server is *not part of the Kubernetes cluster* itself. 
For that reason, I will refer to it as the "Kubernetes control node" and
I will use the ``[kube]`` label before the prompt to indicate commands that should be issued from the 
Kubernetes control node. 

.. note:: 

    The COE 332 kubernetes cluster is not available on the public internet for security reasons.
    You must first SSH to login-coe332 before ssh'ing to the Kubernetes cluster. 

Let's do an in-class diagram to understand what is going on. 

First Commands with k8s
-----------------------

We will use the Kubernetes Command Line Interface (CLI) referred to as "kubectl" (pronounced "Kube control") to make
requests to the Kubernetes API. We could use any HTTP client, including a command-line client such as curl, but ``kubectl``
simplifies the process of formatting requests.

The ``kubectl`` software should already be installed and configured to use the Freetail K8s cluster. Let's verify that
is the case by running the following:

.. code-block:: console

  [kube]$ kubectl version -o yaml

You should see output similar to the following:

.. code-block:: console

  clientVersion:
    buildDate: "2022-05-03T13:46:05Z"
    compiler: gc
    gitCommit: 4ce5a8954017644c5420bae81d72b09b735c21f0
    gitTreeState: clean
    gitVersion: v1.24.0
    goVersion: go1.18.1
    major: "1"
    minor: "24"
    platform: linux/amd64
  kustomizeVersion: v4.5.4
  serverVersion:
    buildDate: "2022-08-17T18:47:37Z"
    compiler: gc
    gitCommit: 95ee5ab382d64cfe6c28967f36b53970b8374491
    gitTreeState: clean
    gitVersion: v1.24.4
    goVersion: go1.18.5
    major: "1"
    minor: "24"
    platform: linux/amd64



This command made an API request to the TACC k8s cluster and returned information about the version
of k8s running there (under ``serverVersion``) as well as the version of the ``kubectl`` that we are running (under
``clientVersion``).

.. note::

  The output of the ``kubectl`` command was yaml because we used the ``-o yaml`` flag. We could have asked for the output
  to be formatted in json with ``-o json``. The ``-o`` flag is widely available on ``kubectl`` commands.


Authentication and Namespaces in Kubernetes
-------------------------------------------
Before we can do any real work on the Kubernetes cluster, we need to understand the concept of a *namespace*.
In Kubernetes, a *namespace* is a logical partition of objects defined on the cluster, and different users 
can have different levels of access (including no access at all) to different namespaces. In this way, 
Kubernetes supports launching different applications -- even different
applications owned by different users or organizations -- on the same physical cluster 
in an isolated way from each other. Each different user or organization would be assigned a different
namespace where their k8s objects would live, and users wouldn't have access to any other namespace. 

That is how the class Kubernetes cluster has been set up. Each of you has been assigned your own namespace
in the Kubernetes cluster where you have administrative access. Inside that namespace, you can create
and manage the Kubernetes objects for your application. And while all of the Kubernetes objects for every 
student is running on the same cluster, you won't see or have access to the objects in different namespaces.

We haven't introduced `pods` yet -- we will shortly -- but let's try a simple experiment: issue 
the following command 

.. code-block:: console 

    [kube]$ kubectl get pods

You will get an error like the following:

.. code-block:: console

  Error from server (Forbidden): pods is forbidden: User "jstubbs" cannot list resource "pods" in API group "" in the namespace "default"


Your ``kubectl`` client is not configured to make requests in your private namespace, so it is using the
default namespace (called "default"), and you do not have access to that namespace.  
Therefore, the first order of business is to configure your kubectl client to use the namespace that is 
private to you -- these namespaces have already been created on the cluster for COE 332. 

There are two ways to configure the namespace. First, we can simply specify the namespace directly on the 
command line using the ``--namespace`` argument. Let's try that now:

.. code-block:: console

    [kube]$ kubectl get pods --namespace=[your tacc username] 

Be sure to change ``[your tacc username]`` to your actual username. For example, I would run:

.. code-block:: console

    [kube]$ kubectl get pods --namespace=jstubbs
    No resources found in jstubbs namespace.

This is the output we expect because we haven't created anything in our namespace yet. 

It's a lot of typing to put ``--namespace=[tacc_username]`` on every command, so what we will do is 
configure our kubectl client to always use that namespace by default. We do that by changing the 
client's configuration file. The kubectl configuration resides in 
the file ``~/.kube/config``. Open that file for editing and add a new line containing the text 
``namespace: [tacc_username]`` within the ``contexts.context`` stanza. This new line should appear 
right below an existing line with the text ``user: [tacc_username]`` about halfway down the file.

For example, here is how mine will look:

.. code-block:: console
   :linenos:
   :emphasize-lines: 10
  
   apiVersion: v1
   clusters:
   - cluster:
   . . .
 
   contexts:
   - context:
       cluster: cluster.local
       user: jstubbs
       namespace: jstubbs
     name: jstubbs@cluster.local
   . . . 


.. warning::

    Remember, in yaml, spacing matters (just like in Python)! Be sure to indent the new line to the same
    amount using spaces and not tabs. 


With the changes made to the file saved, we should now be able to run our get pods command without
specifying the namespace:

.. code-block:: console 

    [kube]$ kubectl get pods
    No resources found in jstubbs namespace.

Introduction to Pods
~~~~~~~~~~~~~~~~~~~~

Pods are a fundamental abstraction within Kubernetes and are the most basic unit of computing that can be deployed onto
the cluster. A pod can be thought of as generalizing the notion of a container: a pod contains one or more containers
that are tightly coupled and need to be scheduled together, on the same computer, with access to a shared file system
and a shared network address.

.. note::

  By far, the majority pods you will meet in the wild, including the ones used in this course, will only include one
  container. A pod with multiple containers can be thought of as an "advanced" use case.


Hello, Kubernetes
~~~~~~~~~~~~~~~~~

To begin, we will define a pod with one container. As we will do with all the resources we want to create in k8s, we
will describe our pod in a yaml file.

Create a file called ``pod-basic.yml``, open it up in an editor and paste the following code in:

.. code-block:: yaml

    ---
    apiVersion: v1
    kind: Pod
    metadata:
      name: hello
    spec:
      containers:
        - name: hello
          image: ubuntu:18.04
          command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']

Let's break this down. The top four attributes are common to all k8s resource descriptions:

  * ``apiVersion`` -- describes what version of the k8s API we are working in. We are using ``v1``.
  * ``kind`` -- tells k8s what kind of resource we are describing, in this case a ``Pod``.
  * ``metadata`` -- in general, this is additional information about the resource we are describing that doesn't pertain
    to its operation. Here, we are giving our pod a ``name``, ``hello``.
  * ``spec`` -- This is where the actual description of the resource begins. The contents of this stanza vary depending
    on the ``kind`` of resource you are creating. We go into more details on this in the next section.


.. warning::

  Only one Kubernetes object of a specific ``kind`` can have a given ``name`` at a time. If you define a second pod
  with the same name you will overwrite the first pod. This is true of all the different types of k8s objects we will
  be creating.


The Pod Spec
~~~~~~~~~~~~

In k8s, you describe resources you want to create or update using a ``spec``. The required and optional parameters
available depend on the ``kind`` of resource you are describing.

The pod spec we defined looked like this:

.. code-block:: yaml

    spec:
      containers:
        - name: hello
          image: ubuntu:18.04
          command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']

There is just one stanza, the ``containers`` stanza, which is a list of containers (recall that pods can contain
multiple containers). Here we are defining just one container. For each container, we provide:

  * ``name`` (optional) -- this is the name of the container, similar to the name attribute in Docker.
  * ``image`` (required) -- the image we want to use for the container, just like with Docker.
  * ``command`` (optional) -- the command we want to run in the container. Here we are running a short BASH script.


Creating the Pod In K8s
~~~~~~~~~~~~~~~~~~~~~~~

We are now ready to create our pod in k8s. To do so, we use the ``kubectl apply`` command. In general, when you have
a description of a resource that you want to create or update in k8s, the ``kubectl apply`` command can be used.

In this case, our description is contained in a file, so we use the ``-f`` flag. Try this now:

.. code-block:: console

  [kube]$ kubectl apply -f pod-basic.yml

If all went well and k8s accepted your request, you should see an output like this:

.. code-block:: console

  pod/hello created


.. note:: 

  The message ``pod/hello created`` indicates that the description of the pod was valid, that
  k8s has saved the pod definition in its database and that it is working on starting the pod on the
  cluster. It does **not** mean the pod is already created/running on the cluster. 

In practice, we won't be creating many ``Pod`` resources directly -- we'll be creating other resources, such as
``deployments`` that are made up of pods -- but it is important to understand pods and to be able to work
with pods using ``kubectl`` for debugging and other management tasks.


.. note::

  The pod we just created is running on the k8s cluster, NOT on your student VM and NOT on kube controller
  node. You will not be able to find it using commands like docker ps, etc.

During the lecture, we'll draw a picture here to help explain what is going on.



Working With Pods
~~~~~~~~~~~~~~~~~

We can use additional ``kubectl`` commands to get information about the pods we run on k8s.

Listing Pods
-------------
For example, we can list the pods on the cluster with ``kubectl get <object_type>`` -- in this case, the object type
is "pods":

.. code-block:: console

  [kube]$ kubectl get pods

    NAME    READY   STATUS    RESTARTS   AGE
    hello   1/1     Running   0          64s

The output is fairly self-explanatory. We see a line for every pod which includes its name, status, the number of times
it has been restarted and its age. Our ``hello`` pod is listed above, with an age of ``64s`` because we just started it
but it is already RUNNING. Additional pods may be listed in my output in class due to prior work sessions.


Getting and Describing Pods
---------------------------

We can pass the pod name to the ``get`` command -- i.e., ``kubectl get pods <pod_name>`` -- to just get information on
a single pod

.. code-block:: console

  [kube]$ kubectl get pods hello
    NAME    READY   STATUS    RESTARTS   AGE
    hello   1/1     Running   0          3m1s

The ``-o wide`` flag can be used to get more information:

.. code-block:: console

  [kube]$ kubectl get pods hello -o wide
    NAME    READY   STATUS    RESTARTS   AGE     IP              NODE            NOMINATED NODE   READINESS GATES
    hello   1/1     Running   0          2m56s   10.233.85.195   kube-worker-2   <none>           <none>

Finally, the ``kubectl describe <resource_type> <resource_name>`` command gives additional information, including the
k8s events at the bottom. While we won't go into the details now, this information can be helpful when troubleshooting
a pod that has failed:

.. code-block:: console

  [kube]$ kubectl describe pods hello 
    Name:         hello
    Namespace:    jstubbs
    Priority:     0
    Node:         kube-worker-2/129.114.39.176
    Start Time:   Sat, 18 Mar 2023 18:07:40 +0000
    Labels:       <none>
    Annotations:  cni.projectcalico.org/containerID: c0903adb8d0df25f2682ad5257a8c9408187298988d915d9aa6cd0d63c7436f2
                cni.projectcalico.org/podIP: 10.233.85.195/32
                cni.projectcalico.org/podIPs: 10.233.85.195/32
    Status:       Running
    IP:           10.233.85.195
    IPs:
    IP:  10.233.85.195
    Containers:
    hello:
        Container ID:  containerd://8d0b93802178e4f4070216158833634f61ba14931f1e63ddd37f165dcb1999e0
        Image:         ubuntu:18.04
        Image ID:      docker.io/library/ubuntu@sha256:8aa9c2798215f99544d1ce7439ea9c3a6dfd82de607da1cec3a8a2fae005931b
        Port:          <none>
        Host Port:     <none>
        Command:
        sh
        -c
        echo "Hello, Kubernetes!" && sleep 3600
        State:          Running
        Started:      Sat, 18 Mar 2023 18:07:44 +0000
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-sfzhr (ro)
    Conditions:
    Type              Status
    Initialized       True 
    Ready             True 
    ContainersReady   True 
    PodScheduled      True 
    Volumes:
    kube-api-access-sfzhr:
        Type:                    Projected (a volume that contains injected data from multiple sources)
        TokenExpirationSeconds:  3607
        ConfigMapName:           kube-root-ca.crt
        ConfigMapOptional:       <nil>
        DownwardAPI:             true
    QoS Class:                   BestEffort
    Node-Selectors:              <none>
    Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
    Events:
    Type    Reason     Age    From               Message
    ----    ------     ----   ----               -------
    Normal  Scheduled  3m27s  default-scheduler  Successfully assigned jstubbs/hello to kube-worker-2
    Normal  Pulling    3m27s  kubelet            Pulling image "ubuntu:18.04"
    Normal  Pulled     3m25s  kubelet            Successfully pulled image "ubuntu:18.04" in 2.81517356s
    Normal  Created    3m25s  kubelet            Created container hello
    Normal  Started    3m24s  kubelet            Started container hello


Getting Pod Logs
----------------

Finally, we can use ``kubectl logs <pod_name>`` command to get the logs associated with a pod:

.. code-block:: console

  [kube]$ kubectl logs hello
    Hello, Kubernetes!

Note that the ``logs`` command does not include the resource name ("pods") because it only can be applied to pods. The
``logs`` command in k8s is equivalent to that in Docker; it returns the standard output (stdout) of the container.


Using Labels
------------

In the pod above we used the ``metadata`` stanza to give our pod a name. We can use ``labels`` to add additional metadata
to a pod. A label in k8s is nothing more than a ``name: value`` pair that we create to organize objects in a 
meaningful way. We can choose any value for ``name`` and ``value`` that we wish but they must be strings. If you
want to use a number like "10" for a label name or value, be sure to enclose it in quotes (i.e., ``"10"``).

You can think of these ``name:value``
pairs as variables and values. So for example, you might create a label called ``shape`` with values 
``circle``, ``triangle``, ``square``, etc. A more realistic label might be ``component_type`` with 
values  ``api``, ``database``, ``worker``, etc. 
Multiple pods can have the same ``name:value`` label.

Let's use the pod definition above to create a new pod with a label.

Create a file called ``pod-label.yml``, open it up in an editor and paste the following code in:

.. code-block:: yaml

    ---
    apiVersion: v1
    kind: Pod
    metadata:
      name: hello-label
      labels:
        version: "1.0"
    spec:
      containers:
        - name: hello
          image: ubuntu:18.04
          command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']

Let's create this pod using ``kubectl apply``:

.. code-block:: console

  [kube]$ kubectl apply -f pod-label.yml
  pod/hello-label created

Now when we list our pods, we should see it

.. code-block::

  [kube]$ kubectl get pods
    NAME          READY   STATUS    RESTARTS   AGE
    hello         1/1     Running   0          22m
    hello-label   1/1     Running   0          22s


Filtering By Labels With Selectors
----------------------------------

Labels are useful because we can use ``selectors`` to filter our results for a given label name and value. To specify
a label name and value, use the following syntax: ``--selector "<label_name>=<label_value>"``.

For instance, we can search for pods with the version 1.0 label like so:

.. code-block:: console

  [kube]$ kubectl get pods  --selector "version=1.0"
    NAME          READY   STATUS    RESTARTS   AGE
    hello-label   1/1     Running   0          4m58s

We can also just use the label name to filter with the syntax ``--selector "<label_name>"``. This will find any pods with
the label ``<label_name>``, regardless of the value.



Additional Resources
~~~~~~~~~~~~~~~~~~~~

 * `k8s Pod Reference <https://kubernetes.io/docs/concepts/workloads/pods/>`_
