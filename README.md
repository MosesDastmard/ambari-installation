# Ambari Installation
## HDP2.5, HDP3.1
### Ambari installation on cluster

Here we will describe how to install Ambari on cluster of machines whether they are hosted by cloud-provider or you setup your cluster on single machine but several VMs using VMWare or Virtual Box. Following steps help you out with having Ambari installed on your cluster.

What you will do:

* Install Ambari(HDP) using command line
* Install Ambari on a cluster (Not a single machine)
* Cluster can b hosted on cloud (Recommended) or locally - if you want to run at least two VMs locally, make sure you have 15Gb free RAM each.
* You will install Ambari on CentOS6 - DON'T try other version of CentOS
* Take care not all the command should run on all machines, BE Carefull you run the righ command for different machines.
* We will call the machine run Ambari server, M1 and all the remain machine M2, M3, .... You will have two group of machines {M1, [M2, M3, ...]}


```command

       ####### Install CentOS 6.10
       Download centos image 6.10 DVD1 ~ 3.5GB
       Install Virtual Box
       Create VM CentOS
       Clone VM M0-M2
       M0 ~ Http Server
       M1,M2 Ambari

       ####### Http server
       yum upgrade openssh -y
       yum update openssh -y
       yum -y install httpd
       chkconfig httpd on
       service httpd start


       service iptables status
       service iptables stop

       mkdir /var/www/html/UTILS
       tar zxvf HDP-2.5.3.0-centos6-rpm.tar.gz -C /var/www/html 
       tar zxvf HDP-UTILS-1.1.0.21-centos6.tar.gz -C /var/www/html/UTILS/


       ####### Iptables Disable

       service iptables status
       service iptables stop

       ###### Edit sshd_config file

       file=/etc/ssh/sshd_config
       cp -p $file $file.old && awk ' $1=="PermitRootLogin" {$2="yes"} $1=="PasswordAuthentication" {$2="yes"} $1=="#PubkeyAuthentication" {$1="PubkeyAuthentication"} {print} ' $file.old > $file
       service sshd restart

       ##### SELinux Disable
       vi /etc/sysconfig/selinux or /etc/selinux/config
       SELINUX=disabled

       ##### Disabling Transparent Huge 

       echo never > /sys/kernel/mm/transparent_hugepage/enabled

       ####### Password-less SSH using public private key pair

       ssh-keygen -t rsa
       ssh-copy-id -i ~/.ssh/id_rsa.pub root@10.138.0.5
       ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.43.16


       ####### Mysql database setup for hive,ambari and oozie

       ###Install and configure MySQL server instance

       yum install mysql-server -y
       /etc/init.d/mysqld start
       mysqladmin -u root password 9903

       #### Setting up the databases
       mysqladmin -u root -p9903 create ambaridb
       mysqladmin -u root -p9903 create hivedb
       mysqladmin -u root -p9903 create ooziedb

       #### Configure users and permission:

       mysql -u root -p9903

       ### Create database owner for ambaridb and grant permission to the database
       USE ambaridb;
       CREATE USER 'ambari'@'localhost' IDENTIFIED BY 'ambari_password';
       GRANT ALL PRIVILEGES ON *.* TO 'ambari'@'localhost';
       CREATE USER 'ambari'@'%' IDENTIFIED BY 'ambari_password';
       GRANT ALL PRIVILEGES ON *.* TO 'ambari'@'%';
       FLUSH PRIVILEGES;

       ### Create database owner for hivedb and grant permission to the database
       USE hivedb;
       CREATE USER 'hive'@'localhost' IDENTIFIED BY 'hive_password';
       GRANT ALL PRIVILEGES ON *.* TO 'hive'@'localhost';
       CREATE USER 'hive'@'%' IDENTIFIED BY 'hive_password';
       GRANT ALL PRIVILEGES ON *.* TO 'hive'@'%';
       FLUSH PRIVILEGES;

       ### Create database owner for ooziedb and grant permission to the database
       USE ooziedb;
       CREATE USER 'oozie'@'localhost' IDENTIFIED BY 'oozie_password';
       GRANT ALL PRIVILEGES ON *.* TO 'oozie'@'localhost';
       CREATE USER 'oozie'@'%' IDENTIFIED BY 'oozie_password';
       GRANT ALL PRIVILEGES ON *.* TO 'oozie'@'%';
       FLUSH PRIVILEGES;

       ### Exit out of MySQL command shell:
       exit;

       ### Setup Ambari server to use ambaridb
       yum install mysql-connector-java* -y


       ####### Configure Ambari Repos 
       yum-config-manager --add-repo http://192.168.43.4/HDP/centos6/hdp.repo
       yum-config-manager --add-repo http://public-repo-1.hortonworks.com/HDP/centos6/2.x/updates/2.5.3.0/hdp.repo
       yum-config-manager --add-repo http://public-repo-1.hortonworks.com/ambari/centos6/2.x/updates/2.4.2.0/ambari.repo
       yum-config-manager --add-repo http://192.168.43.4/ambari.repo
       yum repolist

       ###### Install Ambari
       yum -y install ambari-server


       [root@host]# ambari-server setup
        Using python /usr/bin/python2.6
        Setup ambari-server
        Checking SELinux...
        SELinux status is 'disabled'
        Customize user account for ambari-server daemon [y/n] (n)?
        Adjusting ambari-server permissions and ownership...
        Checking firewall status...
        WARNING: iptables is running. Confirm the necessary Ambari ports are accessible. Refer to the Ambari documentation for more details on ports.
        OK to continue [y/n] (y)?
        Checking JDK...
        [1] OpenJDK 1.8.0
        [2] OpenJDK 1.7.0 (deprecated)
        [3] Custom JDK
        ==============================================================================
        Enter choice (1):
        Downloading JDK from http://birepo-build.svl.ibm.com/repos/IOP-UTILS/RHEL6/x86_64/1.1/openjdk/jdk-1.8.0.tar.gz to /var/lib/ambari-server/resources/jdk-1.8.0.tar.gz
        jdk-1.8.0.tar.gz... 100% (56.5 MB of 56.5 MB)
        Successfully downloaded JDK distribution to /var/lib/ambari-server/resources/jdk-1.8.0.tar.gz
        Installing JDK to /usr/jdk64/
        Successfully installed JDK to /usr/jdk64/
        Completing setup...
        Configuring database...
        Enter advanced database configuration [y/n] (n)? y
        Configuring database...
        ==============================================================================
        Choose one of the following options:
        [1] - PostgreSQL (Embedded)
        [2] - Oracle
        [3] - MySQL
        [4] - PostgreSQL
        ==============================================================================
        Enter choice (1): 3
        Hostname (localhost): 
        Port (3306): 
        Database name (ambari): ambaridb
        Username (ambari): ambari
        Enter Database Password (bigdata): 
        Re-enter password: 
        Configuring ambari database...
        Copying JDBC drivers to server resources...
        Configuring remote database connection properties...
        WARNING: Before starting Ambari Server, you must run the following DDL against the database to create the schema: /var/lib/ambari-server/resources/Ambari-DDL-MySQL-CREATE.sql
        Proceed with configuring remote database connection properties [y/n] (y)? y
        Extracting system views...
        ....ambari-admin-2.1.0_IBM_5.jar
        .
        Adjusting ambari-server permissions and ownership...
        Ambari Server 'setup' completed successfully.

       mysql -u ambari -pambari_password ambaridb <  /var/lib/ambari-server/resources/Ambari-DDL-MySQL-CREATE.sql
       ambari-server setup --jdbc-db=mysql --jdbc-driver=/usr/share/java/mysql-connector-java.jar


       ambari-server start

       hostname -f
       cat ~/.ssh/id_rsa

       ###### Create User in hdfs
       su - hdfs
       hadoop fs -mkdir /user/admin
       hadoop fs -chown admin:hadoop /user/admin
```
