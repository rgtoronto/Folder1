# =====================================
# Spark Performance Benchmark Checklist
# =====================================

## ------------------------
## Environment Preparation:
## ------------------------
### 1. Make sure to check that the network is connected to the 10GB ethernet switch
### 2. Make sure to do a format on all disks /dev/sdc to /dev/sdm

### -------------
### After REBOOT:
### -------------
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
    # source /opt/profile.yarn-spark-bench
    # /opt/hadoop/sbin/stop-dfs.sh
    # /opt/hadoop/sbin/start-dfs.sh
   Verify by running jps on each host:
    # for i in `seq 15 25`; do echo red$i; ssh red$i jps ; done
        - Look for the following:
            red21: NameNode,SecondaryNameNode
            all others: DataNode
   Also verify by listing the HDFS directories:
    # source /opt/profile.conductor-spark-bench
        - Or use any other /opt/profile.<resource manager> to set HADOOP env vars
    # hadoop fs -ls /SparkBench/Terasort
   
   NOTE:
    To stop HDFS:
        /opt/hadoop/sbin/stop-dfs.sh
            - This will stop all HDFS-related daemons across all hosts

### ------------------------------------------------------------
### Configuring and running job for particular RESOURCE Manager:
### ------------------------------------------------------------

### -----------------------------------------------
### Apache Hadoop Yarn v2.6 (Spark 1.5.2)
### -----------------------------------------------

#### [ ] 1. Make configuration changes in Yarn.
    Login to red21:
    a) Edit /opt/hadoop/etc/hadoop/yarn-site.xml
    b) Copy to all hosts.
        # for i in `seq 15 25`; do echo red$i; scp -r /opt/hadoop/etc/hadoop/yarn-site.xml red$i:/opt/hadoop/etc/hadoop/ ; done
        
#### [ ] 2. Make configuration changes in Spark on Yarn
    Login to red21:
    a) Edit
        # vim /opt/spark_yarn/yarn_conf/spark-defaults.conf
        # vim /opt/spark_yarn/yarn_conf/spark-env.sh
    b) Copy to all hosts.
        # for i in `seq 15 25`; do echo red$i; scp -r /opt/spark_yarn/yarn_conf/spark-defaults.conf red$i:/opt/spark_yarn/yarn_conf ; done
        # for i in `seq 15 25`; do echo red$i; scp -r /opt/spark_yarn/yarn_conf/spark-env.sh red$i:/opt/spark_yarn/yarn_conf ; done
    
#### [ ] 3. Restart Yarn:
        # source /opt/profile.yarn-spark-bench
        # /opt/hadoop/sbin/stop-yarn.sh
        # /opt/hadoop/sbin/start-yarn.sh
        
#### [ ] 4. Verify services have started:
        # for i in `seq 15 25`; do echo red$i; ssh red$i jps ; done
        - Look for the following:
            red21: ResourceManager
            all other hosts: NodeManager
            
#### [ ] 5. Start history server for Yarn.
        # /opt/spark-1.5.2-bin-hadoop2.6/sbin/start-history-server.sh /mnt/nfs/yarn/spark-events
       To access the history server:
        Open web browser to http://red21:28082
        
#### [ ] 6. Restart HDFS and clear cache
        # /opt/hadoop/sbin/stop-dfs.sh
        # /opt/hadoop/sbin/start-dfs.sh
    
        # for i in `seq 15 25`; do echo red$i; ssh red$i "echo 3 > /proc/sys/vm/drop_caches"; done
        
#### [ ] 7. Start the test run for Yarn.
        # su - lsfadmin
        # bash
        # source /opt/profile.yarn-spark-bench
        # cd /perf_test/perf_harness
       Run a short warmup run of 2-30-5 before 30-60-60
        # nohup ./step_up_multi_user.sh <steps> <offset> <iterations> yarn-client > /perf_test/perf_data/env_output.log 2>&1 &
            Where:
                steps: the number of users
                offset: delay in seconds before next user starts submission
                iterations: number of jobs submitted for each user
          Example with 30 users, 60s delay, 60 jobs submitted by each user:
          # nohup ./step_up_multi_user.sh 30 60 60 yarn-client > /perf_test/perf_data/env_output.log 2>&1 &

### -----------------------------
### Check STATUS of a current run
### -----------------------------
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
        # ls -l /mnt/nfs/yarn/spark-events/|wc -l
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


         
### ----------------
### CLEANUP Sequence
### ----------------

#### [ ] 0. Run Cleanup script to clean up all directories
    #/perf_test/perf_harness/clean_files_and_dirs.sh

#### [ ] 1. Stop all other Resource Managers.
    Login to red21.

    a) Yarn:
    # source /opt/profile.yarn-spark-bench
    # /opt/hadoop/sbin/stop-yarn.sh

#### [ ] 2. Stop any running history server.
    # ps -ef|grep HistoryServer
    # kill -9 <pid>
    
#### [ ] 3. Remove junk files on each host.
    Login to red21.        
    
#### [ ] (Optional) 4. If a test run fails (i.e. step_up_multi_user.sh is canceled while running)
    a) Run this script:
        # /perf_test/perf_data/clean_failed_run.sh
        
### ---------------------------------
### Copy INPUT directory for Terasort
### ---------------------------------
#### [ ] 1. Copy the Terasort input (2 GB, 40 partitions)
    hadoop fs -put /opt/input_data_for_HDFS/input-20000000-records-40-parts/ /SparkBench/Terasort/Input