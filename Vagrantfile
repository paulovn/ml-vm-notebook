# -*- mode: ruby;  ruby-indent-tabs-mode: t -*-
# vi: set ft=ruby :
# **************************************************************************
# Add specific configuration for running IPython notebooks on a VM
# **************************************************************************

# --------------------------------------------------------------------------
# Variables defining the configuration of notebook 
# Modify as needed

# RAM memory used for the VM, in MB
vm_memory = '2048'
# Number of CPU cores assigned to the VM
vm_cpus = '1'

# Password to use to access the Notebook web interface
vm_password = 'vmuser'

# Username that will run all processes.
vm_username = 'vmuser'

# The virtual machine exports the port where the notebook process by
# forwarding it to this port of the local machine
# So to access the notebook server, you point to http://localhost:<port>
port_nb = 8008


# --------------------------------------------------------------------------
# Vagrant configuration

# Check the command requested -- if ssh we'll change the login user
vagrant_command = ARGV[0]

port_nb_internal = 8008

# The "2" in Vagrant.configure sets the configuration version
Vagrant.configure(2) do |config|

  # This is to avoid Vagrant inserting a new SSH key, instead of the
  # default one (perhaps because the box will be later packaged)
  #config.ssh.insert_key = false

  # vagrant-vbguest plugin: set auto_update to false, if you do NOT want to
  # check the correct additions version when booting this machine
  if Vagrant.has_plugin?("vagrant-vbguest") == true
    config.vbguest.auto_update = false
  end

  # Use our custom username, instead of the default "vagrant"
  if vagrant_command == "ssh"
      config.ssh.username = vm_username
  end
  #config.ssh.username = "vagrant"
  #config.vm.box_download_insecure = true

  config.vm.define "vm-machinelearning-box" do |vgrml|

    # The base box we are using. As fetched from ATLAS
    vgrml.vm.box = "paulovn/ml-base64"
    vgrml.vm.box_version = "= 3.1.0"

    # Alternative place: box elsewhere
    #vgrml.vm.box_url = "http://tiny.cc/ml-base64-310-box"
    # Alternative place: local box
    #vgrml.vm.box_url = "file:///almacen/VM/Export/VagrantBox/ml-base64-LOCAL.json"

    # Disable automatic box update checking. If you disable this, then
    # boxes will only be checked for updates when the user runs
    # `vagrant box outdated`. This is not recommended.
    # vgrml.vm.box_check_update = false

    # Deactivate the usual synced folder and use instead a local subdirectory
    # Tweak group permissions to allow writing by users other than "vagrant"
    vgrml.vm.synced_folder ".", "/vagrant", disabled: true
    vgrml.vm.synced_folder "vmfiles", "/vagrant",
      mount_options: ["dmode=775","fmode=664"],
      disabled: false
    #owner: vm_username
    #auto_mount: false

    # Customize the virtual machine: set hostname & allocated RAM
    vgrml.vm.hostname = "vm-machinelearning"
    vgrml.vm.provider :virtualbox do |vb|
      # Set the hostname in VirtualBox
      vb.name = vgrml.vm.hostname.to_s
      # Customize the amount of memory on the VM
      vb.memory = vm_memory
      # Set the number of CPUs
      vb.cpus = vm_cpus
      # Use the DNS proxy of the NAT engine (helps in some VPN environments)
      #vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      # Control guest clock adjustment
      vb.customize ["guestproperty", "set", :id,
                    "/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold",
                    10000]
      vb.customize ["guestproperty", "set", :id,
                    "/VirtualBox/GuestAdd/VBoxService/--timesync-set-on-restore",
                    1]
      # Display the VirtualBox GUI when booting the machine
      #vb.gui = true
      # Adjust copy/paste between guest & host (for GUI startups)
      vb.customize ["modifyvm", :id, "--clipboard", "bidirectional"]
      vb.customize ["modifyvm", :id, "--draganddrop", "bidirectional"]
    end

    # **********************************************************************
    # Networking

    # ---- NAT interface ----
    # NAT port forwarding
    vgrml.vm.network :forwarded_port,
     #auto_correct: true,
     guest: port_nb_internal,
     host: port_nb                  # Notebook UI

    # RStudio server
    # =====> if using RStudio, uncomment the following line and reload the VM
    #vgrml.vm.network :forwarded_port, host: 8787, guest: 8787

    # ---- bridged interface ----
    # Declare a public network

    # =====> Uncomment the following two lines to enable bridge mode:
    #vgrml.vm.network "public_network",
    #type: "dhcp"

    # ===> if the host has more than one interface, we can set which one to use
    #bridge: "wlan0"
    # ===> we can also set the MAC address we will send to the DHCP server
    #:mac => "08002710A7ED"

    # ---- private interface ----
    # Create a private network, which allows host-only access to the machine
    # using a specific IP.
    #vgrml.vm.network "private_network", ip: "192.72.33.10"


    vgrml.vm.post_up_message = "**** The Vagrant ML-Notebook machine is up. Connect to http://localhost:" + port_nb.to_s + " for notebook access"


    # **********************************************************************
    # Standard provisioning: install configuration files and startup scripts
    # These are run by default upon VM installation

    # .........................................
    # Create the user to run jobs (esp. notebook processes)
    vgrml.vm.provision "01.nbuser",
    type: "shell",
    privileged: true,
    args: [ vm_username, vm_password ],
    inline: <<-SHELL
      # Create user
      id "$1" >/dev/null 2>&1 || useradd -c 'VM User' -m -G vagrant,sudo "$1" -s /bin/bash
      # Set the password for the user
      echo "$1:$2" | chpasswd

      # Create the .bash_profile file
      cat <<'ENDPROFILE' > /home/$1/.bash_profile
