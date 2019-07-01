Quick Start using Docker containers
===================================

You can try out OpenRedukti using Docker container technology.

We assume you have have Docker installed locally.

Also if you want to save Jupyter Notebooks then you may need to use Linux env, as on Windows the file system binding may not work.

Starting OpenRedukti server
---------------------------

::

    docker pull redukti/openredukti:latest
    docker run --rm -it -p 9001:9001/tcp redukti/openredukti:latest

Starting Python3 Jupyter with PyRedukti support
-----------------------------------------------

::

    docker pull redukti/pyredukti:latest
    docker run --rm -it -p 8888:8888/tcp -v"$PWD":/data:z redukti/pyredukti:latest