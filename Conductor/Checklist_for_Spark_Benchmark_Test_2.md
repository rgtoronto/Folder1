# Spark Performance Benchmark Checklist

## Environment Preparation:
### 1. Make sure to check that the network is connected to the 10GB ethernet switch
### 2. Make sure to do a format on all disks /dev/sdc to /dev/sdm
### Cluster 
    red15.eng.platformlab.ibm.com, 
    red16.eng.platformlab.ibm.com, 
    red17.eng.platformlab.ibm.com, 
    red18.eng.platformlab.ibm.com, 
    red19.eng.platformlab.ibm.com, 
    red20.eng.platformlab.ibm.com, 
    red21.eng.platformlab.ibm.com,  master
    red22.eng.platformlab.ibm.com, 
    red23.eng.platformlab.ibm.com, 
    red24.eng.platformlab.ibm.com, 
    red25.eng.platformlab.ibm.com
    
    Credetial to logon:
    root/Perf0rmance
    
    For Spark settings, you could find following files for spark configurations on red21.eng.platformlab.ibm.com
    /opt/sparkCondctor2/spark-1.5.2-hadoop-2.6/conf/spark-defaults.conf
    /opt/sparkCondctor2/spark-1.5.2-hadoop-2.6/conf/spark-env.sh
    
    Conductor with Spark 2.1 log:   
    red21.eng.platformlab.ibm.com, under /perf_test/perf_data/160620_1131_latest
    
    [root@red21 ~]# echo $EGO_TOP
    /opt/conductorPerf2GM
    
    [root@red21 ~]# echo $SPARK_HOME
    /opt/sparkCondctor2/spark-1.5.2-hadoop-2.6

### After REBOOT:

#### [ ] 1. Check to make sure that all hosts are pingable.
    If any hosts are not reachable, login to the host's individual IPMI:
    a) Open web browser to https://redXXcs (login="USERID" / password="PASSW0RD") where the "0" is a zero.
    b) Open up the console to the host and check to see where it is in the boot sequence
         i. If it's stuck before even making it to the OS bootup, try to force shutdown and then force boot
        ii. If it's past the OS boot sequence or forced into safe mode, check that one of the mounted disks isn't the cause
            - Previously on red23,red24,red25 we had to comment out this line in /etc/fstab and then reboot the host:
                /dev/sdc /mnt/c ext4 noatime,nodiratime 0 0

#### [ ] 2. Make sure that the NFS server on red24 is up and running.
    Sometimes the NFS services do not automatically startup.
    a) Manually startup the NFS services on red24:
        # service rpcbind start; service nfs start
    b) As a result all hosts need to manually re-mount the NFS mount point.
        Login to red21 as root and run the following:
        # for i in `seq 15 25`; do echo red$i; ssh red$i mount /mnt/nfs/; done
    c) Check that /mnt/nfs can see the shared folders:
        # for i in `seq 15 25`; do echo red$i; ssh red$i ls -l /mnt/nfs/; done

#### [ ] 3. Start up Hadoop HDFS.
    Login to red21 as root and run the following:
    # source /opt/profile.yarn-spark-bench2
    # /opt/hadoop/sbin/stop-dfs.sh
    # /opt/hadoop/sbin/start-dfs.sh
   Verify by running jps on each host:
    # for i in `seq 15 25`; do echo red$i; ssh red$i jps ; done
        - Look for the following:
            red21: NameNode,SecondaryNameNode
            all others: DataNode
   Also verify by listing the HDFS directories:
    # source /opt/profile.conductor-spark-bench2
        - Or use any other /opt/profile.<resource manager> to set HADOOP env vars
    # hadoop fs -ls /SparkBench/Terasort
   
   NOTE:
    To stop HDFS:
        /opt/hadoop/sbin/stop-dfs.sh
            - This will stop all HDFS-related daemons across all hosts

### Configuring and running job for particular RESOURCE Manager:

### Platform CONDUCTOR for Spark v2.1 (Spark 1.5.2)

