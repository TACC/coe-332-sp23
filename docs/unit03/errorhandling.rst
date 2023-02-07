Error Handling
==============

Why Do We Need Error Handling?
------------------------------

#. Prevents program from crashing if an error occurs
    * If an error occurs in a program, we don’t want the program to unexpectedly crash on the user. Instead, error handling can be used to notify the user of why the error occurred and gracefully exit the process that caused the error.

#. Saves time debugging errors
    * Following reason #1, having the program display an error instead of immediately crashing will save a lot of time when debugging errors.
    * The logic inside the error handler can be updated to display useful information for the developer, such as the code trackback, type of error, etc.

#. Helps define requirements for the program
    * If the program crashes due to bad input, the error handler could notify the user of why the error occurred and define the requirements and constraints of the program.


Error-Handling Practices
------------------------

In this section we’ll discuss the following:

* Exception handling using try - except and try - except - finally
* Assertions
* When to use exceptions vs. assertions

Future You will thank Past You if you apply these error-handling techniques.

Catching and Handling Exceptions
--------------------------------

If you have a block of code that might fail, you can manage any exceptions by placing this code in a ``try: … except: … block:``


The error handling is executed as follows:

The statement inside the try block is executed.
If the statement is successful, both except clauses are skipped and the code inside the finally clause is run.

If the statement inside the try block fails, the code in the first except statement is executed. If the statement fails due to a ValueError (i.e. not being able to convert a non-digit to an int), the code in the except ValueError block is run.

If the statement inside the try block fails and the error is not a ValueError, the second except statement is checked. If the statement fails due to a ZeroDivisionError (i.e. integer is being divided by zero), the code inside the except ZeroDivisionError block is run.

The finally clause will always execute after the last task completes — regardless of whether the last task is in the try block or except block.

Things to look out for when handling exceptions
Don’t let the code swallow the exception. We don’t want errors to go undetected by simply ignoring them. If you need to swallow an exception to avoid a fundamental issue, the architecture of the program needs to be re-evaluated.

.. code-block:: python3

    >>> try:
    >>>    y = 100 / x
    >>> except ZeroDivisionError:
    >>>    pass 


NOTE: Don’t declare new variables inside a ```try``` statement that might not be reached.

Raising Exceptions
------------------

You might want to re-raise an exception to abort a script. For example, if we can’t determine what kind of error is causing the exception, we might want to re-raise it.

Problem – Reraising the exception, that has been caught in the except block:

Using the ```raise```
.. code-block:: python3

    >>> def example():
    >>> try:
    >>>    int('N/A')
    >>> except ValueError:
    >>>    print("Didn't work")
    >>>    raise
          
    >>> example()

    Didn't work
    Traceback (most recent call last):
        File "", line 1, in 
        File "", line 3, in example
    ValueError: invalid literal for int() with base 10: 'N/A'


This problem typically arises when there is no need to take any action in response to an exception (e.g., logging, cleanup, etc.). A very common use might be in catch-all exception handlers.

.. code-block:: python3

    try:
       ...
    except Exception as e:
        # Process exception information in some way
        ...
        # Propagate the exception
        raise


Problem – To have a program issue warning messages (e.g., about deprecated features or usage problems).

.. code-block:: python3

    import warnings
    def func(x, y, logfile = None, debug = False):
        if logfile is not None:
            warnings.warn('logfile argument deprecated',
                                DeprecationWarning)

The arguments to ```warn()``` are a warning message along with a warning class, which is typically one of the following:
UserWarning, DeprecationWarning, SyntaxWarning, RuntimeWarning, ResourceWarning, or FutureWarning.
The handling of warnings depends on how the interpreter is executed and other configuration.

Output when running Python with the -W all option.

.. code-block:: console

    bash % python3 -W all example.py
    example.py:5: DeprecationWarning: logfile argument is deprecated
        warnings.warn('logfile argument is deprecated', DeprecationWarning)

Normally, warnings just produce output messages on standard error. To turn warnings into exceptions, use the -W error option.

.. code-block:: console

    bash % python3 -W error example.py
    Traceback (most recent call last):
    
        File "example.py", line 10, in 
            func(2, 3, logfile ='log.txt')
        File "example.py", line 5, in func
            warnings.warn('logfile argument is deprecated', DeprecationWarning)
    DeprecationWarning: logfile argument is deprecated
    bash %   


User-Defined Exceptions
-----------------------


There are several types of built-in exception classes that inherit from the same base Exception class. A full list of these built-in classes can be found in the official documentation.

It’s also possible to create a custom exception class that inherits from the base Exception class. A custom class might be needed if the developer wishes to integrate a more sophisticated logging system or further inspect an object.

The ```__init__()``` and ```__str__()``` methods are required when defining an Exception class:

