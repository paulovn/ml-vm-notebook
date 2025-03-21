# -*- mode: ruby;  ruby-indent-tabs-mode: t -*-
# vi: set ft=ruby :
# **************************************************************************
# Add specific configuration for running IPython notebooks on a Spark VM
# **************************************************************************

# --------------------------------------------------------------------------
# Variables defining the configuration of Spark & notebook
# Modify as needed

# RAM memory used for the VM, in MB
vm_memory = '2048'

# Number of CPU cores assigned to the VM
vm_cpus = '2'

# Password to use to access the Notebook web interface
vm_password = 'vmuser'

# Username that will run all spark processes.
# (if remote (yarn) mode is ever going to be used, it is advisable to change
# it to a recognizable unique name, so that it is easily identified in the
# server logs)
vm_username = 'vmuser'

# The virtual machine exports the port where the notebook process by
# forwarding it to this port of the local machine
# So to access the notebook server, you point to http://localhost:<port>
port_nb = 8008

# Note there is an additional port exported: the Spark UI driver is
# forwarded to port 4040

# Size of the swap file in MB. Use 0 for no swap
swap_size = 2000

# This defines the Spark notebook processing mode. There are three choices
# available: "local", "yarn", "standalone"
# It can be changed at runtime by executing inside the virtual machine, as
# root user, "service notebook set-mode <mode>"
spark_mode = 'local'

# -------------------
# These 3 options are used only when running non-local tasks. They define
# the access points for the remote cluster.
# They can also be modified at runtime by executing inside the virtual
# machine: "sudo service notebook set-addr <A> <B> <C>"
# **IMPORTANT**: If remote mode is to be used, the virtual machine needs
# a network interface in bridge mode. In that case uncomment the relevant
# lines in the networking section below

# [A] The location of the cluster master (the YARN Resource Manager in Yarn
# mode, or the Spark master in standalone mode)
spark_master = ''
# [B] The host running the HDFS namenode
spark_namenode = ''
# [C] The location (host:port) of the Spark History Server
spark_history_server = ''
# ------------------


# --------------------------------------------------------------------------
# Variables defining the Spark installation in the base box.
# Don't change these

