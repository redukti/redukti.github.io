===========================
Introduction to OpenRedukti
===========================

OpenRedukti is a C++ library for working with Interest Rate Derivative products such as Interest Rate Swaps, and
FRAs. It allows you to build Interest Rate curves with different interpolation methods, and then use these curves
to compute present value and sensitivities of Interest Rate Derivatives.

OpenRedukti is Free Software, licensed under the GNU General Public License, v3. If you wish to use OpenRedukti
under a non-GPL license, you can raise an issue on `GitHub repository <https://github.com/redukti/OpenRedukti>`_. 
A liberal license will be granted to your company at zero cost, provided you agree to allow your company
to be listed as a user of OpenRedukti.

Main Features
=============
* Small library with minimal external dependencies (only external dependencies are BLAS, LAPACK and Google Protocol Buffers) 
* Ability to express an interest rate product as a set of cashflows
* Bootstrap continuously compounded zero coupon interest rate curves using Linear, CubicSpline, and MonotoneConvex interpolators
* Interpolate curves in the discount factor space using LogLinear and LogCubicSpline interpolators
* Compute present value of cashflows
* Compute first and second order derivatives using `Automatic/algorithmic Differentiation <http://www.autodiff.org/>`_.
* Script using `Ravi <https://github.com/dibyendumajumdar/ravi>`_ - a derivative of `Lua <http://www.lua.org>`_ programming language
* Script using Python, see :doc:`pyredukti-intro`.
* Docker images now available to make it easier to try out OpenRedukti and PyRedukti: see :doc:`openredukti-docker-quickstart`.

Background
==========
OpenRedukti is part of the `MyCCP <http://redukti.com/>`_ product that was being developed by REDUKTI LIMITED. The development of MyCPP
was halted in June 2017 due to lack of funding. A decision was taken then to Open Source parts of the MyCCP product, leading to
the release of OpenRedukti.

The main differences between the Open Source release and the proprietary version used in MyCCP are:

* Only the core C++ pricing library has been released
* The functionality for generating cashflows from FpML trades has not been released as this is fine tuned for the needs of MyCCP
* The Limit Checker and VaR Calculator have not been released
* The MyCCP front-end and middle tier components, written in C#, have not been released as these are very specific to requirements of a CCP.

For further details of the full scope of the MyCCP product, please visit `Redukti.Com <http://redukti.com/myccp-product-specifications.html>`_. 

The OpenRedukti Computation Model
=================================
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


Ackowledgements
===============
OpenRedukti gratefully acknowledges ideas and code it is using from other projects.

* My good friend `Christer Rydberg <https://www.linkedin.com/in/christer-rydberg-phd-98012a7/>`_ for showing me how to implement Automatic Differentiation and for help and advice over the years. 
* `Jeffrey Fike's work <http://adl.stanford.edu/hyperdual/>`_ on automatic differentiation using hyperdual vectors.
* `QuantLib <http://quantlib.org/index.shtml>`_ which has provided the basis for some of the key components such as interpolators.
* The monotone convex interpolation method is based on papers and VB code from `Financial Modeling Agency <http://finmod.co.za/#our-research>`_. 
* The `C/C++ Minpack <http://devernay.free.fr/hacks/cminpack/>`_ library provides the Levenberg-Marquardt solver used in curve building.