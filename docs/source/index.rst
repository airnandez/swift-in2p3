.. swift-in2p3 documentation master file, created by
   sphinx-quickstart on Thu Nov 10 14:20:21 2016.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

How to use OpenStack Swift at CC-IN2P3
======================================

.. toctree::
   :maxdepth: 2

------------
Introduction
------------

CC-IN2P3 is operating an experimental installation of the OpenStack object store, also known as `Swift <https://www.openstack.org/software/releases/mitaka/components/swift>`_ . Swift allows you to store objects and retrieve them easily from any connected computer. In this context, an *object* is an opaque sequence of bytes uniquely identified by its name.

This document briefly presents how to get started using Swift.

-------------
Prerequisites
-------------

You need an account specific to the CC-IN2P3's instance of Swift. If you are a user of CC-IN2P3 and would like to have an account, you can apply for one through the user support interface: https://cc-usersupport.in2p3.fr.

For the purposes of this document we will assume that you already have an account and you have been provided the following information:

* authentication URL (e.g. ``https://keystone.example.org:8080``)
* tenant name (e.g. ``lsst``)
* user name (e.g. ``fabio``)
* password (e.g. ``xxxxxxx-xxxx-xxxx-xxxx-xxxxxxx``)

You also need a command line tool to interact with the Swift service. In this document we are using the `official Swift Python client <https://pypi.python.org/pypi/python-swiftclient>`_. You need a working Python installation on your computer. If you don't have one yet, we recommend Continuum Analytics' `Anaconda <https://www.continuum.io/downloads>`_.

Install the Swift client with the command::

	$ pip install python-swiftclient python-keystoneclient

After successfully installing these packages, you will have a ``swift`` command::

	$ swift --version
	python-swiftclient 2.6.0

To get help on how to use this command do::

	$ swift --help

**WARNING**: Please note that if your computer uses macOS and you have Xcode installed, there is already a ``swift`` command installed, which is not the one we are referring to in this document.

-----------------------
Set up your environment
-----------------------

The ``swift`` command is easier to use if you initialize some environmental variables, with the information associated to your account::

	export OS_AUTH_URL=https://keystone.example.org:8080
	export OS_TENANT_NAME=lsst
	export OS_USERNAME=fabio
	export OS_PASSWORD=xxxxxxx-xxxx-xxxx-xxxx-xxxxxxx

-------------------
Some Swift concepts
-------------------

In Swift, you can organize your files in a two-level hierarchy: *containers* and *objects*. An object is uniquely identified by the name of the container it is stored in and the name of the object within the container.

Examples of valid names of a container in Swift are ``documents``,  ``images``, ``my-container``, etc. Examples of valid names of objects stored in a given container are ``myImage.fits``, ``raw/images/r/1235.fits`` or ``papers/higgs.pdf``. Note that you can use the character slash (``/``) as part of the object name. However, there is no notion of directory in Swift.

Also note that the name of containers **must be unique for all users under the same tenant**. In other words, if another user in the same tenant as you are already created a container named ``data`` you cannot create another container with the same name. A good practice to avoid name collisions is to use your user name as a prefix of your containers, for instance ``fabio-data``.

A tenant can have a large number of containers. By default, the containers and the objects inside them are accessible by all the users in the same tenant. See below on how to modify who can access your objects.


----------------------------
Using containers and objects
----------------------------

#. To **create** a container with name ``fabio-data`` use the command::

	$ swift post fabio-data

#. To **upload** a file in your computer with path ``/tmp/myFile.fits`` to the container ``fabio-data`` and store it with name ``images/fits/myFile.fits`` use the command::

	$ swift upload --object-name images/fits/myFile.fits fabio-data /tmp/myFile.fits

#. To **list** the contents, i.e. the names of the objects, in container ``fabio-data`` do::

	$ swift list fabio-data
	images/fits/myFile.fits

#. To **download** the object ``images/fits/myFile.fits`` stored in container ``fabio-data`` and save it in your computer with the path ``/tmp/copy.fits`` do::

	$ swift download -o /tmp/copy.fits fabio-data images/fits/myFile.fits

#. To **delete** the object ``images/fits/myFile.fits`` stored in container ``fabio-data`` do::

	$ swift delete fabio-data images/fits/myFile.fits

#. To **delete** the container ``fabio-data`` and all the objects within it do::

	$ swift delete fabio-data

---------------------------------
Share your objects with the world
---------------------------------

You can share objects stored in Swift with other individuals, who may or may not have an account in the same Swift instance.

For sharing a file with an individual without a Swift account, we suggest you create a container where you store all the objects you want to share publicly. We will name this container ``fabio-public`` and set **world-readable** permissions for it::

	$ swift post fabio-public
	$ swift post -r '.r:*' fabio-public

Any object stored in the container ``fabio-public`` can be downloaded by any individual provided they know the URL of the object.

First, upload the object you want to share to the container ``fabio-public``. The identifier of this object in Swift will be ``images/fits/myFile.fits``::

	$ swift upload --object-name images/fits/myFile.fits fabio-public myFile.fits

Now, you need to build the URL of this particular object. You will share this URL with the people you want to share your object with::

$ URL=`swift auth | grep OS_STORAGE_URL | sed 's/=/ /' | awk '{print $3}'`
$ URL=$URL/fabio-public/images/fits/myFile.fits

You can now share the value of the variable ``$URL``. Any person who knows this URL can download your file, for instance using::

	$ curl -o /scratch/myFile.fits $URL


-------
Support
-------

If you are experiencing issues, please contact the author directly: if you are reading this document it is very likely you know how to reach him.

Alternatively, open an issue in github but please don't add your credentials in your report as github issues are public.

-------
Credits
-------

This document was written and is maintained by Fabio Hernandez at `IN2P3 / CNRS computing center <http://cc.in2p3.fr/>`_ (Lyon, France).


-------
License
-------

Copyright 2016 Fabio Hernandez

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.