# The place where Spark is deployed inside the local machine
spark_basedir = '/opt/spark'


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

  config.vm.boot_timeout = 600

  config.vm.define "spark-35" do |vgrml|

    #config.name = "vgr-pyspark"

    # The base box we are using. As fetched from ATLAS
    vgrml.vm.box = "paulovn/spark-base64"
    vgrml.vm.box_version = "= 3.5.0"

    # Alternative place: a local box
    #vgrml.vm.box_url = "file:///almacen/VM/Export/VagrantBox/spark-base64-LOCAL.json"

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

    # Customize the virtual machine: set hostname & resources (RAM, CPUs)
    vgrml.vm.hostname = "vgr-spark-35"
    vgrml.vm.provider :virtualbox do |vb|
      # Set the hostname in VirtualBox
      vb.name = vgrml.vm.hostname.to_s
      # Customize the amount of memory on the VM
      vb.memory = vm_memory
      # Set the number of CPUs
      vb.cpus = vm_cpus
      # Use the DNS proxy of the NAT engine (helps in some VPN environments)
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
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

    # Spark driver UI
    vgrml.vm.network :forwarded_port, host: 4040, guest: 4040,
     auto_correct: true
    # Spark driver UI for the 2nd application (e.g. a command-line job)
    vgrml.vm.network :forwarded_port, host: 4041, guest: 4041,
     auto_correct: true

    # RStudio server
    # =====> if using RStudio, uncomment the following line and reload the VM
    #vgrml.vm.network :forwarded_port, host: 8787, guest: 8787

    # In case we want to fix Spark ports
    #vgrml.vm.network :forwarded_port, host: 9234, guest: 9234
    #vgrml.vm.network :forwarded_port, host: 9235, guest: 9235
    #vgrml.vm.network :forwarded_port, host: 9236, guest: 9236
    #vgrml.vm.network :forwarded_port, host: 9237, guest: 9237
    #vgrml.vm.network :forwarded_port, host: 9238, guest: 9238

    # ---- bridged interface ----
    # Declare a public network
    # This enables the machine to be connected from outside, which is a
    # must for a Spark driver [it needs SPARK_LOCAL_IP to be set to
    # the outside-visible interface].
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


    vgrml.vm.post_up_message = "**** The Vagrant Spark-Notebook machine is up. Connect to http://localhost:" + port_nb.to_s + " for notebook access"

    # **********************************************************************
    # Standard provisioning: Spark configuration files and startup scripts
    # These are run by default upon VM installation

    # .........................................
    # Create a swap file
    vgrml.vm.provision "00.swap",
    type: "shell",
    privileged: true,
    args: [ swap_size ],
    inline: <<-SHELL
      if [ -f /etc/fstab.swap -a "$1" -gt 0 ]
      then
        SWAPFILE=/swap.img
        echo "Creating swapfile ($1 MB)"
        dd if=/dev/zero of=$SWAPFILE bs=1MiB count=$1
        chmod 600 $SWAPFILE
        mkswap $SWAPFILE

        mv /etc/fstab /etc/fstab.noswap
        cp -p /etc/fstab.swap /etc/fstab

        swapon --all
      fi
    SHELL

    # .........................................
    # Create the user to run Spark jobs (esp. notebook processes)
    vgrml.vm.provision "01.nbuser",
    type: "shell",
    privileged: true,
    args: [ vm_username, vm_password ],
    inline: <<-SHELL
      # Create user
      id "$1" >/dev/null 2>&1 || useradd -c 'VM User' -m -G vagrant,sudo,vboxsf "$1" -s /bin/bash
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
# Python executable to use in a Pyspark driver (used outside notebooks)
export PYSPARK_DRIVER_PYTHON=ipython
# Place where to keep user R packages (used outside RStudio Server)
export R_LIBS_USER=~/.Rlibrary
# Jupyter uses this to define datadir but it is undefined when using "runuser"
test "$XDG_RUNTIME_DIR" || export XDG_RUNTIME_DIR=/run/user/$(id -u)
ENDPROFILE
      chown $1:$1 /home/$1/.bash_profile

      # Create some local files as the designated user
      su -l "$1" <<'USEREOF'
for d in bin tmp .ssh .jupyter .Rlibrary; do test -d $d || mkdir $d; done
chmod 700 .ssh
PYVER=$(ls -d /opt/ipnb/lib/python?.* | xargs -n1 basename)
rm -f bin/{python,$PYVER.7,pip,ipython,jupyter}
ln -s /opt/ipnb/bin/{python,$PYVER,pip,ipython,jupyter} bin
test -d IPNB || { rm -f IPNB; mkdir IPNB; cd IPNB; ln -s /vagrant/IPNB/ host; }
echo 'alias dir="ls -al"' >> ~/.bashrc
echo 'PS1="\\h#\\# \\W> "'   >> ~/.bashrc
USEREOF

      # Install the vagrant public key so that we can ssh to this account
      cp -p /home/vagrant/.ssh/authorized_keys /home/$1/.ssh/authorized_keys
      chown $1:$1 /home/$1/.ssh/authorized_keys
    SHELL

    # .........................................
    # Create the IPython Notebook profile ready to run Spark jobs
    # and install all kernels: Pyspark, Scala, IRKernel, and extensions
    # Prepared for IPython >=4 (so that we configure as a Jupyter app)
    vgrml.vm.provision "10.jupyterconf",
    type: "shell",
    privileged: true,
    keep_color: true,
    args: [ vm_username, vm_password, port_nb_internal, spark_basedir ],
    inline: <<-SHELL
     USERNAME=$1
     NOTEBOOK_BASEDIR=/home/$USERNAME/IPNB
     PASS=$(/opt/ipnb/bin/python -c "from jupyter_server.auth.security import passwd; print(passwd('$2'))")

     # --------------------- Create the Jupyter config
     echo "Creating Jupyter config"
     CFGFILE="/home/$USERNAME/.jupyter/jupyter_notebook_config.py"
     cat <<-EOF > $CFGFILE
