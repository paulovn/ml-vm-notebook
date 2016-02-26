# -*- mode: ruby -*-
# vi: set ft=ruby :
# **************************************************************************
# Add specific configuration for running IPython notebooks on a Spark VM
# **************************************************************************

# --------------------------------------------------------------------------
# Variables defining the configuration of Spark & notebook 
# Modify as needed

# RAM memory used for the VM, in MB
vm_memory = '2048'

# Username that will run all spark processes.
# If remote (yarn) mode is ever going to be used, it is advisable to change it
# to a recognizable unique name, so that it is easily identified in the logs
spark_username = 'sparkvmuser'

# The virtual machine exports the port where the notebook process by forwarding
# it to this port of the local machine
# So to access the notebook server, you point to http://localhost:<port>
port_ipython = 8008

# Note there is an additional port exported: the Spark UI driver is forwarded 
# to port 4040

# This defines the Spark notebook processing mode. There are three choices
# available: "local", "yarn", "standalone"
# It can be changed at runtime by executing inside the virtual machine, as 
# root user, "service spark-notebook set-mode <mode>"
spark_mode = 'local'

# -----------------
# These 3 options are used only when running non-local tasks. They define
# the access points for the remote cluster.
# They can also be modified at runtime by executing inside the virtual
# machine: "sudo service spark-notebook set-addr <A> <B> <C>"
# **IMPORTANT**: If remote mode is to be used, the virtual machine needs
# a network interface in bridge mode. In that case Uncomment the relevant
# lines in the networking section below

# [A] The location of the cluster master (the YARN Resource Manager in Yarn 
# mode, or the Spark master in standalone mode)
spark_master = 'samson01.hi.inet'
# [B] The host running the HDFS namenode
spark_namenode = spark_master
# [C] The location of the Spark History Server
spark_history_server = 'samson03.hi.inet:18080'
# ------------------


# --------------------------------------------------------------------------
# Variables defining the Spark installation in the base box. 
# Don't change these

# The version of Spark we are using
spark_version = '1.6.0'
spark_name = 'spark-' + spark_version + '-bin-hadoop2.6'

# The place where Spark is deployed inside the local machine
spark_basedir = '/opt/spark'


# --------------------------------------------------------------------------
# Vagrant configuration

vagrant_command = ARGV[0]

# The "2" in Vagrant.configure sets the configuration version
Vagrant.configure(2) do |config|

  # Use our custom username, instead of the default "vagrant"
  if vagrant_command == "ssh"
      config.ssh.username = spark_username
  end


  config.vm.define "vm-spark-nb64" do |vgrspark|

    #config.name = "vgr-pyspark"

    # The base box we are using. As fetched from ATLAS
    vgrspark.vm.box_version = "= 0.9.8"
    vgrspark.vm.box = "paulovn/spark-base64"

    # Alternative place: UAM internal
    #vgrspark.vm.box = "uam/spark-base64"
    #vgrspark.vm.box_url = "http://svrbigdata.ii.uam.es/vm/uam-spark-base64.json"
    # Alternative place: TID internal or local box
    #vgrspark.vm.box = "tid/spark-base64"
    #vgrspark.vm.box_url = "http://artifactory.hi.inet/artifactory/vagrant-machinelearning/tid-spark-base64.json"
    #vgrspark.vm.box_url = "file:///almacen/VM/VagrantBox/spark-base64-LOCAL.json"

    # Disable automatic box update checking. If you disable this, then
    # boxes will only be checked for updates when the user runs
    # `vagrant box outdated`. This is not recommended.
    # vgrspark.vm.box_check_update = false

    # Deactivate the usual synced folder and use instead a local subdirectory
    vgrspark.vm.synced_folder ".", "/vagrant", disabled: true
    vgrspark.vm.synced_folder "vmfiles", "/vagrant", 
      mount_options: ["dmode=775","fmode=664"],
      disabled: false
    #owner: spark_username
    #auto_mount: false
  
    # Customize the virtual machine: set hostname & allocated RAM
    vgrspark.vm.hostname = "vm-sparknotebook"
    vgrspark.vm.provider :virtualbox do |vb|
      # Set the hostname in VirtualBox
      vb.name = vgrspark.vm.hostname.to_s
      # Customize the amount of memory on the VM
      vb.memory = vm_memory
      # Display the VirtualBox GUI when booting the machine
      #vb.gui = true
    end

    # **********************************************************************
    # Networking

    # ---- NAT interface ----
    # NAT port forwarding
    vgrspark.vm.network :forwarded_port, 
    guest: port_ipython, 
    host: port_ipython                  # Notebook UI
    # Spark driver UI
    vgrspark.vm.network :forwarded_port, host: 4040, guest: 4040, 
    auto_correct: true
    # Spark driver UI for the 2nd application (e.g. a command-line job)
    vgrspark.vm.network :forwarded_port, host: 4041, guest: 4041,
    auto_correct: true

    # In case we want to fix Spark ports
    #vgrspark.vm.network :forwarded_port, host: 9234, guest: 9234
    #vgrspark.vm.network :forwarded_port, host: 9235, guest: 9235
    #vgrspark.vm.network :forwarded_port, host: 9236, guest: 9236
    #vgrspark.vm.network :forwarded_port, host: 9237, guest: 9237
    #vgrspark.vm.network :forwarded_port, host: 9238, guest: 9238

    # ---- bridged interface ----
    # Declare a public network
    # This enables the machine to be connected from outside, which is a
    # must for Spark [it needs SPARK_LOCAL_IP to be set to the outside-visible
    # interface].
    # --> Uncomment the following two lines to enable bridge mode:
    #vgrspark.vm.network "public_network",
    #type: "dhcp"

    # --> if the host has more than one interface, we can set which one to use
    #bridge: "wlan0"
    # --> we can also set the MAC address we will send to the DHCP server
    #:mac => "08002710A7ED"


    # ---- private interface ----
    # Create a private network, which allows host-only access to the machine
    # using a specific IP.
    #vgrspark.vm.network "private_network", ip: "192.72.33.10"


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
      id "$1" >/dev/null 2>&1 || useradd -c 'User for Spark Notebook' -m -G vagrant "$1"
      su -l "$1" <<-EOF
