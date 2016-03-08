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

### Requirements

* Hardware & OS: A computer with enough free RAM (at least 2 GB is advisable), and 
  around 10 GB of hard disk space, with a 64-bit Windows (7 or above), Linux 64 bits 
  (Ubuntu, RedHat/CentOS, etc) or Mac OS X
* Software: The following must be installed in the computer:
  * [Virtualbox](https://www.virtualbox.org/) 5.0 or above
  * [Vagrant](https://www.vagrantup.com/) 1.7.4 or above

### Process

1. Copy the Vagrantfile + examples into the computer, either by cloning the repository
   or by downloading and uncompressing the ZIP file. Make sure to use a disk or 
   partition with the mentioned 10 GB of free space.
2. Open the Vagrantfile with a text editor and customize, if desired, the options at 
   the top of the file; see the relevant comments. Specially interesting might be 
   the amount of RAM assigned to the Virtual Machine and, if access to a remote 
   cluster is sought, the IP address of the cluster (note that the VM will also work
   with no changes to the Vagrantfile)
3. Open a console/terminal, move (`cd`) to the folder where the Vagrantfile is located
   and execute a `vagrant up` command.
4. Vagrant should launch the process and download the base box from the public 
   repository (this is only done once).
5. Then the VM will be started and provisioned. The process will print progress 
   messages to the terminal.

Note that the base box (the one that was created by the [base repository](https://github.com/paulovn/machine-learning-vm)) should be accessible when provisioning this VM. 
The default URL in the Vagrantfile points to a box publicly available in ATLAS, so 
there should be no problem.

### Operation

Once done, there are two ways of accessing the Spark system:

* A notebook server will be running on [`http://localhost:8008`](http://localhost:8008). Use a Web browser to access it.
* A console session in the VM can be obtained by executing `vagrant ssh`. Once in
  the session, the command-line Spark applications are available (`spark-submit`,
  `spark-shell`, `pyspark`, etc). The logged user has `sudo` permissions.

The `vmfiles` subfolder is configured to be mounted inside the VM as `/vagrant`,
so anything in that subfolder can be accessed within the VM.

Furthermore, the notebook server is configured to browse the files in the 
`vmfiles/IPNB` subdirectory, so to add notebooks place them in it. A few example 
mini-notebooks are already provided there.