from os import environ
c.ServerApp.ip = '0.0.0.0'
c.ServerApp.port = $3
c.ServerApp.allow_origin = '*'
c.ServerApp.root_dir = environ.get('NOTEBOOK_BASEDIR','$NOTEBOOK_BASEDIR')
c.ServerApp.open_browser = False
c.ServerApp.log_level = 'INFO'
#c.ServerApp.password = '$PASS'
c.PasswordIdentityProvider.hashed_password = '$PASS'
EOF
     chown $USERNAME:$USERNAME $CFGFILE

    SHELL

    vgrml.vm.provision "11.pyspark",
    type: "shell",
    privileged: true,
    keep_color: true,
    args: [ spark_basedir, vm_username ],
    inline: <<-SHELL
     KERNEL_NAME='pyspark'
     USERNAME=$2
     KDIR=/home/$USERNAME/.local/share/jupyter/kernels
     KERNEL_DIR="${KDIR}/${KERNEL_NAME}"
     KICONS=$1/kernel-icons

     # --------------------- Install the Pyspark kernel
     echo "Installing Pyspark kernel"
     su -l "$USERNAME" <<-EOF
mkdir -p "${KERNEL_DIR}"
cat <<KERNEL > "${KERNEL_DIR}/kernel.json"
{
    "display_name": "Pyspark (Py3)",
    "language_info": { "name": "python",
                       "codemirror_mode": { "name": "ipython", "version": 3 }
                     },
    "argv": [
	"/opt/ipnb/bin/pyspark-ipnb",
	"-m", "ipykernel",
	"-f", "{connection_file}"
    ],
    "env": {
        "SPARK_HOME": "$1/current"
    }
}
KERNEL
     # Copy Pyspark kernel logo
     cp -p $KICONS/pyspark-icon-64x64.png $KERNEL_DIR/logo-64x64.png
     cp -p $KICONS/pyspark-icon-32x32.png $KERNEL_DIR/logo-32x32.png
EOF
    SHELL


    vgrml.vm.provision "12.scala-kernel",
    type: "shell",
    privileged: true,
    keep_color: true,
    args: [ spark_basedir, vm_username ],
    inline: <<-SHELL
     SPARK_BASE=$1
     USERNAME=$2
     KERNEL_NAME='scala'
     KDIR="/home/$USERNAME/.local/share/jupyter/kernels"
     KERNEL_FILE="${KDIR}/${KERNEL_NAME}/kernel.json"
     su -l "$USERNAME" <<-EOF

       # --------------------- Install a Scala kernel
       echo "Installing a Scala kernel ..."
       echo " .. downloading coursier"
       curl -Lo coursier https://git.io/coursier-cli 2>k.log || cat k.log
       chmod +x coursier
       echo " .. installing almond kernel"
       ./coursier launch --fork almond --scala 2.12.11 --main-class almond.ScalaKernel -- --install --force 2>k.log || cat k.log
       rm -f coursier k.log
       echo " .. configuring almond kernel"
       /opt/ipnb/bin/python <<PYTH
import json
with open('${KERNEL_FILE}') as f:
   k = json.load(f)
k['display_name'] = 'Scala 2.12 (Almond)'
if 'env' not in k:
   k['env'] = {'SPARK_SUBMIT_OPTS': ''}
k['env']['SPARK_HOME'] = "${SPARK_BASE}/current"
k['env']['SPARK_CONF_DIR'] = "${SPARK_BASE}/current/conf"
k['env']['SPARK_SUBMIT_OPTS'] += " -Xms1024M -Xmx2048M -Dlog4j.logLevel=info"
with open('${KERNEL_FILE}','w') as f:
   json.dump(k, f, sort_keys=True)
PYTH
EOF
    SHELL

    # R Kernel
    vgrml.vm.provision "13.ir-kernel",
    type: "shell",
    privileged: true,
    keep_color: true,
    args: [ spark_basedir, vm_username ],
    inline: <<-SHELL
     USERNAME=$2
     KDIR=/home/$USERNAME/.local/share/jupyter/kernels

     # --------------------- Install the IRkernel
     echo "Installing IRkernel ..."
     su -l "$USERNAME" <<EOF
        PATH=/opt/ipnb/bin:$PATH Rscript -e 'IRkernel::installspec()'
        # Add the SPARK_HOME env variable to R
        echo "SPARK_HOME=$1/current" >> /home/$USERNAME/.Renviron
