A 64 bit virtual machine for Machine Learning/Data Science tasks. 
Generated and provisioned with Vagrant.

This instance builds on the tid-spark/base64 VM to supply a VM with an Spark 
notebook process, exported as an HTTP service to a local port.

Requirements are:
* Virtualbox 5
* Vagrant 1.7.4

A "vagrant up" command in this folder should be enough to download the base box,
start the vm and provision it. Before doing so, you may want to customize some 
VM options by looking at the top of the Vagrantfile.
