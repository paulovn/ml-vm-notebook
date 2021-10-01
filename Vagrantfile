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
vm_cpus = '1'

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
spark_master = 'localhost'
# [B] The host running the HDFS namenode
spark_namenode = 'localhost'
# [C] The location (host:port) of the Spark History Server
spark_history_server = 'localhost:18080'
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

  config.vm.define "vm-spark-nb64" do |vgrml|

    #config.name = "vgr-pyspark"

    # The base box we are using. As fetched from ATLAS
    vgrml.vm.box = "paulovn/spark-base64"
    vgrml.vm.box_version = "= 3.1.2"

    # Alternative place: a local box
    #vgrml.vm.box_url = "file:///almacen/VM/VagrantBox/spark-base64-LOCAL.json"

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
    # -- we need it to be internally called "localhost" so that some packages
    # (such as ElasticSearch) can connect to local services
    vgrml.vm.hostname = "localhost"
    vgrml.vm.provider :virtualbox do |vb|
      # Set the hostname in VirtualBox
      vb.name = "vgr-ipnb-spark"
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

    # Spark driver UI
    vgrml.vm.network :forwarded_port, host: 4040, guest: 4040,
     auto_correct: true
    # Spark driver UI for the 2nd application (e.g. a command-line job)
    vgrml.vm.network :forwarded_port, host: 4041, guest: 4041,
     auto_correct: true

    # Hadoop HDFS namenode, webui
    vgrml.vm.network :forwarded_port, host: 9870, guest: 9870,
     auto_correct: true
    # Hadoop HDFS datanodes
    vgrml.vm.network :forwarded_port, host: 50011, guest: 50011,
     auto_correct: true
    vgrml.vm.network :forwarded_port, host: 50012, guest: 50012,
     auto_correct: true
    # Hadoop HDFS datanodes, webui
    vgrml.vm.network :forwarded_port, host: 50081, guest: 50081,
     auto_correct: true
    vgrml.vm.network :forwarded_port, host: 50082, guest: 50082,
     auto_correct: true
    # Hadoop Yarn resource manager, webui
    vgrml.vm.network :forwarded_port, host: 8088, guest: 8088,
     auto_correct: true
    # Hadoop Yarn resource manager, history daemon
    vgrml.vm.network :forwarded_port, host: 19888, guest: 19888,
     auto_correct: true

    # Kibana port
    vgrml.vm.network :forwarded_port, host: 5601, guest: 5601,
     auto_correct: true

    # kafdrop port
    vgrml.vm.network :forwarded_port, host: 9000, guest: 9000,
     auto_correct: true

    # RStudio server
    # =====> if using RStudio, uncomment the following line and reload the VM
    #vgrml.vm.network :forwarded_port, host: 8787, guest: 8787


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
    # Create the user to run Spark jobs (esp. notebook processes)
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
# Python executable to use in a Pyspark driver (used outside notebooks)
export PYSPARK_DRIVER_PYTHON=ipython
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
    # Create the IPython Notebook profile ready to run Spark jobs
    # and install all kernels: Pyspark, Scala, IRKernel, and extensions
    # Prepared for IPython >=4 (so that we configure as a Jupyter app)
    vgrml.vm.provision "10.config",
    type: "shell",
    privileged: true,
    keep_color: true,
    args: [ vm_username, vm_password, port_nb_internal, spark_basedir ],
    inline: <<-SHELL
     USERNAME=$1
     NOTEBOOK_BASEDIR=/home/$USERNAME/IPNB
     PASS=$(/opt/ipnb/bin/python -c "from IPython.lib import passwd; print(passwd('$2'))")

     # --------------------- Create the Jupyter config
     echo "Creating Jupyter config"
     cat <<-EOF > /home/$USERNAME/.jupyter/jupyter_notebook_config.py
