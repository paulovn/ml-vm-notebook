# Machine Learning Virtual Machine

A 64 bit virtual machine for Machine Learning/Data Science tasks. 
Generated and provisioned with Vagrant.

This instance builds on the `ml-base64` VM (which already provides all 
the needed software packages, on an Ubuntu 18.04). On top of that, it configures
and launches a Jupyter Notebook process, exported as an HTTP service to a local
port. It allows creating notebooks with two different kernels:
  * Python 3.6 (plain Python, with additional libraries such as NumPy, SciPy,
    Pandas, Matplotlib, Scikit-learn, etc), 
  * R

The repository also contains a number of small example notebooks.

The contents of the VM are:
* Python 3.6.6
* A virtualenv for Python 3.6.6 with a scientific Python stack (scipy, numpy, matplotplib, pandas, statmodels, scikit-learn, gensim, xgboost, networkx, seaborn, pylucene and a few others) plus IPython 7 + Jupyter notebook
* R 3.5.2 with a few packages installed (rmarkdown, magrittr, dplyr, tidyr, data.table, ggplot2, caret, plus their dependencies). 
* A few small [notebook extensions](https://github.com/paulovn/nbextensions)


## Installation

### Requirements

* Hardware & OS: A computer with enough free RAM (at least 2 GB is advisable), 
  and around 10 GB of hard disk space, with a 64-bit Windows (7 or above), 
  Linux 64 bits (Ubuntu, RedHat/CentOS, etc) or Mac OS X
* Software: The following must be installed in the computer:
  * [Virtualbox](https://www.virtualbox.org/) 5.0 or above
  * [Vagrant](https://www.vagrantup.com/) 1.8 or above

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
   Virtual Machine and. Another configurable value is the notebook
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

The base box (the one that was created by the [base repository](https://github.com/paulovn/machine-learning-vm)) must be downloadable when provisioning this VM. The
default URL in the Vagrantfile points to a box publicly available in 
[VagrantCloud](https://app.vagrantup.com/paulovn/boxes/ml-base64),
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

The Jupyter notebook server starts automatically. It can be managed
(start/stop/restart) in a VM console session (see below) via:

    sudo systemctl (start | stop | restart) notebook

For diagnostics, in addition to the messages appearing directly on notebooks,
logfiles are generated in `/var/log/ipnb` inside the VM:
 * `/var/log/ipnb/jupyter-notebook.out` contains log messages from the
   Jupyter server

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
  running the Jupyter Notebook server), as well as Python commands (use either
  `ipython` or `python`, which points to the virtualenv where all is installed).
* The `vagrant` user is intended for administrative tasks (and is the owner of
  all the installed Python stack).

Both users have `sudo` permissions, in case it is needed for system 
administration tasks.

Once we have a console session, it can also be used to edit the files inside
the VM. For this the  VM includes the standard `vi` and `emacs` editors (for
text consoles), as well as `nano`, a lightweight editor. Note that files in the
`/vagrant` directory can be edited directly in the host, since it is a mounted
folder.


### Security

The VM has not been secured. The base box comes with the default Vagrant key,
and this is overwritten with a private key when launched for the first time, so
ssh connections with certificate are more or less secure. But there are a number
of security holes, among them:
  * Both the `root` and `vagrant` users use `vagrant` as password.
  * Jupyter notebook listens on port 8008 with a very trivial password
    (if the host computer has no firewall, it can be accessed from anywhere)

## Extra packages

There are a number of packages that have been defined in the `Vagrantfile` but
that are not automatically installed. Instead, they must be explicitly
provisioned with:

      vagrant provision --provision-with <name>

It includes the following list:

| name | package contents |
| ---- | ---------------- |
| rstudio | RStudio Server. See below for details |
| nbc | Notebook convert (functionality for Notebook conversion to document formats: LaTeX & PDF) |
| nbc.es | Configure Notebook conversion to documents for Spanish |
| nlp | Some additional Python packages for Natural Language Processing |
| mvn | Maven build automation tool for Java |
| scala  | Scala & SBT |
| dl | Deep Learning libraries (Keras, Theano, Tensorflow) |


Note: for RStudio it will be also necessary to open port 8787 in the
Vagrantfile and reload it. The user/password combination to be used is `vmuser`
& `vmuser`