# .bash_profile - IPNB

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
   . ~/.bashrc
fi

# Add user dir to PATH
export PATH=$HOME/bin:$PATH:$HOME/.local/bin
# Place where to keep user R packages (used outside RStudio Server)
export R_LIBS_USER=~/.Rlibrary
# Jupyter uses this to define datadir but it is undefined when using "runuser"
test "$XDG_RUNTIME_DIR" || export XDG_RUNTIME_DIR=/run/user/$(id -u)
ENDPROFILE
      chown $1.$1 /home/$1/.bash_profile

      # Create some local files as the designated user
      su -l "$1" <<'USEREOF'
for d in bin tmp .ssh .jupyter .Rlibrary; do test -d $d || mkdir $d; done
chmod 700 .ssh
PYVER=$(ls -d /opt/ipnb/lib/python?.? | xargs -n1 basename)
rm -f bin/{python,$PYVER.7,pip,ipython,jupyter}
ln -s /opt/ipnb/bin/{python,$PYVER,pip,ipython,jupyter} bin
test -h IPNB || { rm -f IPNB; ln -s /vagrant/IPNB/ IPNB; }
echo 'alias dir="ls -al"' >> ~/.bashrc
echo 'PS1="\\h#\\# \\W> "'   >> ~/.bashrc
USEREOF

      # Install the vagrant public key so that we can ssh to this account
      cp -p /home/vagrant/.ssh/authorized_keys /home/$1/.ssh/authorized_keys
      chown $1.$1 /home/$1/.ssh/authorized_keys
    SHELL

    # .........................................
    # Create the IPython Notebook profile
    # Prepared for IPython >=4 (so that we configure as a Jupyter app)
    vgrml.vm.provision "10.config",
    type: "shell",
    privileged: true,
    keep_color: true,
    args: [ vm_username, vm_password, port_nb_internal ],
    inline: <<-SHELL
     USERNAME=$1
     NOTEBOOK_BASEDIR=/home/$USERNAME/IPNB
     PASS=$(/opt/ipnb/bin/python -c "from notebook.auth.security import passwd; print(passwd('$2'))")

     # --------------------- Create the Jupyter config
     echo "Creating Jupyter config"
     CFGFILE="/home/$USERNAME/.jupyter/jupyter_notebook_config.py"
     cat <<-EOF >$CFGFILE
from os import environ
c.NotebookApp.ip = '0.0.0.0'
c.NotebookApp.port = $3
c.NotebookApp.password = '$PASS'
c.NotebookApp.notebook_dir = environ.get('NOTEBOOK_BASEDIR','$NOTEBOOK_BASEDIR')
c.NotebookApp.open_browser = False
c.NotebookApp.log_level = 'INFO'
EOF
     chown $USERNAME.$USERNAME $CFGFILE
    SHELL

    vgrml.vm.provision "13.ir",
    type: "shell",
    privileged: true,
    keep_color: true,
    args: [ vm_username ],
    inline: <<-SHELL
     USERNAME=$1
     KDIR=/home/$USERNAME/.local/share/jupyter/kernels

     # --------------------- Install the IRkernel
     echo "Installing IRkernel ..."
     su -l "$USERNAME" <<EOF
       PATH=/opt/ipnb/bin:$PATH Rscript -e 'IRkernel::installspec()'