import os
c.NotebookApp.ip = '0.0.0.0'
c.NotebookApp.port = $3
c.NotebookApp.password = '$PASS'
c.NotebookApp.notebook_dir = os.environ.get('NOTEBOOK_BASEDIR','$NOTEBOOK_BASEDIR')
c.NotebookApp.open_browser = False
c.NotebookApp.log_level = 'INFO'
EOF
     chown $USERNAME.$USERNAME /home/$USERNAME/.jupyter/jupyter_notebook_config.py

     # --- Set the notebook service to start
     systemctl enable notebook

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

    # .........................................
    # Create a configuration file for sparklyr/Rstudio
    vgrml.vm.provision "20.Rconfig",
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
    # Configure Spark execution mode & remote access if defined
    vgrml.vm.provision "31.nbconfig",
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
     # note we do not enable the service -- we'll explicitly start it at the end

     # Create the config for IPython notebook
     CFGD=/etc/sysconfig
     test -d ${CFGD} || { CFGD=/etc/jupyter; mkdir $CFGD; }
     cat <<-EOF > $CFGD/jupyter-notebook
NOTEBOOK_USER="$1"
NOTEBOOK_SCRIPT="/opt/ipnb/bin/jupyter-notebook"
NOTEBOOK_BASEDIR="/home/$1/IPNB"
EOF

     # Configure remote addresses
     if [ "$3" ]; then
       jupyter-notebook-mgr set-addr yarn "$3" "$4" "$5"
       jupyter-notebook-mgr set-addr standalone "$3" "$4" "$5"
     fi

     # Set the name of the initially active config
     echo "Configuring Spark mode as: $2"
     jupyter-notebook-mgr set-mode "$2"

     # Start the service
     systemctl start notebook
  SHELL

    # *************************************************************************
    # Optional provisioning
    # These need to be run explicitly

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
          apt-get update && apt-get install -y --no-install-recommends pandoc texlive-xetex texlive-generic-recommended texlive-fonts-recommended lmodern
          # We modify the LaTeX template to generate A4 pages
          # (comment this out to keep Letter-sized pages)
          perl -pi -e 's|(\\\\geometry\\{)|${1}a4paper,|' /opt/ipnb/lib/python?.?/site-packages/nbconvert/templates/latex/base.tplx
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
          CODE=es
          echo "** Adding support for $LANGUAGE to LaTeX"
          # https://tex.stackexchange.com/questions/345632/f25-texlive2016-no-hyphenation-patterns-were-preloaded-for-the-language-russian
          sudo apt-get install -y texlive-lang-spanish
          LANGDAT=$(kpsewhich language.dat)
          sudo bash -c "echo -e '\n$LANGUAGE hyph-${CODE}.tex\n=use$LANGUAGE' >> $LANGDAT" && sudo fmtutil-sys --all
          echo "** Converting base LaTeX template for $LANGUAGE"
          perl -pi -e 's(\\\\usepackage\\[T1\\]\\{fontenc})(\\\\usepackage{polyglossia}\\\\setmainlanguage{'$LANGUAGE'});' -e 's#\\\\usepackage\\[utf8x\\]\\{inputenc}#%--removed--#;' /opt/ipnb/lib/python?.?/site-packages/nbconvert/templates/latex/base.tplx
      SHELL

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
        if [ "$(readlink $NAME)" = "${NAME}.local" ]
        then
           echo "activating GraphFrames"
           ln -sf ${NAME}.local.graphframes $NAME
           ln -sf spark-env.sh.local.graphframes spark-env.sh
        elif [ "$(readlink $NAME)" = "${NAME}.local.graphframes" ]
        then
           echo "deactivating GraphFrames"
           ln -sf ${NAME}.local ${NAME}
           ln -sf spark-env.sh.local spark-env.sh
        else
           echo "No local configuration active"
        fi
      SHELL

    vgrml.vm.provision "hadoop",
      type: "shell",
      run: "never",
      privileged: true,
      keep_color: true,
      args: [ vm_username ],
      inline: <<-SHELL
        BASE=/opt/hadoop
        test -d $BASE || mkdir -p $BASE
        cd $BASE
        VERSION=3.2.2
        echo "Downloading Hadoop $VERSION ..."
        T=hadoop-$VERSION
        rm -rf $T
        wget --progress=dot:giga https://archive.apache.org/dist/hadoop/core/$T/$T.tar.gz
        echo "Extracting Hadoop $VERSION ..."
        tar zxvf $T.tar.gz $T/bin $T/etc $T/lib/native $T/libexec $T/sbin $T/share/hadoop --exclude jdiff --exclude sources
        rm $T.tar.gz
        PRF=/home/$1/.bash_profile
        if ! grep -q $T $PRF; then
          echo "export PATH=\\$PATH:$BASE/$T/bin" >> $PRF
        fi
        echo "Customizing Hadoop $VERSION ..."
        cat <<-EOF > /etc/hadoop/hadoop-env.sh
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export HADOOP_CONF_DIR=/etc/hadoop
HADOOP_LOG_DIR=/var/log/hadoop
EOF
        # Download a custom VM configuration
        cd /; wget -O - https://tiny.cc/vm-hadoop-conf | tar zxv
        # Install it into the downloaded hadoop
        cd $BASE/$T; mv etc/hadoop etc/hadoop.orig; ln -s /etc/hadoop etc
        # Prepare folders
        for d in /var/log/hadoop /data/hdfs /data/yarn
        do
           mkdir -p $d; chown $1.$1 $d
        done
        # Ensure vmuser can do ssh to localhost
        D=/home/$1/.ssh
        if test ! -f $D/id_rsa.pub; then
          su -l "$1" <<USEREOF
