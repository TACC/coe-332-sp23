Public Access to Your Deployment
================================

In this section, we show how to make your Flask API available on the public internet. 
This procedure assumes you have already created a Deployment and a Service (of type 
ClusterIP) for your Flask API.

There are two new Kubernetes objects you will be creating: 

1. A second ``Service`` object of type ``NodePort`` which selects your deployment using the 
   deployment label and exposes your Flask API on a public port. 
2. An ``Ingress`` object which specifies a subdomain to make your Flask API available on and 
   maps this domain to the public port created in Step 1. 


Create a NodePort Service
--------------------------
The first step is to create a NodePort Service object pointing at your Flask deployment. 


Copy the following code into a new file called ``flasktest_nodeport_service.yml`` or something 
similar:

.. code-block:: yaml    
   :linenos:
   :emphasize-lines: 5, 9

   ---
   kind: Service
   apiVersion: v1
   metadata:
       name: flasktest-service-nodeport
   spec:
       type: NodePort
       selector:
           app: flasktestapp
       ports:
           - port: 5000
             targetPort: 5000



Update the highlighted lines:

1. The ``name`` of the Service object can be anything you want, so long 
   as it is unique among the Services you have defined in your namespace. In particular, it needs to 
   be a different name from your ClusterIP service defined previously. 

2. The value of ``app`` in the ``selector`` stanza needs to match the ``app`` label in your 
   deployment. This should be exactly the same as what you did in Step 5 of the 
   `HomeWork 7 Lab <services.html#homework-7-deploying-our-flask-api-to-k8s>`_. As mentioned there 
   in Step 5, be sure the selector targets the **label** in your Flask deployment,
   **not the deployment name**.
  
As usual, create the NodePort using ``kubectl``:

.. code-block:: console 

    [kube]$ kubectl apply -f flasktest_nodeport_service.yml

Change the command to reference the file name you used. 

Check that the service was created successfully and determine the port that was created for it:

.. code-block:: console 

    [kube]$ kubectl get service
    NAME                         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
    flasktest-service-nodeport   NodePort    10.233.15.48   <none>        5000:31587/TCP   45s

Here we see that port ``31587`` was created for my service. Your port will be different. 

.. note::

  You will use the port identified above when creating the Ingress object in the next section. 


You can test that the NodePort service is working by using the special domain ``coe-332.tacc.cloud``
to exercise your Flask API from the kube-access VM:

.. code-block:: console

    [kube]$ curl coe332.tacc.cloud:31587/hello-service
    Hello world

Change the port (``31587``) to the port associated with your nodeport service, and the URL path
(``/hello-service``) to a path your Flask API recognizes. 

.. note::

   The curl above only works from the kube-access VM. We will open the Flask API to the public 
   internet in the next section. 


Create an Ingress 
-----------------

Next we will create an Ingress object which will map the NodePort port defined previously 
(in my example, ``31587``) to a 
specific domain on the public internet. 

Copy the following code into a new file called ``flasktest_ingress.yml`` or something 
similar:

.. code-block:: yaml    
   :linenos:
   :emphasize-lines: 5, 11, 20

   ---
   kind: Ingress
   apiVersion: networking.k8s.io/v1
   metadata:
     name: flasktest-ingress
     annotations:
       kubernetes.io/ingress.class: "nginx"
       nginx.ingress.kubernetes.io/ssl-redirect: "false"
   spec:
     rules:
     - host: "jstubbs.coe332.tacc.cloud"
       http:
           paths:
           - pathType: Prefix
             path: "/"
             backend:
               service:
                 name: flasktest-service-nodeport
                 port:
                     number: 31587


Be sure to update the highlighted lines:

1. Specify a meaningful ``name`` for the ingress. Keep in mind it should be unique among all 
   Ingress obejcts within your namespace. 
2. Update the ``host`` value to include your username in the subdomain, i.e., use the format 
   ``- host: "<username>.coe332.tacc.cloud"``.
3. Update port number to match the NodePort port you created in step 1. 

Create the Ingress object:

.. code-block:: console 

    [kube]$ kubectl apply -f flasktest_ingress.yml


Double check that the object was successfully created:

.. code-block:: console

    [kube]$ kubectl get ingress
    NAME                CLASS    HOSTS                       ADDRESS   PORTS   AGE
    flasktest-ingress   <none>   jstubbs.coe332.tacc.cloud             80      102s

At this point our Flask API should be available on the public internet from the domain 
we specified in the ``host`` field. We can test by running the following curl command from 
anywhere, including our laptops. 


.. code-block:: console

    [local]$ curl jstubbs.coe332.tacc.cloud/hello-service
    Hello world