EOF
    SHELL

    vgrml.vm.provision "30.extensions",
    type: "shell",
    privileged: true,
    keep_color: true,
    args: [ vm_username ],
    inline: <<-SHELL
     USERNAME=$1
     su -l "$USERNAME" <<EOF
# --------------------- Install the notebook extensions
echo "Installing notebook extensions"
/opt/ipnb/bin/python -c 'from notebook.services.config import ConfigManager; ConfigManager().update("notebook", {"load_extensions": {"toc": True, "toggle-headers": True, "search-replace": True, "python-markdown": True }})'
ln -fs /opt/ipnb/share/jupyter/pre_pymarkdown.py /opt/ipnb/lib/python?.?/site-packages

# --------------------- Put the custom Jupyter icon in place
cd /opt/ipnb/lib/python?.?/site-packages/notebook/static/base/images
mv favicon.ico favicon-orig.ico
ln -s favicon-custom.ico favicon.ico
EOF
    SHELL

    # .........................................
    # Install the Notebook startup script & configure it
    vgrml.vm.provision "31.nbconfig",
    type: "shell",
    privileged: true,
    keep_color: true,
    args: [ vm_username ],
    inline: <<-SHELL
     # Create the system config for IPython notebook
     CFGD=/etc/sysconfig
     test -d ${CFGD} || { CFGD=/etc/jupyter; mkdir $CFGD; }
     cat <<-EOF > $CFGD/jupyter-notebook
NOTEBOOK_USER="$1"
NOTEBOOK_SCRIPT="/opt/ipnb/bin/jupyter-notebook"
NOTEBOOK_BASEDIR="/home/$1/IPNB"
EOF
     # Link the IPython mgr script so that it can be found by root
     SCR=jupyter-notebook-mgr
     chmod 775 /opt/ipnb/bin/$SCR
     rm -f /usr/sbin/$SCR
     ln -s /opt/ipnb/bin/$SCR /usr/sbin

     # Enable the service
     systemctl enable notebook

     # Start the service
     echo "Starting notebook service"
     systemctl start notebook
  SHELL

    # *************************************************************************
    # Optional provisioning
    # These need to be run explicitly


    # .........................................
    # Install RStudio server
    # *** Don't forget to also uncomment forwarding for port 8787!
    vgrml.vm.provision "rstudio",
      type: "shell",
      run: "never",
      keep_color: true,
      privileged: true,
      args: [ vm_username, vm_password ],
      inline: <<-SHELL
        echo "Downloading & installing RStudio Server"
        apt-get update
        apt-get install -y gdebi-core
        # Download & install the package for RStudio Server
        PKG=rstudio-server-2021.09.1-372-amd64.deb
        wget --no-verbose https://download2.rstudio.org/server/bionic/amd64/$PKG
        gdebi -n $PKG && rm -f $PKG
        # Define the directory for the user library, and the working directory
        CNF=/etc/rstudio/rsession.conf
        grep -q r-libs-user $CNF || cat >>$CNF <<EOF