EOF
     # Add the SPARK_HOME env variable to the kernel.json file
     #KERNEL_JSON="${KDIR}/ir/kernel.json"
     #ENVLINE='  "env": { "SPARK_HOME": "'$1'/current" },'
     #POS=$(sed -n '/"argv"/=' $KERNEL_JSON)
     #sed -i "${POS}i $ENVLINE" $KERNEL_JSON
    SHELL

    vgrml.vm.provision "14.gfversion",
    type: "shell",
    run: "never",
    privileged: false,
    keep_color: true,
    args: [ spark_basedir ],
    inline: <<-SHELL
      cd $1/current/conf
      for f in spark-defaults.conf spark-env.sh
      do
        sed -i -e 's/0\.8\.3-spark3\.5/0.8.3-spark3.5/' $f.local.graphframes
      done
    SHELL


    # Update spark config
    vgrml.vm.provision "15.sparkconf",
    type: "shell",
    privileged: true,
    keep_color: true,
    args: [ spark_basedir ],
    inline: <<-SHELL
     cd $1/current/conf
     # Log4j
     mv log4j.properties orig/log4j.properties.old
     cp -p orig/log4j2.properties.template log4j2.properties
     sed -i 's/rootLogger.level = info/rootLogger.level = warn/' log4j2.properties
     # Make the Hive configuration available
     ln -s hadoop/hive-site.xml .
    SHELL

    # .........................................
    # Create a configuration file for sparklyr/Rstudio
    vgrml.vm.provision "20.Rconf",
    type: "shell",
    privileged: true,
    keep_color: true,
    args: [ vm_username ],
    inline: <<-SHELL
      CFG=/usr/local/lib/R/site-library/sparklyr/conf/config-template.yml
      mv $CFG ${CFG}.orig
      # Create a config.yml file for R
      cat <<'ENDFILE' > $CFG
# R configuration options for Spark
default:
  spark.master: "local"
  sparklyr.sparkui.url: "http://localhost:4040"
  rstudio.spark.connections: "local"
ENDFILE
    SHELL

    vgrml.vm.provision "30.icon",
    type: "shell",
    privileged: true,
    keep_color: true,
    args: [ vm_username ],
    inline: <<-SHELL
      USERNAME=$1
      su -l "$USERNAME" <<-'EOF'
# --------------------- Install the notebook extensions
#echo "Installing notebook extensions"
#for ext in toc2/main toggle-headers search-replace python-markdown
#for ext in toc2/main
#do
#        /opt/ipnb/bin/jupyter nbextension enable $ext
#done

# --------------------- Put the custom Jupyter icon in place
cd /opt/ipnb/lib/python?.*/site-packages
B=$PWD
for d in jupyter_server/static jupyter_server/static/favicons
do
        cd $B/$d
        test -f favicon.ico && mv favicon.ico favicon-orig.ico
        ln -s $B/notebook/static/base/images/favicon-custom.ico favicon.ico
done
EOF
     SHELL

    # .........................................
    # Install the Notebook startup script & configure it
    # Configure Spark execution mode & remote access if defined
    vgrml.vm.provision "31.nbconf",
    type: "shell",
    privileged: true,
    keep_color: true,
    args: [ vm_username,
            spark_mode, spark_master, spark_namenode, spark_history_server ],
    inline: <<-SHELL
     # Link the IPython mgr script so that it can be found by root
     SCR=jupyter-notebook-mgr
     chmod 775 /opt/ipnb/bin/$SCR
     rm -f /usr/sbin/$SCR
     ln -s /opt/ipnb/bin/$SCR /usr/sbin

     # Create the system config for IPython notebook
     CFGD=/etc/sysconfig
     test -d ${CFGD} || { CFGD=/etc/jupyter; mkdir $CFGD; }
     cat <<-EOF > $CFGD/jupyter-notebook
