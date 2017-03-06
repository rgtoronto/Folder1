# Install Notice for using derbydb:
    before installing Conductor for report testing, we have to export DERBY_DB_HOST value

# Enabling the Derby database
## Procedure    
    http://www-03preprod.ibm.com/support/knowledgecenter/SS4H63_2.1.0/shared_files/reports_database_move_sym.html

# Report testing on DB2, SQL Server and Oracle

## Database Resource Summary:
### Oracle Enterprise Edition 12c 
    host: asc-25.ma.platformlab.ibm.com
    port: 1521
    user: system
    password: Letmein123
    PS C:\oracle> sqlplus system/Letmein123@asc-25.ma.platformlab.ibm.com/asctest @C:\oracle\egodata.sql USERS USERS
    PS C:\oracle> sqlplus system/Letmein123@asc-25.ma.platformlab.ibm.com/asctest @C:\oracle\create_schema.sql USERS USERS
    jdbc:oracle:thin:@//asc-25.ma.platformlab.ibm.com:1521/asctest
    
### SQL Server 2012
    host: asc-25.ma.platformlab.ibm.com
    port: 1433
    user: sa
    password: Letmein123
    PS C:\sql> sqlcmd -U sa -P Letmein123 -d asctest -i .\egodata.sql
    PS C:\sql> sqlcmd -U sa -P Letmein123 -d asctest -i .\create_schema.sql
    jdbc:sqlserver://asc-25.ma.platformlab.ibm.com:1433;databaseName=asctest

### DB2 Enterprise Edition 10.5 Express-C
    host: asc-22.ma.platformlab.ibm.com
    port: 50000
    user: db2admin
    password: Admin123
    clpplus -nw db2admin/Admin123@localhost:50000/julia @create_schema.sql julia juliatablespace
    clpplus -nw db2admin/Admin123@localhost:50000/julia @egodata.sql julia juliatablespace
    jdbc:db2://asc-22.ma.platformlab.ibm.com:50000/julia:currentSchema=JULIA;
    
## Pre-condition:
### 1. Disable derbydb, plc and purger and automatic startup of reporting services

    egosh service stop derbydb plc purger

### 2. Script to manage Tables/Schemas
#### Remove existing schema for tables
    clpplus -nw db2admin/Admin123@localhost:50000/julia @c:\test\drop_schema.sql julia
#### Drop all tables
    clpplus -nw db2admin/Admin123@localhost:50000/julia @c:\test\egodroptable.sql julia
#### Create tables
    clpplus -nw db2admin/Admin123@localhost:50000/julia @create_schema.sql julia juliatablespace
#### Creat schema for tables
    clpplus -nw db2admin/Admin123@localhost:50000/julia @egodata.sql julia juliatablespace

### 3. setup configure files    
#### 3.1. need to add the following to $EGO_TOP/gui/conf/server_gui.xml file:
    DB2: db2jcc.jar
    SQL Server: sqljdbc.jar
    ORACLE: ojdbc4.jar
    
    3.1.1 copy driver file to the following two locations:
        $EGO_TOP/perf/version_number/lib
        $EGO_TOP/gui/version_number/lib
        
    3.1.2 Modify server_gui.xml file under $EGO_CONFDIR/../../gui/conf/
##### DB2
    <library id="DB2JCCLib">
       <fileset dir="/opt/guo/perf/3.4/lib" includes="db2jcc.jar db2jcc_license_cu.j                                                                                                            ar"/>
    </library>
    <dataSource id="db2" jndiName="jdbc/db2">
        <jdbcDriver libraryRef="DB2JCCLib"/>
        <properties.db2.jcc databaseName="app"
         serverName="asc-15.ma.platformlab.ibm.com" portNumber="50000" />
    </dataSource>
##### ORACLE            
    <library id="ORACLEJCCLib">
            <fileset dir="/opt/guo/perf/3.4/lib" includes="ojdbc7.jar"/>
    </library>
    <dataSource id="oracle" jndiName="jdbc/oracle">
        <jdbcDriver libraryRef="ORACLEJCCLib"/>
        <properties.oracle.jdbc databaseName="asctest"
         serverName="asc-25.ma.platformlab.ibm.com" portNumber="1521" />

##### SQLServer            
    <library id="SQLSERVERJCCLib">
       <fileset dir="/opt/guo/perf/3.4/lib" includes="sql.jar db2jcc_license_cu.j                                                                                                            ar"/>
    </library>
    <dataSource id="sqlserver" jndiName="jdbc/sqlserver">
        <jdbcDriver libraryRef="SQLSERVERJCCLib"/>
        <properties.db2.jcc databaseName="asctest"
         serverName="asc-25.ma.platformlab.ibm.com" portNumber="1433" />
    </dataSource>

#### 3.2. configure database connection:
        $EGO_TOP/perf/version_number/bin/dbconfig.sh -console
        Configure your database connection.
        Data source name: ReportDB
        User ID: [db2admin]
        Password: [********]Admin123
        JDBC driver class name:
          0 - oracle.jdbc.driver.OracleDriver
          1 - com.microsoft.sqlserver.jdbc.SQLServerDriver
          2 - com.vertica.jdbc.Driver
          3 - org.apache.derby.jdbc.ClientDriver*
          4 - org.gjt.mm.mysql.Driver
          5 - com.ibm.db2.jcc.DB2Driver*
          6 - org.postgresql.Driver
        To select an item, enter its number. Or press 7 to type a driver not in the list: [5] 5
        URL of the JDBC connection : [jdbc:db2://asc-22.ma.platformlab.ibm.com:50000/julia:currentSchema=JULIA;]
        Maximum connections : [30]
        Press 1 to Test connection, 2 to Save and exit, 3 to Redisplay, 4 to Cancel: [1] 1
        Successfully connected to database.
        Press 1 to Test connection, 2 to Save and exit, 3 to Redisplay, 4 to Cancel: [1] 2
        Configuration has been sucessfully saved.

### 4. Start plc purger service
### 5. restart WEBGUI service

### tips:
#### does you $EGO_TOP/perf/conf/datasource.xml have uncommented data source configuration after you installation
		 <ds:DataSource Name="ReportDB" 
		 		 		 		 Driver="org.apache.derby.jdbc.ClientDriver" 
		 		 		 		 Connection="jdbc:derby://RHEL7-145.cn.ibm.com:1527/app" 
		 		 		 		 Default="true" 
		 		 		 		 Cipher="aes128"
		 		 		 		 UserName="GxgOlootX4vHFcBTQOl1tg==" 
		 		 		 		 Password="GxgOlootX4vHFcBTQOl1tg=="/>	
#### check if database driver is listed in the file below:
    $EGO_TOP/perf/conf/driver.properties
    
#### Troubleshooting on cannot connect to DB
    1. restart database service
    2. Oracle:
        alter database mount;  
        alter system set db_recovery_file_dest_size=<>G scope=both;  
        alter database open; 