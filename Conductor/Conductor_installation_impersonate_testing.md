# Installation
## Steps
### link to find the latest available build for each product
    https://lweb.eng.platformlab.ibm.com/engr/pcc/release_eng/work/sym/sym_mainline/last/?C=M;O=A 
### Localtion   
    export CLUSTERADMIN=egoadmin
    export CLUSTERNAME=clusterguo
    /bin/bash /pcc/release_eng/work/sym/sym_mainline/last/conductor2.1.0.0_ppc64le.bin --prefix /opt/guo --dbpath /opt/guo/db
    
### Steps to setup
    1. source $EGO_TOP/profile.platform     where $EGO_TOP is the directory that you installed your Conductor package in, i.e. /opt/guo 
    2. cp /pcc/lsfqa-trusted/entitlement/key/conductor/2.1.0/conductor_entitlement.dat /home/guo 
        make a local copy of Conductor entitlement file
    3. switch user to cluster admin,         i.e. egoadmin
        #sudo su egoadmin
    4. egoconfig join $master_host           where $master_host is the host name that you want to Conductor master running on
    5. egoconfig setentitlement /home/guo/conductor_entitlement.dat
    6. switch to root user to assign ego cluster to start cluster
        exit

# How to start up Conductor cluster
    root# egosh ego start
        start Conductor cluster master
    egosh service list
        to check service status
    egosh service start all
        start all egosh service
    egosh service stop all
        stop all egosh service
    egosh ego shutdown
        shut down ego lim

# Setup Impersonate Feature
## Linux machine

### 1. GUI (yaml file to register AI)
        asc_template_version: 2015-10-30
        resources:
            service_activity:
                type: IBM::ASC::Activity
                properties:
                    os: [all]
                    commands:
                        start:
                            command: c:\curl\auth.bat
                        environmentvariables: #optional
                            EGO_REST_URL: https://asc-40.ma.platformlab.ibm.com:8543
        serviceA:
            type: IBM::ASC::Service
            properties:
                consumer:
                    name: ascA
                    executionuser: .\Administrator
                impersonate: Admin
                needcredential: false
                resourcegroup: ComputeHosts
                activity:
                    - { get_resource: service_activity }
### REST to register AI 
#### sample 1
    <appinstanceregparam>
      <consumerpath>/asc</consumerpath>
      <appinstancename>asctestwin</appinstancename>
      <parameters>
      </parameters>
      <apptemplateyaml>
        asc_template_version: 2015-10-30
        resources:
            service_activity:
                type: IBM::ASC::Activity
                properties:
                    os: [all]
                    commands:
                        start:
                            command: c:\curl\auth.bat
                        environmentvariables: #optional
                            EGO_REST_URL: https://asc-40.ma.platformlab.ibm.com:8543
        serviceA:
            type: IBM::ASC::Service
            properties:
                consumer:
                    name: ascA
                    executionuser: .\Administrator
                impersonate: Admin
                needcredential: false
                resourcegroup: ComputeHosts
                activity:
                    - { get_resource: service_activity }
        </apptemplateyaml>
    </appinstanceregparam>
#### sample 2
    <appinstanceregparam>
      <consumerpath>/asc</consumerpath>
      <appinstancename>asctest</appinstancename>
      <parameters>
      </parameters>
      <apptemplateyaml>
    asc_template_version: 2015-10-30
    resources:
        service_activity:
            type: IBM::ASC::Activity
            properties:
                commands:
                    start:
                        command: /home/guo/auth.sh
                        stdoutfile: /tmp/doo.txt
        serviceA:
            type: IBM::ASC::Service
            properties:
                consumer:
                    name: ascA
                    executionuser: root
                impersonate: Admin
                needcredential: false
                resourcegroup: ComputeHosts
                activity:
                    - { get_resource: service_activity }
        </apptemplateyaml>
    </appinstanceregparam>
