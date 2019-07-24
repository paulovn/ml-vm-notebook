# -*- mode: ruby;  ruby-indent-tabs-mode: t -*-
# vi: set ft=ruby :
# **************************************************************************
# Provision a VM for Machine Learning/NLP tasks
# **************************************************************************

# --------------------------------------------------------------------------
# Runtime variables defining VM behaviour
# (used every time the VM starts)

# Launch in graphical mode: true/false
vm_gui = false

# RAM memory used for the VM, in MB
vm_memory = '2048'

# Number of CPU cores assigned to the VM
vm_cpus = '1'

# The virtual machine exports the port where the notebook process by
# forwarding it to this port of the local machine
# So to access the notebook server, you point to http://localhost:<port>
port_nb = 8008

# Note there is an additional port exported: the Spark UI driver is
# forwarded to port 4040

# --------------------------------------------------------------------------
# Configuration variables used at VM provision

# Username that will run all processes.
vm_username = 'vmuser'

# Password to use to access the Notebook web interface 
vm_password = 'vmuser'

# This defines the Spark notebook processing mode. There are three choices
# available: "local", "yarn", "standalone"
# It can be changed at runtime by executing inside the virtual machine, as 
# root user, "service notebook set-mode <mode>"
spark_mode = 'local'


# --------------------------------------------------------------------------
# Variables defining the Spark installation in the base box. 
# Don't change these

# The place where Spark is deployed inside the local machine
spark_basedir = '/opt/spark'


# --------------------------------------------------------------------------
# Some variables that affect Vagrant execution

# Check the command requested -- if ssh we'll change the login user
vagrant_command = ARGV[0]

# Conditionally activate some provision sections
provision_run_rs  = ENV['PROVISION_RSTUDIO'] == '1' || \
        (vagrant_command == 'provision' && ARGV.include?('rstudio'))
provision_run_nbc = (ENV['PROVISION_NBC'] == '1') || \
        (vagrant_command == 'provision' && \
           (ARGV.include?('nbc')||ARGV.include?('nbc.es')))
provision_run_nlp  = ENV['PROVISION_NLP'] == '1' || \
        (vagrant_command == 'provision' && ARGV.include?('nlp'))
provision_run_dl  = ENV['PROVISION_DL'] == '1' || \
        (vagrant_command == 'provision' && ARGV.include?('dl'))
provision_run_mvn = ENV['PROVISION_MVN'] == '1' || \
        (vagrant_command == 'provision' && ARGV.include?('mvn'))

provision_run_nlp = true
provision_run_dl = true
provision_run_desktop = true
provision_run_desktop_full = false



# --------------------------------------------------------------------------
# Vagrant configuration

port_nb_internal = 8008

