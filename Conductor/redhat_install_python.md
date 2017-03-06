    Redhat depends on a specific version of Python for yum to function properly. Because of this the recommended way of python installing is an alt-install.
    
    A very good tutorial can be found here:
    
    A short summary of commands to run (from the previous link):
    yum groupinstall "Development tools"
    yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
    
    wget http://python.org/ftp/python/2.7.6/Python-2.7.6.tar.xz
    tar xf Python-2.7.6.tar.xz
    cd Python-2.7.6
    ./configure --prefix=/usr/local --enable-unicode=ucs4 --enable-shared LDFLAGS="-Wl,-rpath /usr/local/lib"
    make && make altinstall
    
    now install pip:

    wget https://bitbucket.org/pypa/setuptools/raw/bootstrap/ez_setup.py
    python2.7 ez_setup.py
    easy_install-2.7 pip