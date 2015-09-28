# -*- mode: ruby -*-
# vi: set ft=ruby :
# **************************************************************************
# Add specific configuration for running IPython notebook on a spark base VM
# **************************************************************************

vagrant_command = ARGV[0]


# --------------------------------------------------------------------------
# Variables defining the configuration of Spark & notebook 
# Modify as needed

# The virtual machine exports the port where the notebook process by forwarding
# it to this port of the local machine
# So to access the notebook server, you point to http://localhost:<port>
port_ipython = 8008

# Note there is an additional port exported: the Spark UI driver is forwarded 
# to port 4040

# This defines the Spark notebook processing mode: "local" or "yarn"
# It can be changed at runtime by executing inside the virtual machine, as 
# root user, "service spark-notebook mode <mode>"
spark_mode = 'local'

# Username that will run all spark processes.
# If remote (yarn) mode is going to be used, it is advisable to change it
# to a recognizable unique name, so that it is easily identify in cluster logs
spark_username = 'spark-vm'

# ---- These options are used only when running non-local tasks
# When in YARN mode, this defines the location of the Yarn Resource Manager
spark_yarn_master = 'samson01.hi.inet'
# The location of the Spark History Server
spark_history_server = 'samson03.hi.inet:18080'
# ----


# --------------------------------------------------------------------------
# Variables defining the Spark installation in the base box. 
# Don't change these

# The version of Spark we are using
spark_version = '1.5.0'
spark_name = 'spark-' + spark_version + '-bin-hadoop2.6'

# The place where Spark is deployed inside the local machine
spark_basedir = '/opt/spark'


# --------------------------------------------------------------------------
# Vagrant configuration

# The "2" in Vagrant.configure configures the configuration version
Vagrant.configure(2) do |config|

  if vagrant_command == "ssh"
      config.ssh.username = spark_username
  end

  config.vm.define "vgr-tid-sparknb" do |vgrspark|

    #config.name = "vgr-pyspark"

    # The base box we are using 
    vgrspark.vm.box = "tid-spark/base64"
    vgrspark.vm.box_url = "http://artifactory.hi.inet/artifactory/vagrant-machinelearning/tid-spark-base64.box"
    #vgrspark.vm.box_url = "file:///almacen/VM/VagrantBox/tid-spark-base64.json"

    # Disable automatic box update checking. If you disable this, then
    # boxes will only be checked for updates when the user runs
    # `vagrant box outdated`. This is not recommended.
    # vgrspark.vm.box_check_update = false

    # Deactivate the usual synced folder and use instead a local subdirectory
    vgrspark.vm.synced_folder ".", "/vagrant", disabled: true
    vgrspark.vm.synced_folder "vmfiles", "/vagrant", 
    disabled: false
    #owner: spark_username
    #auto_mount: false
  
    # Customize the virtual machine: set hostname & allocated RAM
    vgrspark.vm.hostname = "vgr-tid-spark-nb"
    vgrspark.vm.provider :virtualbox do |vb|
      # Set the hostname in VirtualBox
      vb.name = vgrspark.vm.hostname.to_s
      # Customize the amount of memory on the VM
      vb.memory = "1024"
      # Display the VirtualBox GUI when booting the machine
      #vb.gui = true
    end

    # Networking

    # Port forwarding
    vgrspark.vm.network :forwarded_port, 
    guest: port_ipython, 
    host: port_ipython                  # Notebook UI

    vgrspark.vm.network :forwarded_port, 
    host: 4040, 
    guest: 4040, 
    auto_correct: true                 # Spark driver UI

    # This enables the machine to be connected from outside; 
    # [it needs SPARK_LOCAL_IP to be set to the outside-visible interface ]
    #vgrspark.vm.network "public_network", type: "dhcp"
    # Declare a public network
    #vgrspark.vm.network "public_network", type: "dhcp", bridge: 'Realtek PCIe GBE Family Controller', :mac => "08002710A7ED"

    # Create a private network, which allows host-only access to the machine
    # using a specific IP.
    # vgrspark.vm.network "private_network", ip: "192.168.33.10"

    vgrspark.vm.post_up_message = "**** The Vagrant Spark-Notebook machine is up. Connect to http://localhost:" + port_ipython.to_s


    # **********************************************************************
    # Provisioning: install Spark configuration files and startup scripts

    # .........................................
    # Create the user to run Spark jobs (esp. notebook processes)
    vgrspark.vm.provision "01.nbuser",
    type: "shell", 
    privileged: true,
    args: [ spark_username ],
    inline: <<-SHELL
     id "$1" >/dev/null 2>&1 || useradd -c 'User for Spark Notebook' -m "$1"
     su -l "$1" <<-EOF
