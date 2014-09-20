superhappyfunshow
=================

A proof of concept for how to deploy the OpenStack Services within Docker
containers using Kubernetes on Project Atomic.


Getting Started
=================

Kubernetes deployment on bare metal is a complex topic which is beyond the
scope of this project at this time.  The developers still require a test
environment.  As a result, we have created a Heat based deployment environment.
Our template requires Neutron to operate.

Please see the kubedeploy directory README.md for further information.

Directories
=================
* docker     - contains artifacts for use with docker build to build appropriate images
* kubedeploy - contains a Heat template to deploy a kubernetes cluster (see README.md)