r-libs-user=~/.Rlibrary
session-default-working-dir=/home/$1/R
session-default-new-project-dir=/home/$1/R
EOF
        # Create a link to the host-mounted R subdirectory
        sudo -i -u "$1" bash -c "rm -f R; ln -s /vagrant/R/ R"
        # Set the password for the user, so that it can log in in RStudio
        echo "$1:$2" | chpasswd
        # Send message
        echo "RStudio Server should be accessed at http://localhost:8787"
        echo "(if not, check in Vagrantfile that port 8787 has been forwarded)"
      SHELL

    # .........................................
    # Install the necessary components for nbconvert to work.
    vgrml.vm.provision "nbc",
      type: "shell",
      run: "never",
      privileged: true,
      keep_color: true,
      args: [ vm_username ],
      inline: <<-SHELL
          echo "Installing nbconvert requirements"
          apt-get update && apt-get install -y --no-install-recommends pandoc texlive-latex-recommended texlive-plain-generic texlive-xetex texlive-fonts-recommended lmodern
          # We modify the LaTeX template to generate A4 pages
          # (comment this out to keep Letter-sized pages)
          perl -pi -e 's|(\\\\geometry\\{)|${1}a4paper,|' /opt/ipnb/share/jupyter/nbconvert/templates/latex/base.tex.j2

      SHELL

    # .........................................
    # Optional: modify nbconvert to process Spanish documents
    vgrml.vm.provision "nbc.es",
      type: "shell",
      run: "never",
      privileged: false,
      keep_color: true,
      inline: <<-SHELL
          # Define language
          LANGUAGE=spanish
          echo "** Adding support for $LANGUAGE to LaTeX"
          sudo apt-get install -y texlive-lang-spanish
          echo "** Converting base LaTeX template for $LANGUAGE"
          perl -pi -e 's|(\\\\usepackage\\{fontspec})|${1}\\\\usepackage{polyglossia}\\\\setmainlanguage{'$LANGUAGE'}|' /opt/ipnb/share/jupyter/nbconvert/templates/latex/base.tex.j2
      SHELL

    # .........................................
    # Install additional packages for NLP
    vgrml.vm.provision "nlp",
      type: "shell",
      run: "never",
      privileged: true,
      keep_color: true,
      args: [ vm_username ],
      inline: <<-SHELL
        echo "Installing additional NLP packages"
        # pattern is Python 2 only
        su -l "vagrant" -c "pip install nltk sklearn_crfsuite spacy"
      SHELL

    # .........................................
    # Install Maven
    vgrml.vm.provision "mvn",
      type: "shell",
      run: "never",
      privileged: true,
      keep_color: true,
      args: [ vm_username ],
      inline: <<-SHELL
        VERSION=3.6.3
        DEST=/opt/maven
        echo "Installing Maven $VERSION"
        PKG=apache-maven-$VERSION
        FILE=$PKG-bin.tar.gz
        cd /tmp
        wget http://apache.rediris.es/maven/maven-3/$VERSION/binaries/$FILE
        rm -rf /home/$1/bin/mvn $DEST
        mkdir -p $DEST
        tar zxvf $FILE -C $DEST
        su $1 -c "ln -s $DEST/$PKG/bin/mvn /home/$1/bin"
      SHELL

    # Install Scala development tools
    vgrml.vm.provision "scala",
      type: "shell",
      run: "never",
      privileged: true,
      keep_color: true,
      args: [ vm_username ],
      inline: <<-SHELL
        # Download & install Scala
        cd install
        VERSION=2.12.11
        PKG=scala-$VERSION.deb
        echo "Downloading & installing Scala $VERSION"
        wget --no-verbose http://downloads.lightbend.com/scala/$VERSION/$PKG
        sudo dpkg -i $PKG && rm $PKG
        # Install sbt
        echo "Installing sbt"
        # Install sbt
        echo "deb https://dl.bintray.com/sbt/debian /" > /etc/apt/sources.list.d/sbt.list
        apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823 && apt-get update && apt-get install -y sbt
        # Install scala-mode for Emacs
        echo "Configuring scala-mode in Emacs"
        cat <<EOF >> /home/$1/.emacs

; Install MELPA package repository
(require 'package)
(add-to-list 'package-archives
            '("melpa-stable" . "https://stable.melpa.org/packages/") t)
(package-initialize)
; Install Scala mode
(unless (package-installed-p 'scala-mode)
    (package-refresh-contents) (package-install 'scala-mode))

EOF
         chown $1.$1 /home/$1/.emacs
      SHELL


    # .........................................
    # Install some Deep Learning frameworks
    vgrml.vm.provision "dl",
      type: "shell",
      run: "never",
      privileged: false,
      keep_color: true,
      inline: <<-SHELL
         # Tensorflow 2 needs pip >= 19.0
         pip install --upgrade "pip>19.0"
         pip install --upgrade "tensorflow-cpu>=2.2"
         pip install --upgrade torch==1.10.1+cpu \
             torchvision==0.11.2+cpu torchaudio==0.10.1+cpu \
             -f https://download.pytorch.org/whl/cpu/torch_stable.html
      SHELL


    # *************************************************************************

    # .........................................
    # Start Jupyter Notebook
    # This is normally not needed (the service starts automatically upon boot)
    vgrml.vm.provision "50.nbstart",
      type: "shell",
      run: "never",
      privileged: true,
      keep_color: true,
      inline: "systemctl start notebook"

  end # config.vm.define

end