NOTEBOOK_USER="$1"
NOTEBOOK_SCRIPT="/opt/ipnb/bin/jupyter-notebook"
NOTEBOOK_BASEDIR="/home/$1/IPNB"
EOF

     # Configure remote addresses
     if [ "$3" ]; then
       echo "Configuring Spark server [$3]"
       jupyter-notebook-mgr set-addr yarn "$3" "$4" "$5"
       jupyter-notebook-mgr set-addr standalone "$3" "$4" "$5"
     fi

     # Set the name of the initially active config
     echo "Configuring Spark mode [$2]"
     jupyter-notebook-mgr set-mode "$2"

     # Enable & start the service
     systemctl enable notebook
     echo "Starting notebook service"
     systemctl start notebook
  SHELL

    # *************************************************************************
    # Optional provisioning
    # These need to be run explicitly

    # .........................................
    # Modify Spark configuration to add or remove GraphFrames
    vgrml.vm.provision "graphframes",
      type: "shell",
      run: "never",
      privileged: true,
      keep_color: true,
      args: [ spark_basedir ],
      inline: <<-SHELL
        cd $1/current/conf
        NAME=spark-defaults.conf
        if [ "$(readlink $NAME)" = "opt/${NAME}.local" ]
        then
           echo "activating GraphFrames"
           ln -sf opt/${NAME}.graphframes $NAME
           ln -sf opt/spark-env.sh.graphframes spark-env.sh
        elif [ "$(readlink $NAME)" = "opt/${NAME}.graphframes" ]
        then
           echo "deactivating GraphFrames"
           ln -sf opt/${NAME}.local ${NAME}
           ln -sf opt/spark-env.sh.local spark-env.sh
        else
           echo "No local configuration active"
        fi
      SHELL

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
        PKG=rstudio-server-2024.12.0-467-amd64.deb
        wget --no-verbose https://download2.rstudio.org/server/jammy/amd64/$PKG
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
      privileged: false,
      keep_color: true,
      args: [ vm_username ],
      inline: <<-SHELL
          echo "Installing nbconvert requirements"
          sudo apt-get update && sudo apt-get install -y --no-install-recommends pandoc texlive-latex-recommended texlive-plain-generic texlive-xetex texlive-fonts-recommended lmodern inkscape
          pip install nb-pdf-template
          # We modify the LaTeX template to generate A4 pages
          # (comment this out to keep Letter-sized pages)
          sudo -u vagrant perl -pi -e 's|(\\\\geometry\\{)|${1}a4paper,|' /opt/ipnb/share/jupyter/nbconvert/templates/latex/base.tex.j2
          # Use an improved template
          LINE="c.LatexExporter.template_name = 'latex_authentic'"
          echo $LINE | sudo -u $1 tee -a /home/$1/.jupyter/jupyter_notebook_config.py
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
          perl -pi -e 's|(\\\\usepackage\\{eurosym}.*)|${1}\n    \\\\usepackage{polyglossia}\\\\setmainlanguage{'$LANGUAGE'}|' /opt/ipnb/share/jupyter/nbconvert/templates/latex/base.tex.j2
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
        VERSION=3.9.9
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
        echo "deb https://repo.scala-sbt.org/scalasbt/debian all main" | sudo tee /etc/apt/sources.list.d/sbt.list
        echo "deb https://repo.scala-sbt.org/scalasbt/debian /" | sudo tee /etc/apt/sources.list.d/sbt_old.list
        curl -sL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2EE0EA64E40A89B84B2DF73499E82A75642AC823" | sudo tee /etc/apt/trusted.gpg.d/sbt.asc
        sudo apt-get update
        sudo apt-get install sbt
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
         chown $1:$1 /home/$1/.emacs
      SHELL

    # .........................................
    # Install some Deep Learning frameworks
    vgrml.vm.provision "dl",
      type: "shell",
      run: "never",
      privileged: false,
      keep_color: true,
      inline: <<-SHELL
         pip install --upgrade "tensorflow-cpu==2.18"
         pip install --upgrade torch==2.6.0 torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu
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
