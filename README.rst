What is this?
=============

DCOR uses virtualbox images created with `Vagrant <https://www.vagrantup.com/>`_ for testing.
This repository contains the recipe for creating the
`dcor-test <https://app.vagrantup.com/paulmueller/boxes/dcor-test>`_ vagrant boxes.


Why virtualbox/vagrant?
=======================

There is a docker recipe for CKAN and I could probably also have adapted that to DCOR, but
that looked like a lot of code to maintain. The upside of virtualbox images is that you can
take your time to set it up (which is very convenient if you use vagrant) and then just
distribute the final box image for testing.


How does testing work?
======================
Let's use the `ckanext-dcor_schemas<https://github.com/DCOR-dev/ckanext-dcor_schemas>`_ repository as an example.
If you clone into it, you will have the files
`Vagrantfile <https://github.com/DCOR-dev/ckanext-dcor_schemas/blob/master/Vagrantfile>`_
(the recipe for setting up the box) and
`vagrant-run-tests.sh <https://github.com/DCOR-dev/ckanext-dcor_schemas/blob/master/vagrant-run-tests.sh>`_
(that installs the current working tree into the CKAN environment and executes the tests).
When you run ``vagrant up`` in this directory, vagrant uses ``Vagrantfile`` to setup the virtualbox
image (download and rsync the current working directory of the host to `/testing` on the guest).
To run the tests, simply run:

.. code::

    vagrant ssh -- sudo bash /testing/vagrant-run-tests.sh



Basic box creation workflow
===========================

This is only necessary if you want to create an image from scratch (from an existing
ubuntu box). If you only have to do something incrementally, you can just base your
box on the existing dcor-test images.

Install prerequisites:

.. code::

    apt install virtualbox vagrant


Download and setup the base image:

.. code::

    vagrant up

Login to the virual machine:

.. code::

    vagrant ssh


At this point, install CKAN/DCOR according to the description at
https://dc.readthedocs.io/en/latest/sec_self_hosting.html.


Also, make sure that the following packages are installed in the ckan environment:

.. code::

    pip install codecov coverage pytest-ckan


When job is finished then create the box

.. code::

    # cleanup
    apt autoremove
    apt clean
    dd if=/dev/zero of=/root/zero.bin bs=1M count=1000 && rm -rf /root/zero.bin
    # zero-out swap
    dd if=/dev/zero of=/dev/sda2 bs=1M
    mkswap /dev/sda2  # and edit /etc/fstab with new UUID
    # create package
    vagrant package

