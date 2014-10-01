superhappyfunshow
=================

A proof of concept demonstrating deployment of OpenStack Services within Docker
containers using Kubernetes.


Getting Started
===============

Kubernetes deployment on bare metal is a complex topic which is beyond the
scope of this project at this time.  The developers still require a test
environment.  As a result, one of the developers has created a Heat based
deployment tool that can be
found [here](https://github.com/larsks/heat-kubernetes).


Build Docker Images 
-------------------

Within the docker directory is a tool called build.  This tool will build
all of the docker images that have been implemented.  Each OpenStack service is
implemented as a separate container that can later be registered with
Kubernetes. These containers are published to the the public docker registry and
are referenced in the kubernetes configuration files in this repo.

```
[sdake@bigiron docker]$ sudo ./build
```

A 20-30 minute build process will begin where containers will be built for
each OpenStack service.  Once finished the docker images can be examined with
the docker CLI.

```
[sdake@bigiron docker]$ sudo docker images
```

A list of the built docker images will be shown.

Note at this time the images do not yet work correctly or operate on their
defined environment variables.  They are essentially placeholders.


Use Kubernetes to Deploy OpenStack
----------------------------------

Keystone and MariaDB are the only pods that are implimented. They operate
just enough to verify that services are running and may have bugs in their configurations.

To get Keystone running start by downloading the pod and service json files for MariaDB
to a running kubernetes cluster.
```
curl https://raw.githubusercontent.com/jlabocki/superhappyfunshow/master/docker/mariadb/mariadb-service.json >
mariadb-service.json
curl https://raw.githubusercontent.com/jlabocki/superhappyfunshow/master/docker/mariadb/mariadb.json > mariadb.json
```

Next launch the MariaDB pod and service files.
```
$ kubecfg -c mariadb.json create pods
ID                  Image(s)                       Host                Labels                Status
----------          ----------                     ----------          ----------            ----------
mariadb             kollaglue/fedora-rdo-mariadb   /                   name=mariadb-master   Waiting

$ kubecfg -c mariadb-service.json create services
ID                  Labels              Selector              Port
----------          ----------          ----------            ----------
mariadbmaster                           name=mariadb-master   3306
```
To verify their creation and see their status use the list command. You are ready to move on when the
pod's status reaches **Running**.
```
$ kubecfg list pods
ID                  Image(s)                       Host                Labels                Status
----------          ----------                     ----------          ----------            ----------
mariadb             kollaglue/fedora-rdo-mariadb   10.0.0.3/           name=mariadb-master   Running
```
If MariaDB doesn't move to running within a few minutes use journalctl on the master and the nodes to
troubleshoot. The first command is for the master and the second is for the nodes.
```
$ journalctl -f -l -xn -u kube-apiserver -u etcd -u kube-scheduler
$ journalctl -f -l -xn -u kubelet.service -u kube-proxy -u docker
```
Once the pod's status reaches running you should verify that you can connect to the database through all the
kube-proxies. You can use telnet to do this. Telnet to 3306 on each node and make sure mysql responds.
```
$ telnet 10.0.0.4 3306
Trying 10.0.0.4...
Connected to 10.0.0.4.
Escape character is '^]'.

5.5.39-MariaDB-wsrep

$ telnet 10.0.0.3 3306
Trying 10.0.0.3...
Connected to 10.0.0.3.
Escape character is '^]'.

5.5.39-MariaDB-wsrep
```
If the connection closes before mysql responds then the proxy is not properly connecting to the database.
This can be seen by using jounalctl and watching for a connection error on the node that you can't connect
to mysql through.
```
$ journalctl -f -l -xn -u kube-proxy
```
If you can conect though one and not the other there's probably a problem with the overlay network. Double
check that you're tunning kernel 3.16+ because vxlan support is required. If you kernel version is good
try restarting openvswitch on both nodes. This has usually fixed the connection issues for me.

If you're able to connect to mysql though both proxies then you're ready to launch glance. Use the pod
and service files to launch the pods and services for keystone.
```
$ kubecfg -c keystone.json create pods
$ kubecfg -c keystone-service-5000.json create services
$ kubecfg -c keystone-service-35357.json create services
``` 
The keystone pod should become status running, if it doesn't you can troubleshoot it the same way that the
database was. Once keystone is running you should be able to use the keystone client to do a token-get
against one of the proxy's ip addresses.

Directories
===========

* docker - contains artifacts for use with docker build to build appropriate images
