An Introduction to Python Bindings for OpenRedukti
==================================================

PyRedukti is a Python binding for :doc:`openredukti-intro`.

OpenRedukti is a C++ library that has following features:

* Ability to express an interest rate product as a set of cashflows
* Bootstrap continuously compounded zero coupon interest rate curves using Linear, CubicSpline, and MonotoneConvex interpolators
* Interpolate curves in the discount factor space using LogLinear and LogCubicSpline interpolators
* Compute present value of cashflows
* Compute first and second order derivatives using `Automatic/algorithmic Differentiation <http://www.autodiff.org/>`_.

These features are available as a library, but additionally OpenRedukti provides a server implementation
based upon `gRPC <https://grpc.io/>`_ technology.

The Python bindings operate at two levels.

At the low level, the Python bindings provide wrappers for a number of OpenRedukti classes such as:

* ``Date`` - Date value
* ``ADVar`` - Automatically differentiated variable
* ``Calendar`` - Business Day Calendars
* ``DayFraction`` - Day Count Fraction calculators
* ``InterestRateIndex`` - Various functions related to indices
* ``Interpolators`` - Various 2-d interpolation schemes
* ``YieldCurve`` - Includes two subtypes: ``InterpolatedYieldCurve``, and ``SvenssonCurve``
* ``ScheduleGenerator`` - generates cashflow schedules

Associated with these are a bunch of types and enum values that are generated from Google Protocol Buffer descriptions.

At a higher level, PyRedukti offers the ability to communicate with the OpenRedukti Valuation and Curve Building services.
Using this model, you get following additional capabilities:

* Build Zero Curves from market quotes (PAR rates)
* Prime the Valuation Service with market data such as Zero Curves and Fixings
* Get NPV and Zero/PAR Sensitivities calculated for trades that can be represented as cashflows 

The following classes and utilities provide the core functions:

* ``GRPCAdapter`` - This adapter is used to communicate with the OpenRedukti server using gRPC Protocol
* ``LocalAdapter`` - This adapter communicates with the ``InMemoryRequestProcessor``
* ``InMemoryRequestProcessor`` - This is the OpenRedukti Request Processor embedded locally within Python, useful for ad-hoc experiments
* ``MarketData`` - This class encapsulates reading raw market data from CSV formatted files
* ``ServerCommand`` - This class encapsulates the functions for interacting with the OpenRedukti server 

The OpenRedukti Computation Model
---------------------------------
OpenRedukti's design is quite different from QuantLib. In QuantLib various objects are interlinked via C++ pointers; this approach is 
unsuitable for a server deployment.

OpenRedukti's model decouples the raw market data, its conversion to curves, and the use of curves for valuation purposes.

So in order to value trades, you have to do following:

1. Build curves using raw market data (i.e. quotes). Curve Building uses a multi-curve builder so this is an expensive step. During curve
   building you can optionally generate PAR sensitivities, which is a very expensive step (can take minutes).
2. You then seed a Valuation Service with the curves, fixings and some additional configuration data such as curve mappings. 
   OpenRedukti groups curves into market date sets. Each market data set is identified by the business date, the market data qualifier,
   and a cycle number. This allows multiple market data sets to be computed and registered at the same time. 
   The Valuation Service caches all the registered market data sets in memory.
3. Each curve in the market data set is identified by a scenario number. This should be 0 for the base curves.
   However, you may add additional curves with incremented scenario numbers. Such curves are used for VaR computation in 
   OpenRedukti, and each scenario curve represents a VaR scenario, i.e. the curve is shifted from the base curve in some manner.
   Note that scenario curves never compute sensitivities.
4. Once the required market data set is registered, you submit all trades for valuation to the Valuation Service. 
   Each valuation request must be for a specific market data set, identified by the PricingContext. When you submit a valuation
   request you can specify the scenario range to compute, for instance if you have base scenario 0 and 1250 scenario curves,
   then you can request the range 0-1250. The Valuation Service will compute NPV for each of the scenarios and return the result.
   Sensitivities if requested will only be generated for the base curves.
5. If you need to update the curves for a particular market data set, you have to do the steps above again, and then revalue all your trades
   for that set. Typically though you would create a new market data set rather an update one already created. Your trades have no links to the
   market data set, trades live in your world, OpenRedukti doesn't care about them.
6. Although the OpenRedukti Valuation Service caches market data, it is a stateless service, there is no persistent state. 
   Each request is independent, and if you recreate the InMemoryRequestProcessor instance, or restart the OpenRedukti server,
   all market data is forgotten and must be supplied again.
   If you request valuation for a non-existent market data set then the service will simply tell you it cannot find the data.

Building PyRedukti
------------------

Pre-requisites
++++++++++++++

* Python 3.6 and above
* Following packages are required

  * pip
  * grpcio
  * grpcio-tools
  * cython

Build steps
+++++++++++

* First build OpenRedukti with gRPC support, see :doc:`openredukti-building`.
* We assume that Protobuf is installed under ``$HOME/Software/protobuf``.
* We assume that OpenRedukti is installed under ``$HOME/Software/redukti``.
* We assume that gRPC C++ library is installed under ``$HOME/Software/grpc``.

Build and install Python module
+++++++++++++++++++++++++++++++

* Set ``LD_LIBRARY_PATH`` as follows:

::

    export LD_LIBRARY_PATH=$HOME/Software/protobuf/lib:$HOME/Software/redukti/lib:$HOME/Software/grpc/lib:$LD_LIBRARY_PATH

* Setup Python virtual; environment if necessary
* Execute following steps:

::

    python setup.py bdist_wheel
    pip install dist/pyredukti-0.1-cp36-cp36m-linux_x86_64.whl

Checking Out PyRedukti
----------------------

For example sessions in PyRedukti, please look at the Jupyter Notebook samples in `https://github.com/redukti/PyRedukti/tree/master/notebooks 
<https://github.com/redukti/PyRedukti/tree/master/notebooks>`_.