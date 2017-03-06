### CfC SVT lab env setup on X86
#### 1. Hardware and software
##### 1.1 OS
    Red Hat Enterprise Linux 7.2
##### 1.2 Hardware
    303 2-vCPU VMs
      3 masters (HA): 16G memory, HD 200G
      300 workers: 4G memory, HD 100G

#### 2. Setting up passwordless SSH among master and worker nodes
    Before installing the IBM Spectrum Conductor for Containers you must generate SSH key pairs on your machines and set up passwordless SSH.
    Note:Root user must be used to generate SSH keys and set up passwordless SSH.
    
#### 3. All cluster nodes are able to access each other(including itself)  using hostname/FQDN
    Ensure that all master, worker, and proxy nodes are able to access each other using their host name or fully qualified domain name (FQDN). 
    Host names must be in all lowercase letters.
    
#### 4. Ensure that clocks are synchronized across all nodes in the cluster

#### 5. master.cfc is a built-in domain for the IBM Spectrum Conductor for Containers cluster. Ensure that this domain is not being used by any of nodes

#### 6. On master nodes, ensure that the vm.max_map_countsetting is at least 262144.
    set this minimum vm.max_map_countvalue, run: sysctl -w vm.max_map_count=262144
    
#### 7. Docker installation on each node in cluster and setup as system service
    the latest version of docker
    see https://docs.docker.com/engine/installation/.
    
#### 8. install docker-py
    On master, worker, and proxy nodes, install docker-py.
    For Ubuntu:
    #apt install python-setuptools
    #easy_install pip
    #pip install 'docker-py>=1.7.0'
    For RHEL:
    #yum install python-setuptools
    #easy_install pip
    #pip install 'docker-py>=1.7.0' 
    
#### 9. Default ports should be open and avaiable
    https://ibm.ent.box.com/v/containerDocs#%5B%7B%22num%22%3A273%2C%22gen%22%3A0%7D%2C%7B%22name%22%3A%22XYZ%22%7D%2Cnull%2Cnull%2Cnull%5D
    
#### 10. pre-pulled docker images(workers only):
    nginx:  nginx:1.8.1
    tomcat: tomcat:9.0
    redis:  liuhougangxa/redis-3.2.0:latest
            redis:3.2.0
    debian: debian:jessie