mkdir bin tmp .ssh 2>/dev/null
chmod 700 .ssh
rm -f bin/{python2.7,pip,ipython}
ln -s /opt/ipnb/bin/ext/{python2.7,pip,ipython} /home/$1/bin
test -d .jupyter || mkdir .jupyter
test -h IPNB || { rm -f IPNB; ln -s /vagrant/IPNB/ IPNB; }
echo "export PYSPARK_DRIVER_PYTHON=ipython" >> .bash_profile
EOF
      # Install the vagrant public key so that we can ssh to this account
      cp -p /home/vagrant/.ssh/authorized_keys /home/$1/.ssh/authorized_keys
      chown $1.$1 /home/$1/.ssh/authorized_keys
    SHELL

    # Mount the shared folder as the new created user, so that it can write
    # ---> no, instead we have added the user to the vagrant group and
    # mounted the shared folder with group permissions
#    vgrspark.vm.provision "02.mount",
#    type: "shell",
#    privileged: true,
#    keep_color: true,
#    args: [ spark_username ],
#    inline: <<-SHELL
#umount /vagrant
#mount -t vboxsf -o uid=$(id -u $1),gid=$(id -g $1) vagrant /vagrant
#SHELL

    # .........................................
    # Create the IPython Notebook profile ready to run Spark jobs
    # Prepared for IPython 4 (so that we configure as a Jupyter app)
    vgrspark.vm.provision "03.nbprofile", 
    type: "shell", 
    privileged: true,
    keep_color: true,    
    args: [ spark_username, port_ipython, spark_basedir ],
    inline: <<-SHELL
     # Create the Jupyter config
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
     # Install the IRKernel
     su -l "$1" <<-EOF
PATH=/opt/ipnb/bin:$PATH LD_LIBRARY_PATH=/opt/rh/python27/root/usr/lib64 Rscript -e 'IRkernel::installspec()'
EOF

     # Install the Scala Spark kernel
     KDIR=/home/$1/.local/share/jupyter/kernels/spark
     su -l "$1" <<-EOF
