# Spectrum Scale Beta 4.1.1 / Conductor Spark 2.1 Integration Test
## Install Spectrum Scale Beta 4.1.1
### A. Install Spectrum Scale Master cluster with 3 hosts
#### 0. Prerequisites:
        1. Root access on each host
        2. There is no existing GPFS installation.  Otherwise, see uninstallation steps.
        3. Setup passwordless ssh between all machines within the Spectrum Scale cluster
            $ ssh-keygen -t rsa
            $ ssh-copy-id -i ~/.ssh/id_rsa.pub root@<gpfsserver1> (including the host originally running ssh-keygen)
            $ ssh-copy-id -i ~/.ssh/id_rsa.pub root@<gpfsserver2>
            $ ssh-copy-id -i ~/.ssh/id_rsa.pub root@<gpfsserver3>
#### 1. Installation steps using executable package:
        1. Download the Spectrum_Scale_install-4.1.1.0_x86_64_standard_protocols package
        2. On each host, run the executable.  Optionally use --text-only for text installer and --silent to skip license agreement.
            # .Spectrum_Scale_install-4.1.1.0_x86_64_standard_protocols --text-only --silent
        3. Go to /usr/lpp/mmfs/4.1.1/gpfs_rpms
            # cd /usr/lpp/mmfs/4.1.1/gpfs_rpms
        4. Install all the rpm packages:
            # rpm -ivh gpfs*
            NOTE: There may be some prerequisite packages that need to be installed prior
        For example on x86 Linux:
            # rpm -ivh gpfs.base-4.1.1-0.x86_64.rpm gpfs.docs-4.1.1-0.noarch.rpm gpfs.ext-4.1.1-0.x86_64.rpm gpfs.gpl-4.1.1-0.noarch.rpm gpfs.gskit-8.0.50-40.x86_64.rpm gpfs.hadoop-2-connector-4.1.1-0.x86_64.rpm gpfs.msg.en_US-4.1.1-0.noarch.rpm

#### 2. Build gpfs portability layer modules:
        1. On each host, run:
           # /usr/lpp/mmfs/bin/mmbuildgpl
    
#### 3. Create cluster with one server:
        # /usr/lpp/mmfs/bin/mmcrcluster -N <gpfsserver1>:manager-quorum -C gpfsForConductor

#### 4. Add additional server nodes to cluster:
        # /usr/lpp/mmfs/bin/mmaddnode -N <gpfsserver2>:manager-nonquorum

#### 5. Grant licenses for GPFS servers:
        # /usr/lpp/mmfs/bin/mmchlicense server -N <gpfsserver1>,<gpfsserver2>,<gpfsserver3>

#### 6. Startup GPFS services:
        # /usr/lpp/mmfs/bin/mmstartup -N <gpfsserver1>,<gpfsserver2>,<gpfsserver3>

#### 7. Create NSDs:
##### 1. Create stanza file called "stanza.txt".  The following is an example.
            %pool: pool=system layoutMap=cluster blocksize=256K #using smaller blocksize for metadata
            %pool: pool=fpodata layoutMap=cluster blocksize=1024K
            allowWriteAffinity=yes #this option enables FPO feature
            writeAffinityDepth=1 #place 1st copy on disks local to the node writing data
            blockGroupFactor=128 #yields chunk size of 128MB
            %nsd: nsd=nsd1 device=/dev/sdb servers=<gpfsserver1> usage=metadataOnly failureGroup=1 pool=system
            %nsd: nsd=nsd2 device=/dev/sdc servers=<gpfsserver1> usage=dataOnly failureGroup=1 pool=fpodata
            %nsd: nsd=nsd3 device=/dev/sdb servers=<gpfsserver2> usage=metadataOnly failureGroup=2 pool=system
            %nsd: nsd=nsd4 device=/dev/sdc servers=<gpfsserver2> usage=dataOnly failureGroup=2 pool=fpodata
            %nsd: nsd=nsd5 device=/dev/sdb servers=<gpfsserver3> usage=metadataOnly failureGroup=3 pool=system
            %nsd: nsd=nsd6 device=/dev/sdc servers=<gpfsserver3> usage=dataOnly failureGroup=3 pool=fpodata

##### 2. Create the NSDs using the configuration from the previous file:
        # /usr/lpp/mmfs/bin/mmcrnsd -F stanza.txt
        NOTE: If encounter message "mmcrnsd: Disk device sdb refers to an existing NSD", specify "-v no" to ignore the validation
        
        For 4.1.1
        /usr/lpp/mmfs/bin/mmcrnsd -F stanza.txt -v no
      
#### 8. Create the filesystem (this will reformat the disks).  Provide a name for the file system.
        # /usr/lpp/mmfs/bin/mmcrfs <conductorFS> -F stanza.txt