# The "2" in Vagrant.configure sets the configuration version
Vagrant.configure(2) do |config|

  # This is to avoid Vagrant inserting a new SSH key, instead of the
  # default one (perhaps because the box will be later packaged)
  #config.ssh.insert_key = false

  # set auto_update to false, if you do NOT want to check the correct 
  # additions version when booting this machine
  config.vbguest.auto_update = false

  # Use our custom username, instead of the default "vagrant"
  if vagrant_command == "ssh"
      config.ssh.username = vm_username
  end
  #config.ssh.username = "vagrant"

  config.vm.box_download_insecure = true

  config.vm.define "vm-nlp-nb64" do |vgrml|

    #config.name = "vgr-pyspark"

    # The base box we are using. As fetched from ATLAS
    vgrml.vm.box = "paulovn/spark-base64"
    vgrml.vm.box_version = "= 2.2.1"

    # Alternative place: UAM internal
    #vgrml.vm.box = "uam/spark-base64"
    #vgrml.vm.box_url = "http://svrbigdata.ii.uam.es/vm/uam-spark-base64.json"
    # Alternative place: TID internal
    #vgrml.vm.box = "tid/spark-base64"
    # Alternative place: local box
    #vgrml.vm.box_url = "file:///almacen/VM/VagrantBox/spark-base64-LOCAL.json"
    #vgrml.vm.box_url = "http://artifactory.hi.inet/artifactory/vagrant-machinelearning/tid-spark-base64.json"

    # Disable automatic box update checking. If you disable this, then
    # boxes will only be checked for updates when the user runs
    # `vagrant box outdated`. This is not recommended.
    # vgrml.vm.box_check_update = false

    # Deactivate the usual synced folder and use instead a local subdirectory
    vgrml.vm.synced_folder ".", "/vagrant", disabled: true
    vgrml.vm.synced_folder "vmfiles", "/vagrant",
      mount_options: ["dmode=775","fmode=664"],
      disabled: false
    #owner: vm_username
    #auto_mount: false
  
    # Customize the virtual machine: set hostname & allocated RAM
    vgrml.vm.hostname = "vgr-nlp"
    vgrml.vm.provider :virtualbox do |vb|
      # Set the hostname in VirtualBox
      vb.name = vgrml.vm.hostname.to_s
      # Customize the amount of memory on the VM
      vb.memory = vm_memory
      # Set the number of CPUs
      vb.cpus = vm_cpus
      vb.customize [ "guestproperty", "set", :id,
                     "/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold",
                     10000 ]
      vb.customize [ "guestproperty", "set", :id,
                     "/VirtualBox/GuestAdd/VBoxService/--timesync-set-on-restore",
                     1 ]

      # Display the VirtualBox GUI when booting the machine
      vb.gui = vm_gui
      vb.customize ["modifyvm", :id, "--clipboard", "bidirectional"]
      vb.customize ["modifyvm", :id, "--draganddrop", "bidirectional"]      
    end

    # vagrant-vbguest plugin: set auto_update to false, if you do NOT want to
    # check the correct additions version when booting this machine
    #vgrml.vbguest.auto_update = false

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
    # =====> uncomment if using RStudio
    vgrml.vm.network :forwarded_port, host: 8787, guest: 8787

    # Quiver
    # =====> uncomment if using Quiver visualization for Keras
    #vgrml.vm.network :forwarded_port, host: 5000, guest: 5000

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


    vgrml.vm.post_up_message = "**** The Vagrant NLP machine is up. Connect to http://localhost:" + port_nb.to_s + " for notebook access"


    # **********************************************************************
    # Provisioning: install software, configuration files and startup scripts

    # .........................................
    # Base system configuration
    vgrml.vm.provision "01.system",
    type: "shell", 
    privileged: true,
    inline: <<-SHELL      
      echo "System preparation ..."
      echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections
      # Update base packages
      apt-get update
      apt-get upgrade -y

      # Set locale
      apt-get install -y locales
      locale-gen es_ES.UTF-8 en_US.UTF-8

      # Set timezone
      timedatectl set-timezone Europe/Madrid

      # Set keyboard
      cat > /etc/default/keyboard <<-EOF
XKBMODEL=pc105
XKBLAYOUT=es
XKBOPTIONS=terminate:ctrl_alt_bksp
BACKSPACE=guess
EOF

      # Install dev tools
      apt-get install -y software-properties-common
      apt-get install -y build-essential
    SHELL

    # .........................................
    # Create the user to run jobs (esp. notebook processes)
    vgrml.vm.provision "02.nbuser",
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
# Python executable to use in a Pyspark driver (used outside notebooks)
export PYSPARK_DRIVER_PYTHON=ipython
# Place where to keep user R packages (used outside RStudio Server)
export R_LIBS_USER=~/.Rlibrary
# Load Theano initialization file
export THEANORC=/etc/theanorc:~/.theanorc
# Jupyter uses this to define datadir but it is undefined when using "runuser"
test "$XDG_RUNTIME_DIR" || export XDG_RUNTIME_DIR=/run/user/$(id -u)
ENDPROFILE
      chown $1.$1 /home/$1/.bash_profile

      # Create some local files as the designated user, plus update bashrc
      su -l "$1" <<'USEREOF'
