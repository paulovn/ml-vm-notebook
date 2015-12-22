A 64 bit virtual machine for Machine Learning/Data Science tasks. 
Generated and provisioned with Vagrant.

This instance builds on the tid-spark/base64 VM (which already provides all the
needed software packages). On top of that, it configures and launches a Spark 
notebook process, exported as an HTTP service to a local port.

The repository also contains a number of example notebooks.

Requirements are:
* Virtualbox 5.0 or above
* Vagrant 1.7.4 or above

A `vagrant up` command in this folder should be enough to download the base box,
start the vm and provision it. Before doing so, you may want to customize some 
VM options by looking at the top of the Vagrantfile.

Once done, a notebook server will be running on http://localhost:8008

Note that the base box (the one created by the `base` repository) should be 
accessible when provisioning this VM. The current URL in the Vagrantfile 
points to an intranet address, hence if needed it should be changed to the 
address from where the box can be downloaded.