### auth.sh script 
    #!/bin/sh
    
    url=$EGO_REST_URL
    sleep 1
    reply=`curl -k -XGET -H'Accept: application/json' -H"Authorization: PlatformToken token=$EGO_SERVICE_CREDENTIAL" $url/platform/rest/ego/auth/logon`
    
    echo "your EGO_SERVICE_CREDENTIAL is" $EGO_SERVICE_CREDENTIAL
    echo $reply | grep csrftoken
    if [ $? -eq 0 ]; then
        echo "logon successful"
    else
        echo "logon failed"
        exit
    fi
    
    csrftoken=${reply##*\:\"}
    csrftoken=${csrftoken%%\"\}}
    
    echo "csrftoken="$csrftoken
    
## Windows Machine
### Pre-condition:
#### download curl
    1. https://curl.haxx.se/download.html, choose curl 7.48.0 for Windows SSL to download
    2. Unzip the zip file then you can use curl on windows
### GUI yaml file to register AI
#### Sample 1
    asc_template_version: 2015-10-30
    parameters:
        serviceA_rg:
            type: string
            label: Resource Group for Service A
            description: Resource Group for Service A
            constraints:
               - custom_constraint: asc.getresourcegroup			
        serviceA_execuser:
           type: string
           label: Execution user for Service A
            description: User that Service A instances will run as.	
    resources:
        service_activity:
            type: IBM::ASC::Activity
            properties:
                os: [all]
                commands:
                    start:
                        command: C:\curl\auth.bat
                log_path: c:\asc.log
                environmentvariables: #optional
                    EGO_REST_URL: https://asc-40.ma.platformlab.ibm.com:8543
        serviceA:
           type: IBM::ASC::Service
            properties:
                consumer:
                    name: ascA
                    executionuser: { get_param: serviceA_execuser }
                needcredential: false
                resourcegroup: { get_param: serviceA_rg } # required
                activity:
                    - { get_resource: service_activity }
#### Sample 2
    asc_template_version: 2015-10-30
    resources:
        service_activity:
            type: IBM::ASC::Activity
            properties:
                os: [all]
                commands:
                    start:
                        command: c:\curl\auth.bat
                environmentvariables: #optional
                        EGO_REST_URL: https://asc-40.ma.platformlab.ibm.com:8543
        serviceA:
            type: IBM::ASC::Service
            properties:
                consumer:
                    name: ascA
                    executionuser: .\Administrator
                impersonate: Admin
                needcredential: false
                resourcegroup: ComputeHosts
                activity:
                    - { get_resource: service_activity }
### REST to register AI
#### Sample 1
    <appinstanceregparam>
      <consumerpath>/asc</consumerpath>
      <appinstancename>asctestwin</appinstancename>
      <parameters>
      </parameters>
      <apptemplateyaml>
    asc_template_version: 2015-10-30
    resources:
        service_activity:
            type: IBM::ASC::Activity
            properties:
    		    os: [all]
                commands:
                    start:
                        command: c:\curl\auth.bat
        serviceA:
            type: IBM::ASC::Service
            properties:
                consumer:
                    name: ascA
                    executionuser: ./Administrator
                impersonate: Admin
                needcredential: false
                resourcegroup: ComputeHosts
                activity:
                    - { get_resource: service_activity }
        serviceB:
            type: IBM::ASC::Service
            properties:
                consumer:
                    name: ascB
                    executionuser: ./Administrator
                impersonate: Admin
                needcredential: false
                resourcegroup: ComputeHosts
                activity:
                    - { get_resource: service_activity }
        </apptemplateyaml>
    </appinstanceregparam>
#### Sample 2
    <appinstanceregparam>
      <consumerpath>/asc</consumerpath>
      <appinstancename>asctestt</appinstancename>
      <parameters>
      </parameters>
      <apptemplateyaml>
    asc_template_version: 2015-10-30
    resources:
        service_activity:
            type: IBM::ASC::Activity
            properties:
                os: [all]
                commands:
                    start:
                        command: c:\curl\auth.bat
                environmentvariables: #optional
                        EGO_REST_URL: https://asc-40.ma.platformlab.ibm.com:8543
        serviceA:
            type: IBM::ASC::Service
            properties:
                consumer:
                    name: ascA
                    executionuser: .\Administrator
                impersonate: Admin
                needcredential: false
                resourcegroup: ComputeHosts
                activity:
                    - { get_resource: service_activity }
        </apptemplateyaml>
    </appinstanceregparam>
#### batch file to check impersonate logon successful
    @echo off
    SET url=%EGO_REST_URL%
    
    echo url is: %url% >> C:\\curl\doo.txt
    
    echo %EGO_SERVICE_CREDENTIAL% >> C:\\curl\doo.txt
    
    c:\curl\curl.exe -k -XGET -H "Accept: application/json" -H "Authorization: PlatformToken token=%EGO_SERVICE_CREDENTIAL%" %url%/platform/rest/ego/auth/logon >> c:\\curl\curl.txt
    echo %EGO_SERVICE_CREDENTIAL% >> C:\\curl\doo.txt
    
    findstr /m "csrftoken" c:\\curl\curl.txt
    if %errorlevel%==0 (
        echo "logon successful"  >> C:\\curl\doo.txt)
       exit
    else if (
       echo "logon failed"  >> C:\\curl\doo.txt)
    
    
    #csrftoken=%{reply##*\:\"}%
    #csrftoken=%{csrftoken%%\"\}%
    
    #echo "csrftoken="%csrftoken%
    
    exit

# Useful Information
## How to access Windows VM via a Linux machine
    1. using port forwarding:
    2. from putty, using ssh tunnel, setup local Source port: 3388
    3. Destination: your_remote_windows_host:3389
    4. click Add button
    5. click on open button
    6. From you local windows machine remote logon to remote windows vm machine
    7. Computer: localhost:3388
    8. hostname: the host name you try to access to
    9. Administrator:Letmein123


