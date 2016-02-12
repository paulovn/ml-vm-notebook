# Spark Virtual Machine

A 64 bit virtual machine for Machine Learning/Data Science tasks. 
Generated and provisioned with Vagrant.

This instance builds on the `tid-spark/base64` VM (which already provides all the
needed software packages). On top of that, it configures and launches a Spark 
notebook process, exported as an HTTP service to a local port.

The repository also contains a number of example notebooks.

The contents of the VM are:

* Apache Spark 1.6.0
* Python 2.7.5 from the Software Collections
* A virtualenv for Python 2.7.5 with a scientific Python stack (scipy, numpy, matplotplib, pandas, statmodels, gensim, networkx, scikit-learn) plus IPython 4 + Jupyter notebook
* R 3.2.2 with a few packages installed (rmarkdown, magrittr, dplyr, tidyr, data.table, ggplot2)
* Spark notebook Kernels for Scala ([Spark Kernel](https://github.com/ibm-et/spark-kernel)) and R ([IRKernel](https://github.com/IRkernel/IRkernel)), in addition to the default Python 2.7 kernel.
* A couple of small notebook extensions
* A notebook startup daemon script with facilities to configure Spark execution mode
* Two additional Spark external libraries:
  - The Kafka Spark Streaming artifact
  - The [Spark CSV] (https://github.com/databricks/spark-csv) library

## Installing

Requirements are:
* Virtualbox 5.0 or above
* Vagrant 1.7.4 or above

A `vagrant up` command in the checked-out folder (the same directory where the `Vagrantfile` is)
should be enough to download the base box, start the vm and provision it. Before doing so, 
you may want to customize some VM options by looking at the top of the Vagrantfile.

Once done, a notebook server will be running on [`http://localhost:8008`](http://localhost:8008)

The notebook server is configured to browse the files in the `vmfiles/IPNB` 
subdirectory, so to add notebooks place them there (that subdirectory is shared
between the host and the guest VM). A few example mini-notebooks are already provided 
there.

Note that the base box (the one that was created by the [base repository](https://github.com/paulovn/machine-learning-vm)) should be accessible when provisioning this VM. The default URL in the Vagrantfile 
points to a box publicly available in ATLAS, so there should be no problem.
