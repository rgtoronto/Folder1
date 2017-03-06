# Here is steps and cmd/body you may need:

## 1. Add ipython type:
    curl -X POST -u Admin:Admin -H "Content-type: multipart/form-data" -k -L -F "file=@/home/lixieli/ipython-321-security-jpmc.tar.gz" "https://asc-10.ma.platformlab.ibm.com:8643/platform/rest/conductor/v1/notebooktypes?name=ipython&version=3.2.1&precmd=./scripts/prestart_ipython.sh&startcmd=./scripts/start_ipython.sh&stopcmd=./scripts/stop_ipython.sh&controlwaitperiod=10&jobmonitor=./scripts/jobMonitor.sh&jobmonitormaxupdateinterval=60&env=a=b;c=d&defaultbaseport=8080"

    Package : /home/lixieli/ipython-321-security-jpmc.tar.gz

## 2. Create SIG:

      <conductorinstanceregparam>
      <consumerpath>/notebookConsumer3</consumerpath>
      <conductorinstancename>notebookSIG3</conductorinstancename>
      <parameters>
        <driver_consumer_param>spark</driver_consumer_param>
        <executor_consumer_param>spark</executor_consumer_param>
        <sparkms_batch_consumer_param>spark</sparkms_batch_consumer_param>
        <sparkms_notebook_consumer_param>spark</sparkms_notebook_consumer_param>
        <sparkhs_consumer_param>spark</sparkhs_consumer_param>
        <sparkms_batch_rg_param>notebookRg3</sparkms_batch_rg_param>
        <sparkms_notebook_rg_param>notebookRg3</sparkms_notebook_rg_param>
        <sparkhs_rg_param>notebookRg3</sparkhs_rg_param>
        <driver_rg_param>notebookRg3</driver_rg_param>
        <executor_rg_param>notebookRg3</executor_rg_param>
        <execution_user>root</execution_user>
        <deploy_home>/root/spark/notebookSIG3</deploy_home>
      </parameters>
      <sparkparameters>
        <SPARK_MASTER_PORT>7077</SPARK_MASTER_PORT>
        <SPARK_MASTER_WEBUI_PORT>8090</SPARK_MASTER_WEBUI_PORT>
        <spark.master.rest.port>6066</spark.master.rest.port>
        <spark.history.ui.port>18080</spark.history.ui.port>
        <spark.shuffle.service.port>7337</spark.shuffle.service.port>
      </sparkparameters>
      <sparkversion>1.5.2</sparkversion>
      <notebooks>
       <notebookparam>
        <name>Zeppelin</name>
        <version>0.5.6</version>
        <consumer>notebooks</consumer>
        <executionuser>root</executionuser>
        <rg>notebookrg</rg>
        <deploydir>/tmp/zeppelinbuildin</deploydir>
        <basedatadir>/tmp/zeppelinbuildin/data</basedatadir>
      </notebookparam>
      </notebooks>
    </conductorinstanceregparam>


## 3. Deploy, start the SIG. Assign notebook to user.

## 4. Submit job

# Batch

     --class org.apache.spark.examples.SparkPi --deploy-mode cluster /opt/sparkCondctor2/spark-1.5.2-hadoop-2.6/lib/spark-examples-1.5.2-hadoop2.6.0.jar 10000 
     --class org.apache.spark.examples.SparkPi --deploy-mode cluster /opt/sparkgpfs/spark-1.6.1-hadoop-2.6/lib/spark-examples-1.6.1-hadoop2.6.0.jar 10000
     --class org.apache.spark.examples.JavaWordCount --deploy-mode cluster /opt/sparkgpfs/spark-1.6.1-hadoop-2.6/lib/spark-examples-1.6.1-hadoop2.6.0.jar /gpfs/conductorFS/mytext.txt

# Zeppelin:

    val count = sc.parallelize(1 to 100000).map{i =>
      val x = Math.random()
      val y = Math.random()
      if (x*x + y*y < 1) 1 else 0
    }.reduce(_ + _)
    println("Pi is roughly " + 4.0 * count / 10)

# Ipython:

    import random
    def sample(p):
        x, y = random.random(), random.random()
        return 1 if x*x + y*y < 1 else 0
    
    count = sc.parallelize(xrange(0, 10)).map(sample) \
                 .reduce(lambda a, b: a + b)
    print "Pi is roughly %f" % (4.0 * count / 10)

## 5.check point:


    1) SIG started successfully. Impersonate is defined to user B in spark and notebook master service profile

       2)Batch, zeppelin, ipython jobs are submitted successfully
         Batch. application submit return succeed on GUI. Application status is correct (waiting > running > finish)
         zeppelin/ipython. application get pi result on GUI. Application status is correct (waiting > running )

       3)Job allocation is assigned to user B. 
        After job submission, there will be activity created by spark. View its allocation. ensure the allocation user is impersonate.


       [root@asc-11 install]# egosh alloc view 17
    -------------------------------------------------------------------
    Allocation ID    : 17
    Allocation Client: EGO_SERVICE_CONTROLLER
    Allocation User  : Admin
    
    ALLOCATION REQUEST:
    Allocation Name : plc
    Consumer        : /ManagementServices/EGOManagementServices
    Resource Group  : ManagementHosts
    Requirement     : select('LINUXPPC64'||'LINUXPPC64LE'||'LINUX86'||'X86_64'||'IA64'||'NTX86'||'NTX64'||'SOLX8664'||'SOL64')
    MINSLOTS MAXSLOTS EXCLUSIVE TILE
    0        1        No        1


    ALLOCATED RESOURCE:
    RESOURCE                                                      ALLOCATED OCCUPY
    asc-10.ma.platformlab.ibm.com                                 1         1
    
    BLOCKED RESOURCE LIST:
    
    ACTIVITY LIST:
    54