#### 9. Mount the filesystem:
        # /usr/lpp/mmfs/bin/mmmount <conductorFS> -a
  
#### 10. Verify GPFS is mounted:
        # /usr/lpp/mmfs/bin/mmlsmount all
        Access the mounted file system:
        # cd /gpfs/<conductorFS>/
        # ls

#### 11. [OPTIONAL] Create Fileset and configure policy rules
        FileSet is the recommended granularity of configuration
        1. Create fileset:
        # mmcrfileset conductorFS <fileset1>
        2. Link fileset:
        # mmlinkfileset conductorFS <fileset1> -J <juntionpath>  # JunctionPath should not exist
        3. Define Rules:
        # mmchpolicy conductorFS <policyfile>
        Sample policy file:
        RULE 'history' SET POOL 'fpodata' REPLICATE(1,1) FOR FILESET (fileset1)
        RULE default SET POOL fpodata

### B. Install Spectrum Scale Client
#### 0. Prerequisites:
        1. Root access on each host
        2. There is no existing GPFS installation.  Otherwise, see uninstallation steps.
        3. Setup passwordless ssh between management quorum node and the client host.  Login to <gpfsserver1>.
            $ ssh-copy-id -i ~/.ssh/id_rsa.pub root@<gpfsclient1>
    
#### 1. Installation steps using executable package:
        1. Download the Spectrum_Scale_install-4.1.1.0_x86_64_standard_protocols package
        2. On each host, run the executable.  Optionally use --text-only for text installer and --silent to skip license agreement.
            # .Spectrum_Scale_install-4.1.1.0_x86_64_standard_protocols --text-only --silent
        3. Go to /usr/lpp/mmfs/4.1.1/gpfs_rpms
            # cd /usr/lpp/mmfs/4.1.1/gpfs_rpms
        4. Install all the rpm packages:
            # rpm -ivh gpfs*
            NOTE: There may be some prerequisite packages that need to be installed prior
        For example: 
#####    x86 Linux:
            rpm -ivh gpfs.msg.en_US-4.1.1-0.noarch gpfs.ext-4.1.1-0.x86_64 gpfs.base-4.1.1-0.x86_64 gpfs.gskit-8.0.50-40.x86_64 gpfs.gpl-4.1.1-0.noarch gpfs.hadoop-2-connector-4.1.1-0.x86_64 gpfs.docs-4.1.1-0.noarch
#####    pLE:
            rpm -ivh gpfs.msg.en_US-4.1.1-0.noarch gpfs.ext-4.1.1-0.ppc64le gpfs.base-4.1.1-0.ppc64le gpfs.gskit-8.0.50-40.ppc64le gpfs.gpl-4.1.1-0.noarch gpfs.hadoop-2-connector-4.1.1-0.ppc64le gpfs.docs-4.1.1-0.noarch
              
#### 2. Build GPFS portability layer modules:
        1. On the host being installed as the client, run:
           # /usr/lpp/mmfs/bin/mmbuildgpl
       
#### 3. Add the client node. Run the following on the manager-quorum node:
        # /usr/lpp/mmfs/bin/mmaddnode -N <gpfsclient1>:client-nonquorum
        # /usr/lpp/mmfs/bin/mmchlicense client --accept -N <gpfsclient1>
        # /usr/lpp/mmfs/bin/mmstartup -N <gpfsclient1>
    
#### 4. Verify the node as been added as client:
        # /usr/lpp/mmfs/bin/mmlsnode
    
#### 5. Verify GPFS is mounted.  On the client node, run the following:
        # /usr/lpp/mmfs/bin/mmlsmount all
    
#### 6. Access the mounted file system:
        # cd /gpfs/<conductorFS>/
        # ls

### C. Shutdown and remove node
#### 1. Shutdown the node:
        # /usr/lpp/mmfs/bin/mmshutdown -N <gpfsserver3>

#### 2. Remove the node:
        # /usr/lpp/mmfs/bin/mmdelnode -N <gpfsserver3>

#### 3. Run 'mmlscluster' to verify the node has been removed.
        # /usr/lpp/mmfs/bin/mmlscluster 
        
### D. Uninstall GPFS
#### 0. Refer to the shutdown and remove node section.
#### 1. Uninstall the rpm packages
        a) Run 'rpm -qa | grep gpfs' to get list of packages
        b) Run 'rpm -e <package name1> []...<package name2>]'
#### 2. Delete following directories:
        # rm -rf /var/mmfs/ /usr/lpp/mmfs/ /var/adm/ras/mm* /tmp/mmfs/
        
## Spectrum Scale Connector
### A. Install Spectrum Scale Connector
#### 0. Prerequisites:
        a. These steps need to be performed on each node that requires remote to access the GPFS environment

#### 1. Install the Spectrum Scale connector.
    	IBM Spectrum Scale (GPFS) Hadoop connector 2.5
        # rpm -ivh /usr/lpp/mmfs/4.1.1/gpfs_rpms/gpfs.hadoop-2-connector-4.1.1-0.x86_64.rpm
    
