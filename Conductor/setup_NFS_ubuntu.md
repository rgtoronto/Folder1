#### Nodes
    Host: 172.29.2.68
    Client: 172.29.2.67

### Download and Install
##### Host node
    sudo apt-get update
    sudo apt-get install nfs-kernel-server
    
##### Client node
    sudo apt-get update
    sudo apt-get install nfs-common
    
### Create the Share Directory on the Host Server
    sudo mkdir /var/nfs
    sudo chown nobody:nogroup /var/nfs

### Configure the NFS Exports on the Host Server
    sudo nano /etc/exports
    
    /home       172.29.2.68(rw,sync,no_root_squash,no_subtree_check)
    /var/nfs    172.29.2.67(rw,sync,no_subtree_check)
    - rw: This option gives the client computer both read and write access to the volume.
    - sync: This option forces NFS to write changes to disk before replying. This results in a more stable and consistent environment, since the reply reflects the actual state of the remote volume.
    - nosubtreecheck: This option prevents subtree checking, which is a process where the host must check whether the file is actually still available in the exported tree for every request. This can cause many problems when a file is renamed while the client has it opened. In almost all cases, it is better to disable subtree checking.
    - norootsquash: By default, NFS translates requests from a root user remotely into a non-privileged user on the server. This was supposed to be a security feature by not allowing a root account on the client to use the filesystem of the host as root. This directive disables this for certain shares.

    sudo exportfs -a
    sudo service nfs-kernel-server start
    
### Create the Mount Points and Mount Remote Shares on the Client Server
    sudo mkdir -p /mnt/nfs/home
    sudo mkdir -p /mnt/nfs/var/nfs
    
    sudo mount host_id:/home /mnt/nfs/home
    sudo mount host_id:/var/nfs /mnt/nfs/var/nfs
    
    df -h
    mount -t nfs
    
### Test NFS Access
    sudo touch /mnt/nfs/home/test_home
    sudo touch /mnt/nfs/var/nfs/test_var_nfs
    Look at the ownership of the file in the mounted home directory:
        ls -l /mnt/nfs/home/test_home
        -rw-r--r-- 1 root   root      0 Apr 30 14:43 test_home
        As you can see, the file is owned by root. This is because we disabled the root_squash option on this mount that would have written the file as an anonymous, non-root user.

    On our other test file, which was mounted with the root_squash enabled, we will see something different:

    ls -l /mnt/nfs/var/nfs/test_var_nfs
    -rw-r--r-- 1 nobody nogroup 0 Apr 30 14:44 test_var_nfs
    As you can see, this file was assigned to the "nobody" user and the "nogroup" group. This follows our configuration.

### Make Remote NFS Directory Mounting Automatic
    We can make the mounting of our remote NFS shares automatic by adding it to our fstab file on the client.

    Open this file with root privileges in your text editor:

    sudo nano /etc/fstab
    At the bottom of the file, we're going to add a line for each of our shares. They will look like this:

    1.2.3.4:/home    /mnt/nfs/home   nfs auto,noatime,nolock,bg,nfsvers=4,intr,tcp,actimeo=1800 0 0
    1.2.3.4:/var/nfs    /mnt/nfs/var/nfs   nfs auto,noatime,nolock,bg,nfsvers=4,sec=krb5p,intr,tcp,actimeo=1800 0 0
    The options that we are specifying here can be found in the man page that describes NFS mounting in the fstab file:

    man nfs
    This will automatically mount the remote partitions at boot (it may take a few moments for the connection to be made and the shares to be available).

### Unmount an NFS Remote Share
    If you no longer want the remote directory to be mounted on your system, you can unmount it easily by moving out of the share's directory structure and unmounting, like this:

    cd ~
    sudo umount /mnt/nfs/home
    sudo umount /mnt/nfs/var/nfs
    This will remove the remote shares, leaving only your local storage accessible:

    df -h
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/vda         59G  1.3G   55G   3% /
    none            4.0K     0  4.0K   0% /sys/fs/cgroup
    udev            2.0G   12K  2.0G   1% /dev
    tmpfs           396M  320K  396M   1% /run
    none            5.0M     0  5.0M   0% /run/lock
    none            2.0G     0  2.0G   0% /run/shm
    none            100M     0  100M   0% /run/user
    As you can see, our NFS shares are no longer available as storage space.
    
    