for d in bin tmp .ssh .jupyter .Rlibrary; do test -d $d || mkdir $d; done
chmod 700 .ssh
PYVER=$(ls -d /opt/ipnb/lib/python?.? | xargs -n1 basename)
rm -f bin/{python,$PYVER.7,pip,ipython,jupyter}
ln -s /opt/ipnb/bin/{python,$PYVER,pip,ipython,jupyter} bin
test -h IPNB || { rm -f IPNB; ln -s /vagrant/IPNB/ IPNB; }
echo 'alias dir="ls -al"' >> ~/.bashrc
echo 'PS1="\\h#\\# \\W> "'   >> ~/.bashrc
echo 'export NO_AT_BRIDGE=1' >> ~/.bashrc
USEREOF

      # Install the vagrant public key so that we can ssh to this account
      cp -p /home/vagrant/.ssh/authorized_keys /home/$1/.ssh/authorized_keys
      chown $1.$1 /home/$1/.ssh/authorized_keys
      
    SHELL

    # Mount the shared folder with the new created user, so that it can write
    # ---> don't, instead we add the user to the vagrant group and mount the 
    #      shared folder with group permissions
#    vgrml.vm.provision "02.mount",
#    type: "shell",
#    privileged: true,
#    keep_color: true,
#    args: [ vm_username ],
#    inline: <<-SHELL
#umount /vagrant
#mount -t vboxsf -o uid=$(id -u $1),gid=$(id -g $1) vagrant /vagrant
#SHELL

    # .........................................
    # Install graphical desktop
    if (provision_run_desktop)
      vgrml.vm.provision "05.desktop",
      type: "shell", 
      privileged: true,
      inline: <<-SHELL
        echo "Installing basic desktop environment ..."
        # Lightweight desktop (minimal MATE environment)
        apt-get install -y --no-install-recommends ubuntu-mate-desktop mate-terminal mate-applet-brisk-menu lightdm lightdm-gtk-greeter gedit mozo 
        apt-get install -y virtualbox-guest-dkms virtualbox-guest-utils virtualbox-guest-x11
        # Reduce grub menu wait time
        echo GRUB_RECORDFAIL_TIMEOUT=5 >> /etc/default/grub; update-grub
        # Deactivate automatic start of Display Manager
        systemctl disable lightdm
      SHELL
      
      vgrml.vm.provision "05.desktop.theme",
      type: "shell", 
      privileged: true,
      args: [ vm_username ],
      inline: <<-SHELL
        # Do some theming
        echo -e '[User]\nIcon=/usr/share/icons/mate/48x48/emotes/face-cool.png' >> /var/lib/AccountsService/users/$1
        cat <<LIGHTDM >/etc/lightdm/lightdm-gtk-greeter.conf
[greeter]
background=/usr/share/backgrounds/cosmos/earth-horizon.jpg
user-background=true
LIGHTDM
        cat <<SCHEMA >/usr/share/glib-2.0/schemas/90_desktop.gschema.override
[org.mate.interface]
icon-theme='mate'
gtk-theme='Blue-Submarine'

[org.mate.Marco.general]
theme='Blue-Submarine'

[com.solus-project.brisk-menu]
favourites=['mate-terminal.desktop', 'caja-browser.desktop', 'gedit.desktop']
SCHEMA
        glib-compile-schemas /usr/share/glib-2.0/schemas/

      SHELL
    end

    if (provision_run_desktop_full)
      vgrml.vm.provision "05.desktop-full",
      type: "shell",
      privileged: true,
      inline: "apt-get install -y ubuntu-desktop"
    end


    # .........................................
    # Create the IPython Notebook profile ready to run Spark jobs
    # and install all kernels: Pyspark, SPylon (Scala), IRKernel, and extensions
    # Prepared for IPython >=4 (so that we configure as a Jupyter app)
    vgrml.vm.provision "10.config",
    type: "shell", 
    privileged: true,
    keep_color: true,    
    args: [ vm_username, vm_password, port_nb_internal, spark_basedir ],
    inline: <<-SHELL
     USERNAME=$1
     PASS=$(/opt/ipnb/bin/python -c "from IPython.lib import passwd; print(passwd('$2'))")

     # --------------------- Create the Jupyter config
     echo "Creating Jupyter config"
     cat <<-EOF > /home/$USERNAME/.jupyter/jupyter_notebook_config.py