#### 2. Copy over the following files to the locations indicated:
        # cp /usr/lpp/mmfs/fpo/hadoop-2.5/install_script/gpfs-callback_*.sh /var/mmfs/etc/
        # cp /usr/lpp/mmfs/fpo/hadoop-2.5/gpfs-connector-daemon /var/mmfs/etc/

#### 3. Start the connector service
        # Run following command on each Spectrum Scale node 
            export HADOOP_HOME=/usr/lpp/mmfs/fpo/hadoop-2.5
        # Run following command on Spectrum Scale master-quorum node
            /usr/lpp/mmfs/bin/mmdsh -N all "HADOOP_HOME=${HADOOP_HOME} /usr/lpp/mmfs/fpo/hadoop-2.5/install_script/deploy_connector.sh -d apache -v 2.5 install"
        # Check the status of the service:
            /usr/lpp/mmfs/bin/mmdsh -N all /usr/lpp/mmfs/bin/mmhadoopctl connector status
    
### B. Configuration
#### 1. Create a core-site.xml file as follows.
       Replace "Your_GPFS_Mount_Dir" with the actual GPFS mount location.
        <configuration>
        <property>
        <name>fs.AbstractFileSystem.gpfs.impl</name>
        <value>org.apache.hadoop.fs.gpfs.GeneralParallelFs</value>
        </property>
        <property>
          <name>fs.AbstractFileSystem.hdfs.impl</name>
          <value>org.apache.hadoop.fs.gpfs.GeneralParallelFs</value>
        </property>
        <property>
          <name>fs.gpfs.impl</name>
          <value>org.apache.hadoop.fs.gpfs.GeneralParallelFileSystem</value>
        </property>
        <property>
          <name>fs.hdfs.impl</name>
          <value>org.apache.hadoop.fs.gpfs.GeneralParallelFileSystem</value>
        </property>
        <property>
          <name>gpfs.mount.dir</name>
          <value>Your_GPFS_Mount_Dir</value>
        </property>
        </configuration>
#### 2. Configuration Change on Conductor for Spark
        a. Click "Edit configuration".
        b. Under Settings for Runtime Environment, set the following:
            spark.driver.extraClassPath     /usr/lpp/mmfs/fpo/hadoop-2.5/hadoop-gpfs-2.5.jar
            spark.driver.extraLibraryPath   /usr/lpp/mmfs/fpo/hadoop-2.5
            spark.executor.extraClassPath   /usr/lpp/mmfs/fpo/hadoop-2.5/hadoop-gpfs-2.5.jar
            spark.executor.extraLibraryPath /usr/lpp/mmfs/fpo/hadoop-2.5
        c. Under Settings for Environment Variable, set the following:
            HADOOP_CONF_DIR                 <path_to_dir_containing_core-site.xml_file>
            SPARK_SUBMIT_CLASSPATH          /usr/lpp/mmfs/fpo/hadoop-2.5/hadoop-gpfs-2.5.jar
            SPARK_SUBMIT_LIBRARY_PATH       /usr/lpp/mmfs/fpo/hadoop-2.5
            SPARK_DIST_CLASSPATH            /usr/lpp/mmfs/fpo/hadoop-2.5/hadoop-gpfs-2.5.jar
            
## Usage of Submitting Spark Jobs
### To submit spark job:
#### Spark 1.6.1 package with gpfs://
    --class org.apache.spark.examples.JavaWordCount --deploy-mode cluster /opt/sparkgpfs/spark-1.6.1-hadoop-2.6/lib/spark-examples-1.6.1-hadoop2.6.0.jar gpfs://asc-10/mytext.txt
#### Spark 1.5.2 package with gpfs://
    --class org.apache.spark.examples.JavaWordCount --deploy-mode cluster /opt/spark152/spark-1.5.2-hadoop-2.6/lib/spark-examples-1.5.2-hadoop2.6.0.jar gpfs://asc-10/mytext.txt
#### Spark 1.5.2 package with local file path
    --class org.apache.spark.examples.JavaWordCount --deploy-mode cluster /opt/spark152/spark-1.5.2-hadoop-2.6/lib/spark-examples-1.5.2-hadoop2.6.0.jar /gpfs/conductorFS/mytext.txt
    
## Usage of Spark-shell Jobs
    switch to $CLUSTERADMIN user
    
    scala> val demoRDD = sc.textFile("gpfs://asc-10.ma.platformlab.ibm.com/homedir/mytext.txt")
    scala> val wcountsRDD = demoRDD.flatMap(line => line.split(" ")).map(word => (word, 1)).reduceByKey(_ + _)
    scala> wcountsRDD.saveAsTextFile("hdfs://asc-10/homedir/result")