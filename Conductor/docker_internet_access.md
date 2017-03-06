## Ubuntu 14.04 LTS
    add export http_proxy="http://proxy-ma.platformlab.ibm.com:3128" in /etc/default/docker

## Ubuntu 16.04 LTS

    1. vim /etc/systemd/system/multi-user.target.wants/docker.service
    2. add 
        Environment="HTTP_PROXY=http://proxy-ma.platformlab.ibm.com:3128"
        Environment="HTTPS_PROXY=http://proxy-ma.platformlab.ibm.com:3128"