c = get_config()
# define server
c.NotebookApp.ip = '0.0.0.0'
c.NotebookApp.port = $3
c.NotebookApp.password = u'$PASS'
c.NotebookApp.open_browser = False
c.NotebookApp.log_level = 'INFO'
c.NotebookApp.notebook_dir = u'/home/$USERNAME/IPNB'
# Preload matplotlib
c.IPKernelApp.matplotlib = 'inline'
# Kernel heartbeat interval in seconds.
# This is in jupyter_client.restarter. Not sure if it gets picked up
c.KernelRestarter.time_to_dead = 30.0
c.KernelRestarter.debug = True
EOF
     chown $USERNAME.$USERNAME /home/$USERNAME/.jupyter/jupyter_notebook_config.py
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


    vgrml.vm.provision "12.spylon",
    type: "shell",
    privileged: true,
    keep_color: true,
    args: [ spark_basedir, vm_username ],
    inline: <<-SHELL
     SPARK_BASE=$1
     KERNEL_NAME='spylon-kernel'
     USERNAME=$2
     KDIR=/home/$USERNAME/.local/share/jupyter/kernels
     KERNEL_DIR="${KDIR}/${KERNEL_NAME}"
     KICONS=$SPARK_BASE/kernel-icons

     # --------------------- Install the Scala Spark kernel (Spylon)
     echo "Installing Spylon (Scala) kernel ..."
     su -l "$USERNAME" <<-EOF
       PATH=/opt/ipnb/bin:$PATH python -m spylon_kernel install --user
       /opt/ipnb/bin/python <<PYTH
import json
import os.path
name = os.path.join('$KERNEL_DIR','kernel.json')
with open(name) as f:
   k = json.load(f)
k['display_name'] = 'Scala 2.11 (SPylon)'
k['env']['SPARK_HOME'] = '$SPARK_BASE/current'
k['env']['SPARK_SUBMIT_OPTS'] += ' -Xms1024M -Xmx2048M -Dlog4j.logLevel=info'
with open(name,'w') as f:
   json.dump(k, f, sort_keys=True)
PYTH
EOF
     # Copy Scala kernel logos
     cp -p $1/kernel-icons/scala-spark-icon-64x64.png "${KERNEL_DIR}/logo-64x64.png"
     cp -p $1/kernel-icons/scala-spark-icon-32x32.png "${KERNEL_DIR}/logo-32x32.png"
    SHELL

    vgrml.vm.provision "13.ir",
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
 
    vgrml.vm.provision "20.extensions",
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
    # Configure Spark execution mode & remote access if defined
    vgrml.vm.provision "21.nbconfig",
    type: "shell", 
    privileged: true,
    keep_color: true,    
    args: [ vm_username, spark_mode ],
    inline: <<-SHELL
     # Link the IPython mgr script so that it can be found by root
     SCR=jupyter-notebook-mgr
     chmod 775 /opt/ipnb/bin/$SCR
     rm -f /usr/sbin/$SCR
     ln -s /opt/ipnb/bin/$SCR /usr/sbin
     # note we do not enable the service -- we'll explicitly start it at the end

     # Create the config for IPython notebook
     CFGD=/etc/sysconfig/
     test -d ${CFGD} || { CFGD=/etc/jupyter; mkdir $CFGD; }
     cat <<-EOF > $CFGD/jupyter-notebook
