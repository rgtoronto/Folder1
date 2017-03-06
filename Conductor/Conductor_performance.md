# Environment
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
    
    
# iperf network checking
## Steps
### 1. Download iperf
    
### 2. start iperf server on master node
    iperf -s
### 3. run following script to test on single node:
    HOSTS=`cat /perf_test/perf_harness/ping_list`
    #Use when zookeeper is running
    #MASTER=$(mesos-resolve `cat /etc/mesos/zk` 2>/dev/null)
    #Use in the absence of zookeeper
    MASTER=`cat /perf_test/perf_harness/master_list`
    
    for LINE in $HOSTS
    do
          nohup ssh root@$LINE "iperf -c $MASTER"
    
          sleep 1
    done
### 4. run following script to test on saturation:
    HOSTS=`cat /perf_test/perf_harness/ping_list`
    #Use when zookeeper is running
    #MASTER=$(mesos-resolve `cat /etc/mesos/zk` 2>/dev/null)
    #Use in the absence of zookeeper
    MASTER=`cat /perf_test/perf_harness/master_list`
    
    for LINE in $HOSTS
    do
          nohup ssh root@$LINE "iperf -c $MASTER" &
    
          sleep 1
    done
# FIO disk checking
## Steps
### Make a local folder /opt/fio
    mkdir /opt/fio
### Create following files with scripts
    1. fiobench.sh
    2. seq_cache_read.fio
    3. seq_cache_write.fio

### Run following command on each slave node:
    1. cd /opt/fio
       ./fiobenchfinal.sh
        python ./parseme.py     
    2. get report file from /home/guo/red_
        
### Run following command on each salve node:
        1. cd /home/guo
           ./fiowrapper.sh
           python ./parse_concurrent.py
        2. get report file from /home/guo/red_cluster_concurrent.out
# useful scripts:
## copy hosts file
    root@mesos-10:~/perf_utilities# cat copy_hosts.sh
    #!/bin/bash
    for i in {1..9}
    do
        slave="mesos-0$i"
        echo "Copying hosts to ${slave}"
        scp /etc/hosts root@${slave}:/etc
    done
    echo "Complete!"
    exit 1
# IBM Box
    https://ibm.box.com/s/9p13nz8t1z51ve6slsei70q1ipp9a3ow

# Conductor installation and configuration

# To access red cluster with short host name
        To configure a connection-specific DNS name
        Click Start, click Control Panel, and then click Classic View.
        Double-click Network and Sharing Center.
        In the Network and Sharing Center window, click Manage network connections.
        In the Network Connections window, right-click the connection that you want to manage, and then click Properties.
        Do one of the following:
            To configure IP version 4 (IPv4) for the connection, perform the following steps:
                Click Internet Protocol Version 4 (TCP/IPv4), and then click Properties.
                Click Advanced, and then click the DNS tab.
                In DNS suffix for this connection, type the connection-specific DNS suffix.
            To configure IP version 6 (IPv6) for the connection, perform the following steps:
                Click Internet Protocol Version 6 (TCP/IPv6), and then click Properties.
                Click Advanced, and then click the DNS tab.
                In DNS suffix for this connection, type the connection-specific DNS suffix.
        Click OK twice, and then click Close.
# Steps to install Conductor Spark cluster on RED machine
## Master node
    export CLUSTERADMIN=lsfadmin
    export CLUSTERNAME=clusterPerf2GM
    /pcc/release_eng/work/sym/sym7.1.2/RC5/conductorspark2.1.0.0_x86_64.bin --prefix /opt/conductorPerf2GM --dbpath /opt/conductorPerf2GM
    source /opt/conductorPerf2GM/profile.platform
    sudo su lsfadmin
    source /opt/conductorPerf2GM/cshrc.platform
    egoconfig join red21
    cp /pcc/lsfqa-trusted/entitlement/key/conductor/2.1.0/conductor_spark_entitlement.dat /home/guo
    egoconfig setentitlement /home/guo/conductor_spark_entitlement.dat
    sudo su
    egosetsudoers.sh
    sudo su lsfadmin
    source /opt/conductorPerf2GM/cshrc.platform
    egosh ego start
## Slave node
    export CLUSTERADMIN=lsfadmin
    export CLUSTERNAME=clusterPerf2GM
    /pcc/release_eng/work/sym/sym7.1.2/RC5/conductorspark2.1.0.0_x86_64.bin --prefix /opt/conductorPerf2GM --dbpath /opt/conductorPerf2GM
    source /opt/conductorPerf2GM/profile.platform
    sudo su lsfadmin
    source /opt/conductorPerf2GM/cshrc.platform
    egoconfig join red21
    sudo su
    egosetsudoers.sh
    sudo su lsfadmin
    source /opt/conductorPerf2GM/cshrc.platform
    egosh ego start