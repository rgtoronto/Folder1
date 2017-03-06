# Spectrum Scale Beta 4.2.1 / Conductor for Spark Integration Test
## Install Spectrum Scale Beta 4.2.1
### A. Install Spectrum Scale Master cluster with 3 hosts
#### 0. Prerequisites:
        1. Root access on each host
        2. There is no existing GPFS installation.  Otherwise, see uninstallation steps.
        3. Setup passwordless ssh between all machines within the Spectrum Scale cluster
            $ ssh-keygen -t rsa
            $ ssh-copy-id -i ~/.ssh/id_rsa.pub root@<gpfsserver1> (including the host originally running ssh-keygen)
            $ ssh-copy-id -i ~/.ssh/id_rsa.pub root@<gpfsserver2>
            $ ssh-copy-id -i ~/.ssh/id_rsa.pub root@<gpfsserver3>
#### 1.1 [Option A] Installation steps using .tar package:
        1. Download the gpfs_4.2_20150930_build.tar package
        2. On each host, untar the package:
            # tar xvf gpfs_4.2_20150930_build.tar
        3. Install all the rpm packages:
            # rpm -ivh gpfs*
            NOTE: There may be some prerequisite packages that need to be installed prior
        
#### 1.2. [Option B] Installation steps using executable package:
        1. Download the Spectrum_Scale_Standard-4.2.1.0_beta-x86_64-Linux-install package
        2. On each host, run the executable.  Optionally use --text-only for text installer and --silent to skip license agreement.
            # ./Spectrum_Scale_Protocols_Standard-4.2.1.0_beta-x86_64-Linux-install --text-only --silent
        3. Go to /usr/lpp/mmfs/4.2.1.0/gpfs_rpms
            # cd /usr/lpp/mmfs/4.2.1.0/gpfs_rpms
        4. Install all the rpm packages:
            # rpm -ivh gpfs*
            NOTE: There may be some prerequisite packages that need to be installed prior
        For example: 
#####    x86 Linux:
            rpm -ivh gpfs.base-4.2.1-0.x86_64.rpm gpfs.docs-4.2.1-0.noarch.rpm gpfs.ext-4.2.1-0.x86_64.rpm gpfs.gpl-4.2.1-0.noarch.rpm gpfs.gskit-8.0.50-57.x86_64.rpm gpfs.msg.en_US-4.2.1-0.noarch.rpm
#####    pLE:
            rpm -ivh gpfs.base-4.2.1-0.ppc64le.rpm gpfs.docs-4.2.1-0.noarch.rpm gpfs.ext-4.2.1-0.ppc64le.rpm gpfs.gpl-4.2.1-0.noarch.rpm gpfs.gskit-8.0.50-57.ppc64le.rpm gpfs.msg.en_US-4.2.1-0.noarch.rpm 
#####    UBUNTU:
        If installing on Ubuntu, use dpkg:
            # dpkg -i gpfs.base_4.2.0-0_ppc64el.deb gpfs.docs_4.2.0-0_all.deb gpfs.ext_4.2.0-0_ppc64el.deb gpfs.gpl_4.2.0-0_all.deb gpfs.gskit_8.0.50-47_ppc64el.deb gpfs.msg.en-us_4.2.0-0_all.deb
            If packages require depedencies, run:
            # apt-get -f install
            Verify gpfs packages are installed:
            # dpkg -l|grep gpfs        
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

##### 2. Create the NSDs using the configuration from the previous file:----------------?NSD
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
    
#### 1.1. [Option A] Installation steps using .tar package:
        1. Download the gpfs_4.2_20150930_build.tar package
        2. On each host, untar the package:
            # tar xvf gpfs_4.2_20150930_build.tar
        3. Install all the rpm packages:
            # rpm -ivh gpfs*
            NOTE: There may be some prerequisite packages that need to be installed prior
        
#### 1.2. [Option B] Installation steps using executable package:
        1. Download the Spectrum_Scale_Standard-4.2.1.0_beta-x86_64-Linux-install package
        2. On each host, run the executable.  Optionally use --text-only for text installer and --silent to skip license agreement.
            # ./Spectrum_Scale_Standard-4.2.1.0_beta-x86_64-Linux-install --text-only --silent
        3. Go to /usr/lpp/mmfs/4.2.1.0
            # cd /usr/lpp/mmfs/4.2.1.0
        4. Install all the rpm packages:
            # rpm -ivh gpfs*
            NOTE: There may be some prerequisite packages that need to be installed prior
    		For example: 
##### x86 Linux:
    		rpm -ivh gpfs.base-4.2.1-0.x86_64.rpm gpfs.docs-4.2.1-0.noarch.rpm gpfs.ext-4.2.1-0.x86_64.rpm gpfs.gpl-4.2.1-0.noarch.rpm gpfs.gskit-8.0.50-57.x86_64.rpm gpfs.msg.en_US-4.2.1-0.noarch.rpm
    		
##### pLE:
    		rpm -ivh gpfs.base-4.2.1-0.ppc64le.rpm gpfs.docs-4.2.1-0.noarch.rpm gpfs.ext-4.2.1-0.ppc64le.rpm gpfs.gpl-4.2.1-0.noarch.rpm gpfs.gskit-8.0.50-57.ppc64le.rpm gpfs.msg.en_US-4.2.1-0.noarch.rpm 
    		do so with yum list | grep <>
               yum install 
   