NOTEBOOK_USER="$1"
NOTEBOOK_SCRIPT="/opt/ipnb/bin/jupyter-notebook"
EOF

     # Set the name of the initially active config
     echo "Configuring Spark mode as: $2"
     jupyter-notebook-mgr set-mode "$2"
  SHELL

    # *************************************************************************
    # Optional packages

    # .........................................
    # Install RStudio server
    # Do it only if explicitly requested (either by environment variable 
    # PROVISION_RSTUDIO when creating or by --provision-with rstudio) 
    # *** Don't forget to also uncomment forwarding for port 8787!
    if (provision_run_rs)
      vgrml.vm.provision "rstudio",
      type: "shell",
      keep_color: true,
      privileged: true,
      args: [ vm_username, vm_password ],
      inline: <<-SHELL
        echo "Downloading & installing RStudio Server"
        apt-get install -y gdebi-core
        # Download & install the package for RStudio Server
        PKG=rstudio-server-1.1.463-amd64.deb
        wget --no-verbose https://download2.rstudio.org/$PKG
        gdebi -n $PKG && rm -f $PKG
        # Define the directory for the user library, and the working directory
        CNF=/etc/rstudio/rsession.conf
        grep -q r-libs-user $CNF || cat >>$CNF <<EOF 
r-libs-user=~/.Rlibrary
session-default-working-dir=/vagrant/R
session-default-new-project-dir=/vagrant/R
EOF
        # Create a link to the host-mounted R subdirectory
        sudo -i -u "$1" bash -c "rm -f R; ln -s /vagrant/R/ R"
        # Send message
        echo "RStudio Server should be accessed at http://localhost:8787"
        echo "(if not, check in Vagrantfile that port 8787 has been forwarded)"
      SHELL
    end

    # .........................................
    # Install the necessary components for nbconvert to work.
    # Do it only if explicitly requested (either by environment variable 
    # PROVISION_NBC when creating or by --provision-with nbc)
    if (provision_run_nbc)
      vgrml.vm.provision "nbc",
      type: "shell",
      privileged: true,
      keep_color: true,
      args: [ vm_username ],
      inline: <<-SHELL
          echo "Installing nbconvert requirements"
          apt-get update && apt-get install -y --no-install-recommends pandoc texlive-xetex texlive-generic-recommended texlive-fonts-recommended lmodern
          # We modify the LaTeX template to generate A4 pages
          # (comment this out to keep Letter-sized pages)
          perl -pi -e 's|(\\\\geometry\\{)|${1}a4paper,|' /opt/ipnb/lib/python?.?/site-packages/nbconvert/templates/latex/base.tplx
      SHELL

      # .........................................
      # Optional: modify nbconvert to process Spanish documents
      vgrml.vm.provision "nbc.es",
      type: "shell",
      privileged: false,
      keep_color: true,
      inline: <<-SHELL
          # Define language
          LANGUAGE=spanish
          CODE=es
          echo "** Adding support for $LANGUAGE to LaTeX"
          # https://tex.stackexchange.com/questions/345632/f25-texlive2016-no-hyphenation-patterns-were-preloaded-for-the-language-russian
          sudo apt-get install -y texlive-lang-spanish
          LANGDAT=$(kpsewhich language.dat)
          sudo bash -c "echo -e '\n$LANGUAGE hyph-${CODE}.tex\n=use$LANGUAGE' >> $LANGDAT" && sudo fmtutil-sys --all
          echo "** Converting base LaTeX template for $LANGUAGE"
          perl -pi -e 's(\\\\usepackage\\[T1\\]\\{fontenc})(\\\\usepackage{polyglossia}\\\\setmainlanguage{'$LANGUAGE'});' -e 's#\\\\usepackage\\[utf8x\\]\\{inputenc}#%--removed--#;' /opt/ipnb/lib/python?.?/site-packages/nbconvert/templates/latex/base.tplx
      SHELL

    end

    # .........................................
    # Install additional packages for NLP
    # Do it only if explicitly requested (either by environment variable 
    # PROVISION_NLP when creating or by --provision-with nlp) 
    if (provision_run_nlp)
      vgrml.vm.provision "nlp.1",
      type: "shell",
      privileged: true,
      keep_color: true,
      args: [ vm_username ],
      inline: <<-SHELL
        echo "Installing NLP packages (I)"
        su -l "vagrant" -c "pip install nltk sklearn_crfsuite spacy"
      SHELL

      vgrml.vm.provision "nlp.2",
      type: "shell",
      privileged: true,
      keep_color: true,
      args: [ vm_username ],
      inline: <<-SHELL
        echo "Installing NLP packages (II) ..."
        echo "Installing Unitex"
        FILE=Unitex-GramLab-3.1-linux-x86_64.run
        wget http://releases.unitexgramlab.org/3.1/linux-x86_64/$FILE
        echo y | sh ./$FILE --nox11 --target /opt/Unitex-GramLab
        cd /opt/Unitex-GramLab/Src/C++/build
        make install
        # remove the assistive package (does not not exist in headless Java)
        sed -i -e '/^assistive_technologies=/s/^/#/' /etc/java-*-openjdk/accessibility.properties
        # Add Unitex dir to PATH
        sudo -u $1 sed -i -e '/export PATH=.*/s|$|:/opt/Unitex-GramLab/App|' /home/$1/.bash_profile

        echo "Installing GrapeNLP"
        add-apt-repository ppa:grapenlp/ppa
        apt-get update
        apt-get install -y grapenlp
      SHELL

      vgrml.vm.provision "nlp.3",
      type: "shell",
      privileged: true,
      keep_color: true,
      args: [ vm_username ],
      inline: <<-SHELL
        echo "Installing NLP packages (III) ..."
        # Install Unitex in MATE menu
        cat <<EOF >/usr/share/applications/unitex.desktop
