# Spark Virtual Machine

A 64 bit virtual machine for Machine Learning/Data Science tasks. 
Generated and provisioned with Vagrant.

This instance builds on the `spark-base64` VM (which already provides all 
the needed software packages). On top of that, it configures and launches a
Jupyter Notebook process, exported as an HTTP service to a local port. It 
allows creating notebooks with four different kernels:
  * Python 2.7 (plain Python, with additional libraries such as NumPy, SciPy,
    Pandas, Matplotlib, Scikit-learn), 
  * Pyspark (Python 2.7 + libraries + Spark),
  * Scala 2.11 + Spark
  * R (with SparkR available, though not loaded by default).

The repository also contains a number of small example notebooks.

The contents of the VM are:

* [Apache Spark](http://spark.apache.org/) 2.0.2
* Python 2.7.5
* A virtualenv for Python 2.7.5 with a scientific Python stack (scipy, numpy, matplotplib, pandas, statmodels, scikit-learn, gensim, networkx, theano+keras, mpld3, seaborn) plus IPython 5 + Jupyter notebook
* R 3.3.1 with a few packages installed (rmarkdown, magrittr, dplyr, tidyr, data.table, ggplot2, caret, plus their dependencies). Plus SparkR & [sparklyr](http://spark.rstudio.com/) for interaction with Spark.
* Spark notebook Kernels for Python 2.7, Scala ([Toree](https://toree.incubator.apache.org/)) and R ([IRKernel](https://github.com/IRkernel/IRkernel)), in addition to the default "plain" (i.e. non-Spark capable) Python 2.7 kernel.
* A few small [notebook extensions](https://github.com/paulovn/nbextensions)
* A notebook management script with facilities to configure Spark execution mode
 * RStudio Server, listening on port 8787 (see [below](#rstudio)).

**Important**: the default Python kernel for notebooks is **not** Spark-aware.
To develop notebooks in Python for Spark, the `Pyspark (Py 2)` kernel must be
specifically selected. Hence Spark Python Notebooks that were created elsewhere
(or with former versions of this VM) will not work initially. 
They can be made to work by changing its kernel (use the option in the menubar)
to the "Pyspark" kernel. Once done, the change is stored in the notebook, so
saving it will make it work in future executions.


## Installation

### Requirements

* Hardware & OS: A computer with enough free RAM (at least 2 GB is advisable), 
  and around 10 GB of hard disk space, with a 64-bit Windows (7 or above), 
  Linux 64 bits (Ubuntu, RedHat/CentOS, etc) or Mac OS X
* Software: The following must be installed in the computer:
  * [Virtualbox](https://www.virtualbox.org/) 5.0 or above
  * [Vagrant](https://www.vagrantup.com/) 1.7.4 or above

### Process

1. Copy the Vagrantfile + examples into the computer, either by cloning the 
   repository or by downloading and extracting all files in the packaged
   [ZIP file](https://github.com/paulovn/ml-vm-notebook/archive/develop.zip). 
   Make sure to use a disk or partition with the mentioned 10 GB of free space.
   Also, in Windows it might be advisable to avoid using a folder name with
   spaces (since sometimes it causes problems).

2. If desired, open the Vagrantfile with a text editor and customize the 
   options at the top of the file; see the relevant comments. 
   Specially interesting might be the amount of RAM/CPUs assigned to the
   Virtual Machine and, if access to a remote Spark cluster is sought, the
   IP address of the cluster. Another configurable value is the notebook
   access password (in the `vm_password` variable).
   Note that no customization is needed to make the VM work (i.e. it will 
   happily work with no changes to the Vagrantfile)

3. Open a console/terminal, move (`cd`) to the folder where the Vagrantfile is
   located and execute a `vagrant up` command.

4. Vagrant should launch the process and download the base box from the public
   repository (this takes time, depending on your network bandwidth, but it is
   only done once).

5. Then the VM will be started and provisioned. The process will print progress
   messages to the terminal.

The base box (the one that was created by the [base repository](https://github.com/paulovn/machine-learning-vm)) must be downloadable when provisioning this VM.

The default URL in the Vagrantfile points to a box publicly available in ATLAS,
so there should be no problem as long as there is a working Internet connection.


## Operation

Once installation finishes, a notebook server will be running on
[`http://localhost:8008`](http://localhost:8008). Use a Web browser to access
it. By default it is also accessible externally, i.e. other machines in the
network can also connect to it (unless the host computer has a firewall that
blocks port 8008).

Using the notebook interface will require the access password defined in
the `Vagrantfile` (unless changed before installation, it will be `vmuser`)

The additional files in the ZIP create a layout for sharing content between
the host and the VM:

 * The `vmfiles` subfolder in the host is configured to be mounted inside
   the VM as the `/vagrant` directory, so anything in that subfolder can be 
   accessed within the VM.
 * Furthermore, the notebook server is configured to browse the files in the
   `vmfiles/IPNB` subdirectory, so to add notebooks place them in that 
   subdirectory. A few example mini-notebooks are already provided there.
 * Finally, the `vmfiles/hive` subdirectory is the place configured in
   Spark SQL for its metastore & tables (so it should survive to changes in
   the VM).

The Jupyter notebook server starts automatically. It can be managed
(start/stop/restart) in a console session (see below) via:

    sudo systemctl (start | stop | restart) notebook

For diagnostics, in addition to the messages appearing directly on notebooks,
logfiles are generated in `/var/log/ipnb` inside the VM:

 * `/var/log/ipnb/jupyter-notebook.out` contains log messages from the Jupyter 
   server
 * `/var/log/ipnb/jupyter-notebook.out` contains log messages from the
   Jupyter server
 * `/var/log/ipnb/spark.log` contains log messages from Spark

### Console

In addition to creating, editing and executing Notebooks via the Web interface,
there may also be the need to operate through a console session. There are
three ways to obtain console access to the VM:

1. By using the console option in the Notebook Web interface (use the right
   menu: *New* -> *Terminal*)

2. By opening a text terminal on the host machine, changing to the directory
   where the Vagrantfile is located and executing `vagrant ssh`

3. By using a standard SSH application (`ssh` in Linux, in Windows e.g. 
   [PuTTY](http://www.putty.org/)). Connect to the following destination:
    - Host: 127.0.0.1 (localhost)
    - Port: 2222
    - User: `vagrant`
    - Password: `vagrant`

In the two first cases the user logged in the console session will be `vmuser`
(or, if that was changed in the Vagrantfile, the username defined there). In the
last case it will be `vagrant`. 

* The `vmuser` user is intended to execute processing tasks (and is the one 
  running the Jupyter Notebook server), including Spark command-line 
  applications such as `spark-submit`, `spark-shell`, `pyspark`, etc as well as
  Python commands (use either `ipython` or `python`, which points to the
  virtualenv where all is installed).
  
* The `vagrant` user is intended for administrative tasks (and is the owner of
  all the installed Python & Spark stack).

Both users have `sudo` permissions, in case it is needed for system 
administration tasks.

Once we have a console session, it can also be used to edit the files inside
the VM. For this the  VM includes the standard `vi` and `emacs` editors (for
text consoles), as well as `nano`, a lightweight editor. Note that files in the
`/vagrant` directory can be edited directly in the host, since it is a mounted
folder.


### Spark administration

Inside the VM (i.e. as seen from within a console session), Spark is installed
in `/opt/spark/current/`. The two important Spark config files are:

 * `/opt/spark/current/conf/spark-env.sh`: environment variables used by the
   Spark deployment inside the VM
 * `/opt/spark/current/conf/spark-defaults.conf`: configuration properties
   used by Spark

The Spark kernel in Jupyter Notebook launches with the configuration defined
by those two files. If they are changed, Spark kernels currently running will
need to be restarted to make them read the new values.

Command-line spark processes launched from a console session (through
`spark-submit`, `pyspark`, etc) use also the same configuration, unless 
overriden by command-line arguments.

An additional configuration file is `/opt/spark/current/conf/hive-site.xml`.
This one is used by Spark SQL to determine the behaviour related to Hive tables.
In particular, two important bits defined there are the location of the 
metastore DB (the VM uses Derby as a local metastore) and the place where 
system tables will be written. Both places are configured to subfolders of the
`./vmfiles/hive` directory in the host (mounted in the VM).

Note that due to Derby being a single-process DB, only one Spark SQL process
may be active at any given moment (since it would collide over the use of the
metastore). This means that there should be only one active Notebook (or Spark
shell) making use of an `sqlContext`. 

To allow more than one Notebook at the same time, the 
`javax.jdo.option.ConnectionURL` configuration property in `hive-site.xml` 
should be commented out; this would revert Spark to the default behaviour of
creating the metastore in the local directory in which the Notebook is running
(hence there can be more than one active Notebook running Spark SQL, provided
that they are in different directories).
 

### RStudio

This version of the Vagrantfile installs a couple of additional packages onto 
the base box:

 * The `neuralnet` R package
 * [RStudio Server](https://support.rstudio.com/hc/en-us/articles/200552306-Getting-Started), forwarding its port (8787) to the host machine. 
   Therefore, to access the RStudio interface, go to [`http://localhost:8787`](http://localhost:8787). The credentials to use are: user `vmuser`, password `vmuser`. 
   The local directory `vmfiles/R` is intended to be used to hold all R code 
   developed withon RStudio: R files here will be visible in both the host and 
   the VM. To make RStudio Server point to this directory by default, set the 
   option in
      _Tools -> Global Options -> Default working directory_


### Security

The VM has not been secured. The base box comes with the default Vagrant key,
and this is overwritten with a private key when launched for the first time, so
ssh connections with certificate are more or less secure. But there are a number
of security holes, among them:

  * Both the `root` and `vagrant` users use `vagrant` as password.
  * The `vmuser` user has `vmuser` as password (if RStudio has been installed)
  * Jupyter notebook listens on port 8008 with a very trivial password.
    (if the host computer has no firewall, it can be accessed from anywhere)