##### UBUNTU
            If installing on Ubuntu, use dpkg:
            # dpkg -i gpfs.base_4.2.0-0_ppc64el.deb gpfs.docs_4.2.0-0_all.deb gpfs.ext_4.2.0-0_ppc64el.deb gpfs.gpl_4.2.0-0_all.deb gpfs.gskit_8.0.50-47_ppc64el.deb gpfs.msg.en-us_4.2.0-0_all.deb
            If packages require depedencies, run:
            # apt-get -f install
            Verify gpfs packages are installed:
            # dpkg -l|grep gpfs
              
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
        The RPM for the GPFS connector can be found packaged in the Spectrum Scale 4.2 Standard package and should be already installed in Section B
        The gpfs.hadoop-connector-2.7.0-6.x86_64.rpm package can be found under 
    	Installing IBM Spectrum Scale Hadoop Connectorhttp://blue04.eng.platformlab.ibm.com:9090/kc/temp_spectrum_conductor_spark_2.1.0/managing_cluster/hadoop_connector_install.dita 
	
        ------------------	
    	new transparency version
    	https://www.ibm.com/developerworks/community/wikis/home?lang=en#!/wiki/General%20Parallel%20File%20System%20(GPFS)/page/HDFS_Transparency?section=1.overview
        ----------- ----------


    	2.5.1 IBM Spectrum Scale (GPFS) Hadoop connector 2.7
    	https://www.ibm.com/developerworks/community/wikis/home?lang=en#!/wiki/General%20Parallel%20File%20System%20(GPFS)/page/Hadoop%20Connector%20Download%20%26%20Info?section=2.5.1IBM%20Spectrum%20Scale

        # rpm -ivh gpfs.hadoop-connector-2.7.0-6.x86_64.rpm
    
        On Ubuntu:
        # dpkg -i gpfs.hadoop-connector_2.7.0-6_ppc64el.deb
    
#### 2. Copy over the following files to the locations indicated:
        # cp /usr/lpp/mmfs/hadoop/install_script/gpfs-callback_*.sh /var/mmfs/etc/
        # cp /usr/lpp/mmfs/hadoop/install_script/gpfs-log4j.properties /var/mmfs/etc/
    
#### 3. Start the connector service:
        # /usr/lpp/mmfs/bin/mmhadoopctl connector start
       Check the status of the service:
        # /usr/lpp/mmfs/bin/mmhadoopctl connector status
    
### B. Configuration
#### 4. Create a core-site.xml file as follows.
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
## Configuration Change on Conductor for Spark
        a. Click "Edit configuration".
        b. Under Settings for Runtime Environment, set the following:
            spark.driver.extraClassPath     /usr/lpp/mmfs/hadoop/hadoop-gpfs-2.7.0.jar
            spark.driver.extraLibraryPath   /usr/lpp/mmfs/hadoop
            spark.executor.extraClassPath   /usr/lpp/mmfs/hadoop/hadoop-gpfs-2.7.0.jar
            spark.executor.extraLibraryPath /usr/lpp/mmfs/hadoop
        c. Under Settings for Environment Variable, set the following:
            HADOOP_CONF_DIR                 <path_to_dir_containing_core-site.xml_file>
            SPARK_SUBMIT_CLASSPATH          /usr/lpp/mmfs/hadoop/hadoop-gpfs-2.7.0.jar
            SPARK_SUBMIT_LIBRARY_PATH       /usr/lpp/mmfs/hadoop
            SPARK_DIST_CLASSPATH            /usr/lpp/mmfs/hadoop/hadoop-gpfs-2.7.0.jar
## Usage of Submitting Spark Jobs
### To submit spark job:
#### Spark 1.6.1 package with gpfs://
    --class org.apache.spark.examples.JavaWordCount --deploy-mode cluster /opt/sparkgpfs/spark-1.6.1-hadoop-2.6/lib/spark-examples-1.6.1-hadoop2.6.0.jar gpfs://asc-10/homedir/mytext.txt
#### Spark 1.5.2 package with gpfs://
    --class org.apache.spark.examples.JavaWordCount --deploy-mode cluster /opt/spark152/spark-1.5.2-hadoop-2.6/lib/spark-examples-1.5.2-hadoop2.6.0.jar gpfs://asc-10/homedir/mytext.txt
#### Spark 1.5.2 package with local file path
    --class org.apache.spark.examples.JavaWordCount --deploy-mode cluster /opt/spark152/spark-1.5.2-hadoop-2.6/lib/spark-examples-1.5.2-hadoop2.6.0.jar /gpfs/conductorFS/mytext.txt
    
## Usage of Spark-shell Jobs
    scala> val demoRDD = sc.textFile("gpfs://asc-10.ma.platformlab.ibm.com/homedir/mytext.txt")
    scala> val wcountsRDD = demoRDD.flatMap(line => line.split(" ")).map(word => (word, 1)).reduceByKey(_ + _)
    scala> wcountsRDD.saveAsTextFile("hdfs://asc-10/homedir/result")