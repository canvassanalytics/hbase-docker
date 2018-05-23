HBase in Docker
===============

This configuration builds a docker container to run HBase (with
embedded Zookeeper) running on the files inside the container.

NOTE
----

The approach here requires editing the local server's `/etc/hosts`
file to add an entry for the container hostname.  This is because
HBase uses hostnames to pass connection data back out of the
container (from it's internal Zookeeper).

Hopefully this can be improved with Docker's newer networking
but this hasn't been fixed yet.


Build Image
-----------

    $ docker build -t datuh/hbase .


Pull image
----------

If you want to pull the image already built then use this

    $ docker pull datuh/hbase


Run HBase
---------

To run HBase by hand:

    $ mkdir data
    $ id=$(docker run --name=hbase-docker -h hbase-docker -d -v $PWD/data:/data dajobe/hbase)

To run it and adjust the host system's locally by editing
`/etc/hosts` to alias the DNS hostname 'hbase-docker' to the
container, use this:

    $ ./start-hbase.sh

This will require you to enter your sudo password to edit the host
machine's `/etc/hosts` file

If you want to run multiple hbase dockers on the same host, you can
give them different hostnames with the '-h' / '--hostname' argument.
You may have to give them different ports though.  Not tested.


Find Hbase status
-----------------

Master status if docker container DNS name is 'hbase-docker'

    http://hbase-docker:16010/master-status

The region servers status pages are linked from the above page.

Thrift UI

    http://hbase-docker:9095/thrift.jsp

REST server UI

    http://hbase-docker:8085/rest.jsp

(Embedded) Zookeeper status

    http://hbase-docker:16010/zk.jsp


See HBase Logs
--------------

If you want to see the latest logs live use:

    $ docker attach $id

Then ^C to detach.

To see all the logs since the HBase server started, use:

    $ docker logs $id

and ^C to detach again.

To see the individual log files without using `docker`, look into
the data volume dir eg $PWD/data/logs if invoked as above.


Test HBase is working via python over Thrift
--------------------------------------------

Here I am connecting to a docker container with the name 'hbase-docker'
(such as created by the start-hbase.sh script).  The port 9090 is the
Thrift API port because [Happybase][1] [2] uses Thrift to talk to HBase.

    $ ipython
    Python 2.7.9 (default, Mar  1 2015, 12:57:24)
    Type "copyright", "credits" or "license" for more information.
    
    IPython 2.3.0 -- An enhanced Interactive Python.
    ?         -> Introduction and overview of IPython's features.
    %quickref -> Quick reference.
    help      -> Python's own help system.
    object?   -> Details about 'object', use 'object??' for extra details.
    
    In [1]: import happybase
    
    In [2]: connection = happybase.Connection('hbase-docker', 9090)
    
    In [3]: connection.create_table('table-name', { 'family': dict() } )
    
    In [4]: connection.tables()
    Out[4]: ['table-name']
    
    In [5]: table = connection.table('table-name')
    
    In [6]: table.put('row-key', {'family:qual1': 'value1', 'family:qual2': 'value2'})
    
    In [7]: for k, data in table.scan():
       ...:     print k, data
       ...:
    row-key {'family:qual1': 'value1', 'family:qual2': 'value2'}
    
    In [8]:
    Do you really want to exit ([y]/n)? y
    $

(Simple install for happybase: `sudo pip install happybase` although I
use `pip install --user happybase` to get it just for me)


Proxy HBase UIs locally
-----------------------

If you are running docker on a remote machine, it is handy to see
these server-private urls in a local browser so here is a
~/.ssh/config fragment to do that

    Host my-docker-server
    Hostname 1.2.3.4
        LocalForward 127.0.0.1:16010 127.0.0.1:16010
        LocalForward 127.0.0.1:9095 127.0.0.1:9095
        LocalForward 127.0.0.1:8085 127.0.0.1:8085

When you `ssh my-docker-server` ssh connects to the docker server and
forwards request on your local machine on ports 16010 / 16030 to the
remote ports that are attached to the hbase container.

The bottom line, you can use these URLs to see what's going on:

  * http://localhost:16010/master-status for the Master Server
  * http://localhost:9095/thrift.jsp for the thrift UI
  * http://localhost:8085/rest.jsp for the REST server UI
  * http://localhost:16010/zk.jsp for the embedded Zookeeper

to see what's going on in the container and since both your local
machine and the container are using localhost (aka 127.0.0.1), even
the links work!





Loading on Server not connected to Internet
-----

From a computer with internet access

    $ docker pull <image_name>
    $ docker save -o <image_name.docker> <image_name>

This will save the image to the current directory in a file of image_name.docker

    $ scp <file_name>.docker <username>@<hostname>: /file/path/on/server

This will copy the docker image to the destination server
From the server run:
    
    $ docker load --input <file_name>.docker

Starting HBase Thrift Server on HDInsight
-----
SSH to the cluster (get from HDInsight Blade)

    $ sudo /usr/hdp/current/hbase-master/bin/hbase-daemon.sh start thrift
    
# Some Tuning Parameters for HBase
### Zookeeper Session Timeout
```bash
<property>
    <name>zookeeper.session.timeout</name>
    <value>20000</value>
</property>
```
### HBase RPC Timeout
```bash
<property>
    <name>hbase.rpc.timeout</name>
    <value>900000</value> <!-- 15 minutes -->
</property>
```
### HBase Region Server Lease Period
```bash
<property>
    <name>hbase.regionserver.lease.period</name>
    <value>900000</value> <!-- 900 000, 15 minutes -->
</property>
```
### HBase Thrift Connection Max
Sets an unlimited timeout on the Thrift Connect,  this is useful for for when connection are idle for a while
```bash
<property>
    <name>hbase.thrift.connection.max-idletime</name>
    <value>1800000</value>
</property>
```

# Backing up and restoring data from one cluster to another
## From the Source
connect to the head node and run
```
mkdir backups
pwd
hbase org.apache.hadoop.hbase.mapreduce.Export "<tableName>" "/results/from/pwd"
```
If you run that from the root login directory it will just be saved locally, optionall you could supply "/<path>/<to>/<dir>"

Now copy it to the other server head node
```
ls
scp ~/<filename.x>/ username@remote:~
del filename.x
```
the `ls` shows you the name of the file
`scp` copies the file to the destination server
`del` deletes the file to clean up the server

## from the Destination server head node
```
ls
```
this should show you the file
Now import the file
```
hbase org.apache.hadoop.hbase.mapreduce.Import "<tableName>" "."
```
You will either need a new table name or you will first have to disable the exiting table


## From Source
connect to the head node and run
```
$ hbase org.apache.hadoop.hbase.mapreduce.Export "<table_name>" "wasbs://<Blob_Container>@<Storage_Account_Name>.blob.core.windows.net/hbase/backups/<folder_name>"
```
* <table_name>: is the table name you want to backup
* <Blob_Container>: is the name of the blob used by hbase; get this from Azure Explorer
* <Storage_Account_Name>: the the storage account name used by HDInsight; get this from the azure portal
* <Flolder_Name>: is the directory on azure storage where the backup will be place

Now go to Azure storage explorer and you should see something in the blob container storage account at hbase-prod/hbase/backups

Copy the contents of that directory to the same directory on the destination server azure storage blob account

## From Destination
If you are copying to a new table on the destination you will first need to create the table using:
```
$ hbase shell
hbase(main):003:0> create 'table-name', 'data'
```
That will create the table named `table-name` with a column family of `data`

Then you can run the import
```
$ hbase org.apache.hadoop.hbase.mapreduce.Import "<table_name>" "wasbs://<Blob_Container>@<Storage_Account_Name>.blob.core.windows.net/hbase/backups/<Flolder_Name>"
```
* <table_name>: the table name on the destination instance
* <Blob_Container>:  is the name of the blob used by the destination hbase instance; get this from Azure Explorer
* <Storage_Account_Name>: the the storage account name used by the destination HDInsight instance; get this from the azure portal
* <Flolder_Name>: is the directory where you copied the file to