ssh-keygen  -f $D/id_rsa -N ""
cat $D/id_rsa.pub >> $D/authorized_keys
USEREOF
        fi

      SHELL

    vgrml.vm.provision "elk",
      type: "shell",
      run: "never",
      privileged: true,
      keep_color: true,
      args: [ vm_username ],
      inline: <<-SHELL
        /bin/echo -e "\n..... Installing ELK stack"
        SRV=artifacts.elastic.co
        wget -qO - https://$SRV/GPG-KEY-elasticsearch | apt-key add -
        test -f /etc/apt/sources.list.d/elastic-7.x.list || \
	      echo "deb https://$SRV/packages/7.x/apt stable main" | \
	      sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list

        apt-get update
        apt-get install -y elasticsearch
        apt-get install -y kibana logstash
        mkdir -p /var/log/kibana && chown kibana.kibana /var/log/kibana
        #mkdir -p "$ELDIR" && chown elasticsearch.elasticsearch ${ELDIR}

        wget http://tiny.cc/cfgfilemgr -O /opt/ipnb/bin/app-config-manager.py
        MAKECONF='/opt/ipnb/bin/python /opt/ipnb/bin/app-config-manager.py --rewrite'

        ${MAKECONF} --sep ': ' /etc/kibana/kibana.yml \
		'logging.dest: "/var/log/kibana/kibana.log"' \
		'console.enabled: false' \
                'server.host: 0.0.0.0'

        ${MAKECONF} --sep ': ' /etc/elasticsearch/elasticsearch.yml \
		'indices.fielddata.cache.size: 20%' \
		'cluster.name: vm-example' \
		'bootstrap.memory_lock: true' \
                'xpack.security.enabled: false'

        ${MAKECONF} --sep = /etc/default/elasticsearch \
		"ES_HEAP_SIZE=${ES_HEAP:-512m}"

        cat <<EOF >/etc/elasticsearch/jvm.options.d/heap.options
