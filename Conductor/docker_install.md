## Docker and docker-py install
### Ubuntu 16.04
#### Docker install script
    sudo apt-get install -y --no-install-recommends \
        apt-transport-https \
        ca-certificates \
        curl \
        software-properties-common
    	
    curl -fsSL https://apt.dockerproject.org/gpg | sudo apt-key add -
    
    apt-key fingerprint 58118E89F3A912897C070ADBF76221572C52609D
    
    sudo add-apt-repository \
           "deb https://apt.dockerproject.org/repo/ \
           ubuntu-$(lsb_release -cs) \
           main"
    	   
    sudo apt-get update
    
    sudo apt-get -y install docker-engine
    
#### install docker-py
    #apt install python-setuptools || yum install python-setuptools -y
    apt install python-setuptools || apt-get install python-setuptools -y
    easy_install pip -y 
    pip install docker-py -y 

###### RHEL 7.2
    under construction... :)