#### [ ] 1. Create the Spark Instance Group (SIG)
    a) Create the following resource groups:
        ComputeHosts:
            - static: red[15-20,22-25]
            - 33 slots each
                There are 32 cores on each machine that we want to use for the executor's tasks
                However 1 slot must be allocated for the shuffle service but we don't want the number of tasks to be limited to 31 so we bump up the total to 33 slots
        SparkMasterRG:
            - static: red21
            - 1 slot
        SparkHistoryServerRG:
            - static: red21
            - 1 slot
        SparkDriversRG:
            - This one is not used since we run in client mode
    
    b) Create new top level consumer and allocate appropriate to the above resource groups
    
    c) Specify the execution user to 'lsfadmin'
    
    d) Specify the deployment directory to /opt/sparkConductor2
    
    d) Make sure to use "spark.eventLog.enabled true" since this cannot be configured using REST API.  See next step to make changes using REST API.

#### [ ] 2. Make changes to the Spark configuration:

    [Option 1]
    a) Stop the SIG from the WEBGUI
        From web browser go to https://red21.eng.platformlab.ibm.com:8443/platform
    b) Edit the configuration and Save
    c) Startup the SIG
    
    [Option 2]
    a) Make the change directly to the configuration files on host doing the job submission, which is red21.
        i) Edit the following files:
            /opt/sparkCondctor2/spark-1.5.2-hadoop-2.6/conf/spark-defaults.conf
            /opt/sparkCondctor2/spark-1.5.2-hadoop-2.6/conf/spark-env.sh
            
    [Option 3]
    Use the REST API:
    Source the environment
    # source /opt/profile.conductor-spark-bench2
    a) Stop the Spark Instance Group.
    b) Edit
        /opt/sparkConfig.json:
    -----
    {
      "sparkparameters":
      {
        "spark.executor.memory": "8g",
        "spark.eventLog.dir": "file:/mnt/nfs/conductor/spark-events",
        "spark.executor.cores": "8",
        "spark.port.maxRetries": "512",
        "spark.history.fs.logDirectory": "file:/mnt/nfs/conductor/spark-events",
        "SPARK_LOCAL_DIRS":  "/mnt/c/spark/data,/mnt/d/spark/data,/mnt/e/spark/data,/mnt/f/spark/data,/mnt/g/spark/data,/mnt/h/spark/data,/mnt/i/spark/data,/mnt/j/spark/data,/mnt/k/spark/data,/mnt/l/spark/data,/mnt/m/spark/data",
        "JAVA_HOME": "/usr/lib/jvm/java-1.7.0-openjdk/jre",
        "SPARK_EGO_EXECUTOR_SLOTS_MAX": "8",
        "SPARK_EGO_ENABLE_PREEMPTION": "false"
      }
    }
    -----
    
    Use the following to customize ports (if required): currently using default values
    -----
        "spark.ui.port": "4100",
        "spark.history.ui.port": "18092",
        "spark.shuffle.service.port": "7400",
        "spark.master.rest.port": "6100",
        "SPARK_MASTER_PORT": 7100",
        "SPARK_MASTER_WEBUI_PORT": "8100",
        "SPARK_EGO_LOGSERVICE_PORT": "28100"
    -----    
    
    c) To get the Spark instance group ID:
    https://red21.eng.platformlab.ibm.com:8643/platform/rest/conductor/v1/instances

    d) Run the following curl command to update the arguments (make sure to replace the Spark instance group ID with the correct one.
    
    # curl -u Admin:Admin -k -H "Content-Type:application/json" -H "Accept:application/json" -X PUT --data-binary @/opt/configConductorSpark.json https://red21.eng.platformlab.ibm.com:8643/platform/rest/conductor/v1/instances/251d15ed-773a-4cdc-8848-8209f6e61daf/configuration


#### [ ] 3. Restart Conductor:
        # source /opt/profile.conductor-spark-bench2
        
        In case of need for restart:
        # egosh user logon -u Admin -x Admin
        # egosh service stop all
        # egosh ego shutdown all
        
        # egosh ego start all
        
        
#### [ ] 4. Verify Conductor has started up.
        # source /opt/profile.conductor-spark-bench2
        # egosh user logon -u Admin -x Admin
        # egosh service list
        
#### [ ] 5. Start history server.
        It should already be started as part of SIG's startup.
        Manually stop and start:
        # egosh service stop SparkPerformance-sparkhs
        # egosh service start SparkPerformance-sparkhs
       Access the history server at http://red21:18091
       
#### [ ] 6. Restart HDFS and clear cache
        # /opt/hadoop/sbin/stop-dfs.sh
        # /opt/hadoop/sbin/start-dfs.sh
    
        # for i in `seq 15 25`; do echo red$i; ssh red$i "echo 3 > /proc/sys/vm/drop_caches"; done
       
#### [ ] 7. Start the test run for Conductor.
        # su - lsfadmin
        # bash
        # source /opt/profile.conductor-spark-bench2
        # cd /perf_test/perf_harness
       Run a short warmup run of 2-30-5 before 30-60-60
        # nohup ./step_up_multi_user.sh <steps> <offset> <iterations> spark://red21:7099 > /perf_test/perf_data/env_output.log 2>&1 &
            Where:
                steps: the number of users
                offset: delay in seconds before next user starts submission
                iterations: number of jobs submitted for each user
          Example with 30 users, 60s delay, 60 jobs submitted by each user:
          # nohup ./step_up_multi_user.sh 30 60 60 spark://red21:7099 > /perf_test/perf_data/env_output.log 2>&1 &
            
### Check STATUS of a current run
#### [ ] 1. Tailing one of the driver_output_<user>.log files.
        # cd /perf_test/perf_data
        # grep ERROR driver_output_*
            - Find any errors messages:
                For example, any "SparkUncaughtExceptionHandler" indicates a failed job due to akka configuration
         
#### [ ] 2. Check the number of directories created under HDFS output directory.
        # hadoop fs -ls /SparkBench/Terasort/Output/ | wc -l
            - There should be an output directory created for each Spark job submission.
            - The number should reach <steps> x <iterations>
            - Periodically running the command can give an estimate of the progress
            
#### [ ] 3. Check the number of files under spark-events directory (read by the history server).
        # ls -l /mnt/nfs/conductor/spark-events/|wc -l
            - There should always be a file for a successful job indicated by these lines:
                {"Event":"SparkListenerJobEnd","Job ID":0,"Completion Time":1454090875632,"Job Result":{"Result":"JobSucceeded"}}
                {"Event":"SparkListenerJobEnd","Job ID":1,"Completion Time":1454090888666,"Job Result":{"Result":"JobSucceeded"}}
            - The total number of files should equal <steps> x <iterations>
                - If there are less, this indicates there are failed jobs
                    # grep "JobSucceeded" /mnt/nfs/conductor/spark-events/*|wc -l
                        -There should be 3600 entries (2 jobs per application)
                
#### [ ] 4. Check for the executor memory size and cores:
        # for i in `seq 15 25`; do echo red$i; ssh red$i "ps -ef|grep executor" ; done
        
#### [ ] 5. Monitor the master host red21 by running 'nmon' interactively in another window.

### CLEANUP Sequence before each run

#### [ ] 0. Run Cleanup script to clean up all directories
    #/perf_test/perf_harness/clean_up_all_hosts.sh

#### [ ] 1. Stop all other Resource Managers.
    Login to red21.

    c) Conductor:
    # source /opt/profile.conductor-spark-bench2
    # egosh user logon -u Admin -x Admin
    # egosh service stop all
    # egosh ego shutdown all
    
#### [ ] 2. Stop any running history server.
    # ps -ef|grep HistoryServer
    # kill -9 <pid>
    
#### [ ] 3. Remove junk files on each host.
    Login to red21.        
    
#### [ ] (Optional) 4. If a test run fails (i.e. step_up_multi_user.sh is canceled while running)
    a) Run this script:
        # /perf_test/perf_data/clean_failed_run.sh
        
### Copy INPUT directory for Terasort
#### [ ] 1. Copy the Terasort input (2 GB, 40 partitions)
    hadoop fs -put /opt/input_data_for_HDFS/input-20000000-records-40-parts/ /SparkBench/Terasort/Input