.. code-block:: python3

    # A python program to create user-defined exception
    # class MyError is derived from super class Exception
    class MyError(Exception):
 
        # Constructor or Initializer
        def __init__(self, value):
            self.value = value
 
        # __str__ is to print() the value
        def __str__(self):
            return(repr(self.value))
 
 
    try:
        raise(MyError(3*2))
 
    # Value of Exception is stored in error
    except MyError as error:
    print('A New Exception occurred: ', error.value)


Ouput

.. code-block:: console
    ('A New Exception occurred: ', 6)



NOTE: help(Exception)

Assertions
----------

Assertions evaluate an expression to true or false. If the expression is false, python will raise an AssertionError exception. Assertions can serve as a powerful developer tool when testing your code.

The syntax for assertions is ```assert Expression[, Arguments]:```

.. code-block:: python3

   >>> a = 20
   >>> assert a < 10, "something went wrong"


The code above will throw this error:

.. code-block:: console

   Traceback (most recent call last):
     File "file.py", line 2, in <module>
       assert a < 10,  "something went wrong"
   AssertionError: something went wrong


.. code-block:: python3

    #!/usr/bin/python

    def KelvinToFahrenheit(Temperature):
       assert (Temperature >= 0),"Colder than absolute zero!"
       return ((Temperature-273)*1.8)+32

    print KelvinToFahrenheit(273)
    print int(KelvinToFahrenheit(505.78))
    print KelvinToFahrenheit(-5)


Output

.. code-block:: console

    32.0
    451
    Traceback (most recent call last):
       File "test.py", line 9, in <module>
          print KelvinToFahrenheit(-5)
       File "test.py", line 4, in KelvinToFahrenheit
          assert (Temperature >= 0),"Colder than absolute zero!"
    AssertionError: Colder than absolute zero!



So When Should We Use Assertions vs. Exceptions?
------------------------------------------------

This really comes down to a case-by-case basis, and there’s room for debate here. In my opinion, exceptions should be used when handling external inputs and outputs due to user input, hardware, network, etc. Exceptions should be used when you want to gracefully exit a program, log data, and notify the user of why such an error occurred.

Assertions have a fail-fast approach and should be used to find errors in your code and detect bugs.

If there are assertions in your production code, my advice is to make sure exception handling is set up to catch any AssertionErrors. On the off chance that an assertion fails in production, at least the code will handle the exception safely by ideally exiting the program, logging the issue, and notifying the user.


Pytest
------

Pytest is a testing framework based on python. It is mainly used to write API test cases.

Pytest is a python based testing framework, which is used to write and execute test codes. In the present days of REST services, pytest is mainly used for API testing even though we can use pytest to write simple to complex tests, i.e., we can write codes to test API, database, UI, etc.


Advantages of Pytest
--------------------

The advantages of Pytest are as follows −

* Pytest can run multiple tests in parallel, which reduces the execution time of the test suite.

* Pytest has its own way to detect the test file and test functions automatically, if not mentioned explicitly.

* Pytest allows us to skip a subset of the tests during execution.

* Pytest allows us to run a subset of the entire test suite.

* Pytest is free and open source.

* Because of its simple syntax, pytest is very easy to start with.

.. code-block:: console
    pip install pytest

    pytest -h


Running pytest without mentioning a filename will run all files of format ```test_*.py``` or ```*_test.py``` in the current directory and subdirectories. Pytest automatically identifies those files as test files. We can make pytest run other filenames by explicitly mentioning them.

Pytest requires the ```test``` function names to start with ```test```. Function names which are not of format ```test*``` are not considered as ```test``` functions by pytest. We cannot explicitly make pytest consider any function not starting with test as a test function.

Example

create a file called: ```test_square.py```

.. code-block:: python3

    import math

    def test_sqrt():
       num = 25
       assert math.sqrt(num) == 5

    def testsquare():
       num = 7
       assert 7*7 == 40

    def tesequality():
       assert 10 == 11

run it

.. code-block:: console

    pytest

output

.. code-block:: console

    test_square.py .F
    ============================================== FAILURES 
    ==============================================
    ______________________________________________ testsquare 
    _____________________________________________
       def testsquare():
       num=7
    >  assert 7*7 == 40
    E  assert (7 * 7) == 40
    test_square.py:9: AssertionError
    ================================= 1 failed, 1 passed in 0.06 seconds 
    =================================


Additional Resources
--------------------
* `Python 3 Error Handling <https://docs.python.org/3/tutorial/errors.html>`
* `Python 3 Assertions <https://docs.pytest.org/en/7.1.x/how-to/assert.html>`
* `Python 3 Exception Class <https://docs.python.org/3/library/exceptions.html>`
* `Pytest`



