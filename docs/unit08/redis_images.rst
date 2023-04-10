Storing and Retrieving Images in Redis
======================================

Storing Image Function
----------------------

So here is a bit of code that allows you to store an image into Redis. It
doesn't actually store the *image*, it stores the *bytes* and a key value

.. code-block:: python3
   :linenos:
   
   import redis

   rd=redis.Redis(host='127.0.0.1', port=6379, db=1)

   #write image to redis

   # read the raw file bytes into a python object
   file_bytes = open('./my_sine.png', 'rb').read()

   # set the file bytes as a value in Redis
   rd.set('key', file_bytes)


Retrieving Image Function
-------------------------

And here is a bit of code that allows you to *retrieve* and image from Redis.
Once again, we're not actually retrieving the image, we're retrieving the bytes,
and then writing the bytes out to a file

.. code-block:: python3
    :linenos:
   
    import redis

    rd=redis.Redis(host='127.0.0.1', port=6379, db=1)

    #define and open the file path for writing binary
    path = './myimage.png'
    with open(path, 'wb') as f:
      #get the image based on the 'key' and write that binary stream to the
      #file path
      
      f.write(rd.get('key'))
      

There is a couple tweaks you have to implement to retrieve the file from Flask.
Remember, your user will not be executing a Python script, but will be hitting a
route in your Flask app.

Your Flask app will be *returning* the file, and you will pipe, >>, the file to
someplace local. For Flask, you would be adding this line of code to your *retrieve*
*image* function:

.. code-block:: python3
   :linenos:
   
   return send_file(path, mimetype='image/png', as_attachment=True)   


Hint: There are public imaging platforms available with open APIs that can manage
getting the file to the user. Imagine this scenario, your file gets uploaded to *imgur*
using *imgur*'s public API. Your Flask app would then return the *url* to the image on *imgur*


EXERCISE
~~~~~~~~

Last time you created a path that creates and saves a plot to Redis. For this exercise, take your
revisit yoru flask app and create a new route called '/getplot'. Which will retrieve your image
from Redis and copy it to your VM

