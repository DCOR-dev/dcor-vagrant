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
Let's use the `ckanext-dcor_schemas <https://github.com/DCOR-dev/ckanext-dcor_schemas>`_ repository as an example.
If you clone into it, you will have the files
`Vagrantfile <https://github.com/DCOR-dev/ckanext-dcor_schemas/blob/master/Vagrantfile>`_
(the recipe for setting up the box) and
`vagrant-run-tests.sh <https://github.com/DCOR-dev/ckanext-dcor_schemas/blob/master/vagrant-run-tests.sh>`_
(that installs the current working tree into the CKAN environment and executes the tests).
When you run ``vagrant up`` in this directory, vagrant uses ``Vagrantfile`` to setup the virtualbox
image (download and rsync the current working directory of the host to `/testing` on the guest).
To run the tests, simply run::

    vagrant ssh -- sudo bash /testing/vagrant-run-tests.sh



Basic box creation workflow
===========================

This is only necessary if you want to create an image from scratch (from an existing
ubuntu box). If you only have to do something incrementally, you can just base your
box on the existing dcor-test images.

Install prerequisites::

    apt install virtualbox vagrant

Download and setup the base image::

    cd from_scratch
    vagrant up

Login to the virual machine::

    vagrant ssh

At this point, install CKAN/DCOR according to the description at
https://dc.readthedocs.io/en/latest/sec_self_hosting.html.

Also, make sure that the following packages are installed in the ckan environment::

    pip install codecov coverage pytest-ckan


When job is finished then create the box::

    # cleanup
    apt autoremove
    apt clean
    # choose the zeroing size according ot the difference in `df -h` and
    # the file size in ~/VirtualBox VMs/from_previous_default_...
    dd if=/dev/zero of=/root/zero.bin bs=1M count=1000 && rm -rf /root/zero.bin
    # zero-out swap
    swapoff /dev/sda2
    dd if=/dev/zero of=/dev/sda2 bs=1M
    mkswap /dev/sda2  # and edit /etc/fstab and /etc/initramfs-tools/conf.d/resume with new UUID
    # logout and create the package
    vagrant package --output dcor-test_0.1.0.box


Box upgrade workflow
====================

If apt/CKAN/DCOR packages only require updates, then it makes
sense to base a new version of an image on an existing image::

    cd from_previous
    vagrant destroy
    vagrant up
    vagrant ssh  # login and do the updating

Once finished with the update, follow the box creation steps above.
