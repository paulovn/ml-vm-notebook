# Spark Virtual Machine

A 64 bit virtual machine for Machine Learning/Data Science tasks. 
Generated and provisioned with Vagrant.

This instance builds on the `tid-spark/base64` VM (which already provides all 
the needed software packages). On top of that, it configures and launches a
Jupyter Notebook process, exported as an HTTP service to a local port. It 
allows creating notebooks with four different kernels:
  * Python 2.7 (plain), 
  * Python 2.7 + Spark (pyspark),
  * Scala + Spark
  * R (with SparkR available, though not loaded by default).

The repository also contains a number of example notebooks.

The contents of the VM are:

* Apache Spark 1.6.1
* Python 2.7.8 from the Software Collections
* A virtualenv for Python 2.7.8 with a scientific Python stack (scipy, numpy,
* matplotplib, pandas, statmodels, scikit-learn, gensim, networkx, theano+keras, mpld3, seaborn) plus IPython 4 + Jupyter notebook
* R 3.3.0 with a few packages installed (rmarkdown, magrittr, dplyr, tidyr, data.table, ggplot2, caret plus their dependencies)
* Spark notebook Kernels for Python 2.7, Scala ([Toree](https://toree.incubator.apache.org/)) and R ([IRKernel](https://github.com/IRkernel/IRkernel)), in addition to the default "plain" (i.e. non-Spark capable) Python 2.7 kernel.
* A few small [notebook extensions](https://github.com/paulovn/nbextensions)
* A notebook startup daemon script with facilities to configure Spark execution mode
* Three additional Spark external libraries:
  - The Kafka Spark Streaming artifact
  - The [Spark CSV](https://github.com/databricks/spark-csv) library
  - The [GraphFrames](http://graphframes.github.io/) Spark package.

**Important**: in this installation the default Python kernel for notebooks 
is **not** Spark-aware. Hence Python Notebooks running Spark tasks that were 
created with previous versions of this VM will not work initially. 
They can be made to work by changing its kernel (use the option in the menubar)
to the "Pyspark" kernel. Once done, it is stored in the notebook, so saving
it will make it work in future executions (but the saved notebook will *not*
work in a previous version of the VM).


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
   Specially interesting might be the amount of RAM assigned to the Virtual 
   Machine and, if access to a remote Spark cluster is sought, the IP address 
   of the cluster. 
   Note that no customization is needed to make the VM work (i.e. it will 
   happily work with no changes to the Vagrantfile)

3. Open a console/terminal, move (`cd`) to the folder where the Vagrantfile is 
   located and execute a `vagrant up` command.

4. Vagrant should launch the process and download the base box from the public 
   repository (this is only done once).

5. Then the VM will be started and provisioned. The process will print progress 
   messages to the terminal.

Note that the base box (the one that was created by the [base repository](https://github.com/paulovn/machine-learning-vm)) should be accessible when provisioning this VM. 
The default URL in the Vagrantfile points to a box publicly available in ATLAS,
so there should be no problem.


## Operation

Once installation finishes, a notebook server will be running on
[`http://localhost:8008`](http://localhost:8008). Use a Web browser to access
it. By default it is also accessible externally, i.e. other machines in the
network can also connect to it (unless the host computer has a firewall that
blocks port 8008).

The additional files in the ZIP create a layout for sharing content between
the host and the VM:
 * The `vmfiles` subfolder in the host is configured to be mounted inside the
   VM as the `/vagrant` directory, so anything in that subfolder can be 
   accessed within the VM.
 * Furthermore, the notebook server is configured to browse the files in the 
   `vmfiles/IPNB` subdirectory, so to add notebooks place them in that 
   subdirectory. A few example mini-notebooks are already provided there.

The Jupyter notebook server starts automatically. It can be managed
(start/stop/restart) in a console session (see below) via:

    sudo service notebook (start | stop | restart)


### Console

In addition to creating, editing and executing Notebooks via the Web interface,
there may also be the need to operate through a console session. There are 
three ways to obtain console access to the VM:

1. By using the console option in the Notebook Web interface (use the
   right menu: *New* -> *Terminal*)

2. By opening a text terminal on the host machine, changing to the directory
   where the Vagrantfile is located and executing `vagrant ssh`

3. By using a standard SSH application (`ssh` in Linux, in Windows e.g. 
   [PuTTY](http://www.putty.org/)). Connect to the following destination:
    - Host: 127.0.0.1 (localhost)
    - Port: 2222
    - User: vagrant
    - Password: vagrant

In the two first cases the user logged in the console session will be `vmuser`
(or, if that was changed in the Vagrantfile, the username defined there). In
the last case it will be `vagrant`. 
* The `vmuser` user is intended to execute processing tasks (and is the one 
  running the Jupyter Notebook server), including Spark command-line 
  applications such as `spark-submit`, `spark-shell`, `pyspark`, etc as
  well as Python commands (use `ipython` or `python2.7`, not the bare `python`
  command, which executes the base OS Python 2.6).
* The `vagrant` user is intended for administrative tasks (and is the owner of 
  all the installed Python & Spark stack).

Both users have `sudo` permissions, in case it is needed for system 
administration tasks.

Once we have a console session, it can also be used to edit the files inside 
the VM. For this the  VM includes the standard `vi` and `emacs` editors (for 
text consoles), as well as `nano`, a lightweight editor. Note that files in 
the `/vagrant` directory can be edited directly in the host, since it is 
a mounted folder.


### Spark administration

Inside the VM (i.e. as seen from within a console session), Spark is installed 
in `/opt/spark/current/`. The two important Spark config files are:
* `/opt/spark/current/conf/spark-env.sh`: environment variables used by the
  Spark deployment inside the VM
* `/opt/spark/current/conf/spark-defaults.conf`: configuration properties
  used by Spark

The Spark kernel in Jupyter Notebook launches with the configuration defined by 
those two files. If they are changed, Spark kernels currently running will 
need to be restarted to make them read the new values.

Command-line spark processes launched from a console session (through
`spark-submit`, `pyspark`, etc) use also the same configuration, unless 
overriden by command-line arguments.


### Security

The VM has not been secured. The base box comes with the default Vagrant key, 
and this is overwritten with a private key when launched for the first time, 
so ssh connections with certificate are more or less secure. But there are a 
number of security holes, among them:
  * Both the `root` and `vagrant` users use `vagrant` as password.
  * Jupyter notebook listens on port 8008 with no restrictions (so that
    if the host computer has no firewall, it can be accessed from anywhere)


### RStudio

This version of the Vagrantfile installs a couple of additional packages onto the base box:
 * The `neuralnet` R package
 * RStudio server, forwarding its port (8787) to the host machine. Therefore, 
   to access the RStudio interface, go to http://localhost:8787