mkdir bin tmp .ssh 2>/dev/null
chmod 700 .ssh
rm -f bin/{python2.7,pip,ipython}
ln -s /opt/ipnb/bin/ext/{python2.7,pip,ipython} /home/$1/bin
test -d .jupyter || mkdir .jupyter
test -h IPNB || { rm -f IPNB; ln -s /vagrant/IPNB/ IPNB; }
EOF
     # Install the vagrant public key so that we can ssh to this account
     cp -p /home/vagrant/.ssh/authorized_keys /home/$1/.ssh/authorized_keys
     chown $1.$1 /home/$1/.ssh/authorized_keys
    SHELL

    # Mount the shared folder as the new created user, so that it can write
    vgrspark.vm.provision "02.mount",
    type: "shell",
    privileged: true,
    keep_color: true,
    args: [ spark_username ],
    inline: <<-SHELL
umount /vagrant
mount -t vboxsf -o uid=`id -u $1`,gid=`id -g $1` vagrant /vagrant
SHELL

    # .........................................
    # Configure the IPython Notebook profile to run Spark jobs
    # Prepared for IPython 4 (so that we configure as a Jupyter app)
    vgrspark.vm.provision "03.nbprofile", 
    type: "shell", 
    privileged: true,
    keep_color: true,    
    args: [ spark_username, port_ipython ],
    inline: <<-SHELL
      su -l "$1" -c "cat <<-EOF > /home/$1/.jupyter/jupyter_notebook_config.py
c = get_config()
# define server
c.NotebookApp.ip = '*'
c.NotebookApp.port = $2
c.NotebookApp.open_browser = False
c.NotebookApp.log_level = 'INFO'
c.NotebookApp.notebook_dir = u'/home/$1/IPNB'  
# Preload matplotlib
c.IPKernelApp.matplotlib = 'inline'
EOF
"
     su -l "$1" <<-EOF
python2.7 -c 'from notebook.services.config import ConfigManager; ConfigManager().update("notebook", {"load_extensions": {"toc": True}})'
EOF
    SHELL

    # .........................................
    # Create the config profiles for defining how Spark notebook submits tasks
    vgrspark.vm.provision "04.nbconfig",
    type: "shell", 
    privileged: true,
    keep_color: true,    
    args: [ spark_basedir, spark_name, spark_mode, 
            spark_history_server, spark_yarn_master, spark_username ],
    inline: <<-SHELL
       CFG=/etc/sysconfig/spark-notebook-config

       # Set the name of the initially active config
       echo "Configuring Spark mode as: $3"
       echo "$3" > $CFG

       # [A] Config for local mode
       cat <<-EOF > ${CFG}-local
NOTEBOOK_USER=$6
NOTEBOOK_SCRIPT="$1/$2/bin/pyspark"

PYSPARK_PYTHON=/opt/ipnb/bin/ext/python2.7
PYSPARK_DRIVER_PYTHON=/opt/ipnb/bin/ext/jupyter-notebook
PYSPARK_SUBMIT_ARGS='--master local[*] --driver-memory 1536M --num-executors 4 --executor-cores 2 --executor-memory 1g'
EOF

       # [B] Config for yarn mode
       cat <<-EOF > ${CFG}-yarn
NOTEBOOK_USER=$6
NOTEBOOK_SCRIPT="$1/$2/bin/pyspark"

HADOOP_CONF_DIR=$1/$2/conf/hadoop
YARN_CONF_DIR=$1/$2/conf/hadoop
YARN_OPTS="--conf spark.yarn.historyServer.address=$4"

PYSPARK_PYTHON=/opt/ipnb/bin/ext/python2.7
PYSPARK_DRIVER_PYTHON=/opt/ipnb/bin/ext/jupyter-notebook
PYSPARK_SUBMIT_ARGS="--master yarn-client --deploy-mode client  --driver-memory 1536M  --num-executors 16 --executor-cores 2 --executor-memory 1g $YARN_OPTS"
EOF
      mkdir -p /opt/hadoop/etc 
      if [ "$4" ]; then
        service spark-notebook config-yarn "$5"
      fi 

       # [C] Config for Spark standalone mode
       cat <<-EOF > ${CFG}-master
NOTEBOOK_USER=$6
NOTEBOOK_SCRIPT="$1/$2/bin/pyspark"

PYSPARK_PYTHON=/opt/ipnb/bin/ext/python2.7
PYSPARK_DRIVER_PYTHON=/opt/ipnb/bin/ext/jupyter-notebook
PYSPARK_SUBMIT_ARGS="--master spark://$5:7077 --deploy-mode client  --driver-memory 1536M  --num-executors 16 --executor-cores 2 --executor-memory 1g"
EOF

      # Link the spark startup script, and set it up for starting
      rm -f /etc/init.d/spark-notebook
      chmod 775 /opt/ipnb/bin/ext/spark-notebook
      ln -s /opt/ipnb/bin/ext/spark-notebook /etc/init.d
      chkconfig --add spark-notebook
  SHELL


  # .........................................
  # Start Spark Notebook
  vgrspark.vm.provision "05.nbstart", 
    type: "shell", 
    privileged: true,
    keep_color: true,    
    inline: "/etc/init.d/spark-notebook start"

  end

end
