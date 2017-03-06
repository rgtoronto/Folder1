# Install cfc build

This is a step by step guide to install IBM Spectrum Conductor for Containers (CfC) from the internal daily build.

## Most of all, this is only for rc 0.4 build, please open https://ibm.ent.box.com/v/containerDocs for stable document


## System requirements

* Operating System: Ubuntu 16.04+ and RHEL/CentOS 7.0+
* CPU architecture: amd64 and ppc64le
* Software: Docker engine >= 1.11.1, docker-py >= 1.7
* At least Three nodes, one for Boot/Master node, one for Proxy node, the other for Worker node
* All master and worker nodes must be able to access each other through the hostname/FQDN
* passwordless SSH access set up as root user. This must be set to allow the Boot node to access all Master and Worker nodes.
* Make sure the `vm.max_map_count` setting of master node is not less than 262144.
  * Run the command `sysctl -w vm.max_map_count=262144` to set it to 262144, and update the `vm.max_map_count` setting in `/etc/sysctl.conf` to set this value permanently.


## How to setup passwordless SSH access

On boot node, run folllowing commands:

```
# mkdir -p keys_dir
# ssh-keygen -t rsa -f keys_dir/ssh_key -P ''
# ssh-copy-id -i keys_dir/ssh_key root@master_nodes_ip
# ssh-copy-id -i keys_dir/ssh_key root@worker_nodes_ip
# ls keys_dir/ssh_key
```

And the file `keys_dir/ssh_key` is what you need in following steps.


## Before you begin

On all nodes in your cluster. Perform the following prerequisite steps:

1. Install Docker engine: <https://docs.docker.com/engine/installation/>
2. Install docker-py:

        apt install python-setuptools || yum install python-setuptools
        easy_install pip
        pip install docker-py

3. Ensure that all the nodes in your cluster can resolve the CfC Docker registry domain. On all nodes run:
   `ping    ma1dock1.platformlab.ibm.com`

   If a host is unable to ping `ma1demo1`, add the following line to the `/etc/hosts` file of the host.

    `9.21.49.147 ma1dock1.platformlab.ibm.com`
4. Ensure that all the nodes trust the CfC Docker registry. On all nodes run:

     `wget -O - http://ma1dock1.platformlab.ibm.com/static/trust-me | bash`


## Installing an IBM Spectrum Conductor for Containers cluster

The following steps must be run from the boot node as `root`.

#### Download the IBM Spectrum Conductor for Containers installer image

* For Linux 64-bit

        docker pull ma1dock1.platformlab.ibm.com/stable/cfc-installer:0.4.0

* For Linux Power 64-bit LE

        docker pull ma1dock1.platformlab.ibm.com/stable/cfc-installer-ppc64le:0.4.0

#### Extract the configuration files
This step should be run inside a designated working directory.

* For Linux 64-bit

        docker run --rm --net=host -e LICENSE=accept -v $(pwd):/data ma1dock1.platformlab.ibm.com/stable/cfc-installer:0.4.0 cp -r cluster /data

* For Linux Power 64-bit LE

        docker run --rm --net=host -e LICENSE=accept -v $(pwd):/data ma1dock1.platformlab.ibm.com/stable/cfc-installer-ppc64le:0.4.0 cp -r cluster /data


A cluster directory is created inside your current working directory. This cluster directory contains the following files:

* `ssh_key` - this file contains the SSH private key file that is used to login to your nodes. Ensure that you overwrite this file with the SSH private key for your cluster.

* `hosts` - For more information about hosts file, see [hosts file doc](https://github.ibm.com/platformcomputing/cfc-installer/blob/master/docs/hosts.md).

* `config.yaml` - For more information about config.yaml file, see [config file doc](https://github.ibm.com/platformcomputing/cfc-installer/blob/master/docs/config.yaml.md).

#### Modify the configuration files in the `cluster` directory to match your environment

For `hosts` file, add the master and worker IP addresses.

For `config.yaml` file, requires no modification unless the `flannel_network` and `service_cluster_ip_range` conflicts with the IP addresses of your nodes.

#### Deploy your environment

Run this command from inside your working directory. Your working directory is the directory that contains the `cluster` directory.

* For Linux 64-bit

        docker run --rm --net=host -e LICENSE=accept -v $(pwd)/cluster:/installer/cluster ma1dock1.platformlab.ibm.com/stable/cfc-installer:0.4.0 install

 install

* For Linux Power 64-bit LE

        docker run --rm --net=host -e LICENSE=accept -v $(pwd)/cluster:/installer/cluster ma1dock1.platformlab.ibm.com/stable/cfc-installer-ppc64le:0.4.0 install

 install

    The installer supports the following arguments:

    * `-l master` - deploys only on the master node
    * `-l worker` - deploys only on the worker nodes
    * `-l IP_ADDRESS` - deploys only on the specified node

#### Verify the status of your installation

* If the installation is successful, the final line of the messages that are returned from your installer contains the access information for your cluster.

* If you encounter errrors in your install, see the [Troubleshooting doc](https://github.ibm.com/platformcomputing/cfc-installer/blob/master/docs/trouble-shooting.md).


## Uninstalling

To uninstall an IBM Spectrum Conductor for Containers cluster. Run the following command from inside the working directory on your boot node:

* For Linux 64-bit

        docker run --rm --name=installer --net=host -e LICENSE=accept -v $(pwd)/cluster:/installer/cluster ma1dock1.platformlab.ibm.com/stable/cfc-installer:0.4.0 uninstall

* For Linux Power 64-bit LE

        docker run --rm --name=installer --net=host -e LICENSE=accept -v $(pwd)/cluster:/installer/cluster ma1dock1.platformlab.ibm.com/stable/cfc-installer-ppc64le:0.4.0 uninstall

<b>Note</b>: Uninstalling will not remove your Docker images.

## Additional Resources
   * The complete IBM Spectrum Conductor for Containers User Guide can be accessed here: https://ibm.box.com/v/containerDocs
   * Need more information? Connect with the CfC team on slack https://ibm-cloud-tech.slack.com/signup