mkdir -p $KDIR
cat <<-KERNEL >$KDIR/kernel.json
{
    "display_name": "Scala 2.10",
    "language_info": { "name": "scala" },
    "argv": [
        "$3/spark-kernel/bin/spark-kernel",
        "--profile",
        "{connection_file}"
    ],
    "codemirror_mode": "scala",
    "env": {
        "SPARK_OPTS": "--master=local[2] --driver-java-options=-Xms1024M --driver-java-options=-Xmx2048M --driver-java-options=-Dlog4j.logLevel=info",
        "MAX_INTERPRETER_THREADS": "16",
        "CAPTURE_STANDARD_OUT": "true",
        "CAPTURE_STANDARD_ERR": "true",
        "SEND_EMPTY_OUTPUT": "false",
        "SPARK_HOME": "$3/current",
        "PYTHONPATH": "$3/current/python/pyspark:$3/current/python/lib/py4j-0.8.2.1-src.zip"
     }
}
KERNEL
# Copy logo
cp -p $3/spark-kernel/scala-spark-icon-64x64.png $KDIR/logo-64x64.png
EOF

     # Install the notebook extensions
     su -l "$1" <<-EOF
python2.7 -c 'from notebook.services.config import ConfigManager; ConfigManager().update("notebook", {"load_extensions": {"toc": True, "toggle-headers": True }})'
EOF

     # Put the custom icon in place
     cd /opt/ipnb/lib/python2.7/site-packages/notebook/static/base/images
     mv favicon.ico favicon-orig.ico
     ln -s favicon-custom.ico favicon.ico

    SHELL

    # .........................................
    # Install the Spark notebook startup script & configure it
    # Configure Spark execution mode & remote access if defined
    vgrspark.vm.provision "04.nbconfig",
    type: "shell", 
    privileged: true,
    keep_color: true,    
    args: [ spark_basedir, spark_username, spark_mode, 
            spark_master, spark_namenode, spark_history_server ],
    inline: <<-SHELL
     # Link the spark startup script, and set it up for starting
     rm -f /etc/init.d/spark-notebook
     chmod 775 /opt/ipnb/bin/ext/spark-notebook
     ln -s /opt/ipnb/bin/ext/spark-notebook /etc/init.d
     chkconfig --add spark-notebook

     # Create the config for spark notebook
     cat <<-EOF > /etc/sysconfig/spark-notebook
NOTEBOOK_USER=$2
NOTEBOOK_SCRIPT="$1/current/bin/pyspark"
PYSPARK_DRIVER_PYTHON=/opt/ipnb/bin/ext/jupyter-notebook
EOF

     # Configure remote addresses
     if [ "$4" ]; then
       service spark-notebook set-addr yarn "$4" "$5" "$6"
       service spark-notebook set-addr standalone "$4" "$5" "$6"
     fi 

     # Set the name of the initially active config
     echo "Configuring Spark mode as: $3"
     service spark-notebook set-mode "$3"
  SHELL


    # .........................................
    # Create a subdirectory in the shared folder linked to the 
    # default name of the Hive warehouse directory
    vgrspark.vm.provision "05.hive.warehouse",
    type: "shell",
    keep_color: true,
    privileged: true,
    inline: <<-SHELL
      mkdir -p /user/hive
      ln -s /vagrant/warehouse /user/hive/warehouse
    SHELL


    # .........................................
    # Install the necessary components for nbconvert to work.
    # Do it only if the environment variable PROVISION_NBCONVERT has a 1 value
    if (ENV['PROVISION_NBCONVERT'] == '1')
      vgrspark.vm.provision "05.nbconvert",
        type: "shell",
        run: "never",
        keep_color: true,
        inline: <<-SHELL
          echo "Installing nbconvert requisites"
          sudo yum install -y pandoc inkscape
          pip install pandoc
          DIR=$(kpsewhich -var-value TEXMFLOCAL)
          mkdir -p $DIR
          cd $DIR
          for p in collectbox adjustbox; do
            wget --no-verbose http://mirrors.ctan.org/install/macros/latex/contrib/$p.tds.zip
          unzip $p.tds.zip
          done
          texhash
        SHELL
    end


    # .........................................
    # Start Spark Notebook
    # Note: we make this one to run every time the machine boots, since during 
    # the VM boot sequence the startup script is executed before vagrant has 
    # mounted the shared folder, and hence it fails. 
    # Running it as a provisioning makes it run *after* vagrant mounts, so 
    #  this way it works.
    # [An alternative would be to force mounting on startup, by adding the
    # vboxsf mount point to /etc/fstab during provisioning]
    vgrspark.vm.provision "10.nbstart", 
      type: "shell", 
      run: "always",
      privileged: true,
      keep_color: true,    
      inline: "/etc/init.d/spark-notebook start"


  end # config.vm.define

end
