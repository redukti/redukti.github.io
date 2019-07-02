Quick Start using Docker containers
===================================

You can try out OpenRedukti using Docker container technology.

We assume you have have Docker installed locally.

Starting OpenRedukti server
---------------------------

::

    docker pull redukti/openredukti:latest
    docker run --rm -it -p 9001:9001/tcp redukti/openredukti:latest

Starting Python3 Jupyter with PyRedukti support
-----------------------------------------------

Note that following maps ``$PWD`` to ``/data`` so that any files you create in ``/data`` should appear under ``$PWD``::

    docker pull redukti/pyredukti:latest
    docker run --rm -it -p 8888:8888/tcp -v"$PWD":/data:z redukti/pyredukti:latest

The output from the container should tell you how to connect to the Jupyter instance using your Browser.

Important
---------

* To allow the Jupyter notebook to connect to OpenRedukti you will probably need to connect the containers to
  a network and setup aliases. I will add instructions soon
* If you want to save Notebooks outside the container then map a volume to ``/data`` as shown above. However this may
  not work on Windows. 
* These Docker images are for testing only. 