# setup NFS on redhat
    1. Basic NFS Configuration
    In this config will guide you trough a quick and basic configuration of NFS server on RHEL7 Linux system. We do not take any security concerns into the consideration, nor we will be concerned with fine tuning and access control. In our scenario we define two hosts:
    NFS Server, IP 10.1.1.100
    NFS Client, IP 10.1.1.18
    Assuming your already have a running Redhat 7 Linux system in order to setup NFS server you will need to install few additional packages:
    1.1. NFS Server configuration
    Run the below commands to begin the NFS Server installation:
    [nfs-server ]# yum install nfs-utils rpcbind
    Next we export some arbitrary directory called /opt/nfs. Create /opt/nfs directory:
    [nfs-server ]# mkdir -p /opt/nfs
    and edit /etc/exports NFS exports file to add the below line while replacing the IP address 10.1.1.18 with the IP address of your client:
    /opt/nfs 10.1.1.18(no_root_squash,rw,sync)
    Next make sure to enable 2049 port on your firewall to allow clients requests:
    [nfs-server ]# firewall-cmd --zone=public --add-port=2049/tcp --permanent
    [nfs-server ]# firewall-cmd --reload
    Start rpcbind daemon and NFS server in this order:
    [nfs-server ]# service rpcbind start; service nfs start
    Check the NFS server status:
    [nfs-server ]# service nfs status 
    nfs-server.service - NFS Server
       Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; disabled)
       Active: active (exited) since Thu 2014-12-11 08:12:46 EST; 23s ago
      Process: 2780 ExecStart=/usr/sbin/rpc.nfsd $RPCNFSDARGS $RPCNFSDCOUNT (code=exited, status=0/SUCCESS)
      Process: 2775 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=0/SUCCESS)
      Process: 2773 ExecStartPre=/usr/libexec/nfs-utils/scripts/nfs-server.preconfig (code=exited, status=0/SUCCESS)
     Main PID: 2780 (code=exited, status=0/SUCCESS)
       CGroup: /system.slice/nfs-server.service
    1.2. NFS Client configuration
    To be able to mount NFS exported directories on your client the following packages needs to be installed. Depending on your client's Linux distribution the installation procedure may be different. On Redhat 7 Linux the installation steps are as follows:
    [nfs-client ]# yum install nfs-utils rpcbind
    [nfs-client ]# service rpcbind start
    What remains is to create a mount point directory eg. /mnt/nfs and mount previously NFS exported /opt/nfs directory:
    [nfs-client ]# mkdir -p /mnt/nfs
    [nfs-client ]# mount 10.1.1.110:/opt/nfs /mnt/nfs/
    Test correctness of our setup between NFS Server and NFS client. Create an arbitrary file within NFS mounted directory on the client side:
    [nfs-client ]# cd /mnt/nfs/
    [nfs-client ]# touch NFS.test
    [nfs-client ]# ls -l
    total 0
    -rw-r--r--. 1 root root 0 Dec 11 08:13 NFS.test
    Move the the server side and check whether our newly NFS.test file exists:
    [nfs-server ]# cd /opt/nfs/
    [nfs-server ]# ls -l
    total 0
    -rw-r--r--. 1 root root 0 Dec 11 08:13 NFS.test
    2. Configuring permanent NFS mount
    Now that we have a basic NFS configuration on RHEL7 Linux system done, next we can add additional settings such as server persistence and permanent client mount using /etc/fstab. In order to have our NFS exports permanently available after the NFS server system reboot we need to make sure that nfs service starts after reboot:
    [nfs-server ]# systemctl enable nfs-server
    ln -s '/usr/lib/systemd/system/nfs-server.service' '/etc/systemd/system/nfs.target.wants/nfs-server.service'
    To allow client to mount NFS exported directory permanently after reboot we need to define a mount procedure within /etc/fstab config file. Open /etc/fstab file and add the following line:
    10.1.1.110:/opt/nfs	/mnt/nfs	nfs	defaults 		0 0
    3. Mount User Home Directory
    In the following steps we will export a user home directory /home/rhel7. Since NFS needs full access privileges to access /home/rhel7:
    [nfs-server ]# ls -ld /home/rhel7/
    drwx------. 2 rhel7 rhel7 59 Jul 17 14:22 /home/rhel7/
    we will bind it to a new directory:
    [nfs-server ]# mkdir -p /exports/rhel7
    [nfs-server ]# mount --bind /home/rhel7/ /exports/rhel7/
    To make the above permanent add the following line into your /etc/fstab file:
    /home/rhel7    /exports/rhel7   none    bind  0  0
    Next, add another export line into /etc/exports file:
    /exports/rhel7 10.1.1.18(no_root_squash,rw,sync)
    Re-export all NFS directories:
    [nfs-server ]# exportfs -ra
    What has left is to mount the above user directory using our client host:
    [nfs-client ]# mount 10.1.1.110:/exports/rhel7 /mnt/rhel7/
    [nfs-client ]# cd /mnt/rhel7/
    [nfs-client ]# ls
    [nfs-client ]# touch RHEL7-test-nfs
    [nfs-client ]# ls
    RHEL7-test-nfs
    Confirm that the file RHEL7-test-nfs exists on NFS server:
    # ls -l /home/rhel7/
    total 0
    -rw-r--r--. 1 root root 0 Dec 11 09:13 RHEL7-test-nfs