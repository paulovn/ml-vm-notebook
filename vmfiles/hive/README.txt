This folder may contain two subdirectories, created & used by Spark SQL 
processes running in the virtual machine:
  * the "metastore_db" subdirectory contains the Derby metastore used by Spark
    SQL to locate tables
  * the "warehouse" subdirectory contains the tables that Spark SQL saves as 
    system tables.

Since the metastore & tables are on the Vagrant shared folder, they will be 
preserved accross VM reinstallations. Folder contents should not be 
modified outside Spark.

Note that Spark uses this directory because it has been configured that way by
the Hive configuration file in /opt/spark/current/conf/hive-site.xml. It can
be made to point elsewhere by modifying that file.

Note also that the Derby DB used by Spark in local mode is a single-process
DB, which means that there can only be one Spark SQL process running at the
same time in the VM, otherwise there will be conflicts. 

Deleting the javax.jdo.option.ConnectionURL configuration in hive-site.xml 
would make Spark SQL use the default behaviour of creating a metastore_db in 
the current directory, hence allowing more than one Spark SQL process, as long 
as they are run from different directories (however having more than 1
Spark SQL process running in the VM is not advisable anyway, since the VM
might run into memory problems).