-Xms${ES_HEAP:-256m}
-Xmx${ES_HEAP:-512m}
EOF

       # This is applicable if the process is running inside a VM and the data
       # folder is a mounted folder
       id -u vagrant >/dev/null 2>&1 && usermod logstash -G vagrant -a

       su -l "vagrant" -c "pip install elasticsearch elasticsearch-dsl"
      SHELL

    vgrml.vm.provision "kafka",
      type: "shell",
      run: "never",
      privileged: true,
      keep_color: true,
      args: [ vm_username ],
      inline: <<-SHELL
        KAFKA_VERSION=2.8.1
        KAFKA_BASE=/opt/kafka
        KAFKA_ADDR=localhost:9092
        ZK_PORT=2181
        /bin/echo -e "\n..... Installing Kafka $KAFKA_VERSION"
        TARFILE=kafka_2.12-$KAFKA_VERSION.tgz
        mkdir -p $KAFKA_BASE; cd $KAFKA_BASE
        rm -rf $TARFILE kafka_2.12-$KAFKA_VERSION current kafka-services
        wget --progress=dot:giga https://archive.apache.org/dist/kafka/$KAFKA_VERSION/$TARFILE || exit 1
        tar zxvf $TARFILE
        ln -s kafka_2.12-$KAFKA_VERSION/ current
        rm -f $TARFILE
        /bin/echo -e "\n..... Configuring Kafka"
        cd current
        mkdir -p -m 777 var/data var/logs var/run
        KAFKADIR=${KAFKA_BASE}/current
        DATADIR=${KAFKADIR}/var/data
        C=config/server.properties
        test -f $C.orig || cp -p $C $C.orig
        sed -i \
            -e "s@\#advertised.listeners=.*@advertised.listeners=PLAINTEXT://$KAFKA_ADDR@"\
            -e "s@\#advertised.listeners=.*@advertised.listeners=PLAINTEXT://$KAFKA_ADDR@"\
            -e "s@zookeeper.connect=.*@zookeeper.connect=localhost:$ZK_PORT@" \
            -e "s@log.dirs=/tmp/kafka-logs@log.dirs=$DATADIR/kafka@
                \$ a 
                \$ a auto.create.topics.enable=true" $C
        echo "..... Configuring Zookeeper"
        Z=config/zookeeper.properties
        test -f $Z.orig || cp -p $Z $Z.orig
        sed -i \
            -e "s@dataDir=/tmp/zookeeper@dataDir=$DATADIR/zookeeper@" \
            -e "s@clientPort=.*@clientPort=$ZK_PORT@" $Z
        echo "..... Download management scripts"
        cd bin
        wget --progress=dot:giga -O - https://tiny.cc/kafka-services | tar zxv
      SHELL

    vgrml.vm.provision "kafdrop",
      type: "shell",
      run: "never",
      privileged: true,
      keep_color: true,
      args: [ vm_username ],
      inline: <<-SHELL
        DIR=/opt/kafka
        cd $DIR || exit 1
        wget --progress=dot:giga -O - https://tiny.cc/kafdrop | tar zxv
        echo "Kafdrop installed. Use $DIR/kafdrop/bin/kafdrop (start|stop)"
      SHELL

    vgrml.vm.provision "nosql",
      type: "shell",
      run: "never",
      privileged: true,
      args: [ vm_username, spark_basedir ],
      inline: <<-SHELL
        /bin/echo -e "\n..... Installing NOSQL stack"
        wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add -
        echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list
        add-apt-repository ppa:redislabs/redis
        apt-get update
        apt-get install -y mongodb-org redis
        chmod go+rx /etc/redis
        chmod 644 /etc/redis/redis.conf
        su -l "vagrant" -c "pip install pymongo redis"
        SPARK_CFG="$2/current/conf/spark-defaults.conf"
        grep -q "# NoSQL" $SPARK_CFG || cat <<EOF >>$SPARK_CFG

# -------------------------------------------------------------------------
# NoSQL sources

# MongoDB
##spark.jars.packages=org.mongodb.spark:mongo-spark-connector_2.12:3.0.1
# Redis
##spark.jars.packages=com.redislabs:spark-redis_2.12:2.6.0
EOF
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
        PKG=rstudio-server-1.4.1106-amd64.deb
        wget --no-verbose https://download2.rstudio.org/server/xenial/amd64/$PKG
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
         pip install --upgrade pip
         pip install --upgrade tensorflow-cpu
         pip install --upgrade torch torchvision
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
