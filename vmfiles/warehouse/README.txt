This folder contains the tables that Spark SQL in the VM saves as system tables.
Its contents should not be modified outside Spark.

Since it is on the Vagrant shared folder, it will be preserved accross VM 
reinstallations.

Using a table requires access to the corresponding `metastore_db` folder 
(which typically exists in the folder where the Notebook that created the 
table is).