[Desktop Entry]
Name=Unitex
Comment=Unitex Grammar Development
Exec=/opt//Unitex-GramLab/App/Unitex
Icon=/opt/Unitex-GramLab/App/Unitex.ico
Categories=Development;NLP;Java;
Terminal=false
Type=Application
Keywords=development;Java;grammar;NLP;
EOF
        # Add unitex to menu favorites
        F=/usr/share/glib-2.0/schemas/90_desktop.gschema.override
        PREV=$(grep '^favourites=' $F)
        NEW="${PREV%]*}, 'unitex.desktop']"
        sed -i -e "/^favourites=/s/.*/$NEW/" $F
        glib-compile-schemas /usr/share/glib-2.0/schemas/
      SHELL
      
    end


    # .........................................
    # Install Maven
    # Do it only if explicitly requested (either by environment variable 
    # PROVISION_MVN when creating or by --provision-with mvn) 
    if (provision_run_mvn)
      vgrml.vm.provision "mvn",
      type: "shell",
      privileged: true,
      keep_color: true,
      args: [ vm_username ],
      inline: <<-SHELL
        VERSION=3.5.4
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
    end


    # .........................................
    # Install some Deep Learning stuff
    # Do it only if explicitly requested (either by environment variable 
    # PROVISION_DL when creating or by --provision-with dl) 
    if (provision_run_dl)
      vgrml.vm.provision "dl",
      type: "shell",
      privileged: false,
      keep_color: true,
      inline: <<-SHELL
         echo "Installing DL packages"
         sudo apt-get install -y git
         pip install --upgrade setuptools tensorflow
         pip install --upgrade --no-deps git+git://github.com/Theano/Theano.git
         pip install --upgrade keras quiver
         pip install --upgrade torch torchvision
         sudo apt-get remove -y git 
       SHELL
    end


    # *************************************************************************

    # .........................................
    # Start Jupyter Notebook
    # Note: this one we run it every time the machine boots, since during 
    # the VM boot sequence the startup script is executed before vagrant has 
    # mounted the shared folder, and hence it fails. 
    # Running it as a provisioning makes it run *after* vagrant mounts, so 
    # this way it works.
    # [An alternative would be to force mounting on startup, by adding the
    # vboxsf mount point to /etc/fstab during provisioning]
    vgrml.vm.provision "50.nbstart", 
      type: "shell", 
      run: "always",
      privileged: true,
      keep_color: true,    
      inline: "systemctl start notebook"

    # .........................................
    # Start display manager, if we ar booting in graphical mode
    if (vm_gui)
      vgrml.vm.provision "60.gui",
      type: "shell",
      run: "always",
      privileged: true,
      inline: "systemctl start lightdm"
    end
    
  end # config.vm.define


end
