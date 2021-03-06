How to install Hatohol on CentOS 7 (x86_64) with yum repository
=================================================================

Installation of needed packages
-------------------------------
### Register the yum repository of Project hatohol.
Use the following command to save the repo file to /etc/yum.repos.d/ directory.

    # wget -P /etc/yum.repos.d/ http://project-hatohol.github.io/repo/hatohol-el7.repo

### Register the yum repository of EPEL.
Use the following command to add EPEL repository.

    # yum install epel-release

### Install Hatohol Server
Use the following command to install Hatohol Server.

    # yum install hatohol-server

### Install Hatohol Web Frontend
Use the following command to install Hatohol Web.

    # yum install hatohol-web

### Install dependent packages
The following packages are installed automatically by the above commands.

- Packages provided from repository of CentOS
    - glib2
    - libsoup
    - sqlite
    - mysql
    - libuuid
    - MySQL-python
    - httpd
    - mod_wsgi
    - python-argparse
    - python-pip
    - json-glib
    - Django
- Package provided from repository of EPEL
    - librabbitmq

### Runtime dependent packages
The following packages are not installed automatically.

- Packages provided from repository of CentOS
    - mariadb-server

Use the following command to install them.

    # yum install mariadb-server

Setup
-----
### Setup of MariaDB server
If you have already used MariaDB server and use it, you can omit following steps.
Start MariaDB server and enable automatic start up on the machine boot.

    # systemctl enable mariadb
    # systemctl start mariadb

### Initialization of Hatohol DB
Use the following command to initialize Hatohol DB

    $ hatohol-db-initiator --db-user <User name of MariaDB root user> --db-password <User password of MariaDB root user>

Tips:

- If the root password of the MariaDB server is not set, use "".
- You can change the DB name, user name and the password of the created DB with --hatohol-db-user and --hatohol-db-password options.
    - You can change them to fix the hatohol.conf as the following. hatohol.conf is putted on ${prefix}/etx/hatohol/hatohol.conf.

```
[mysql]
- database=hatohol
+ database=your DB name
- user=hatohol
+ user=user name of MariaDB root user
- password=hatohol
+ password= password of MariaDB root user

[FaceRest]
workers=4

** NOTE **
The mark '+' at the head means a newly added line.
The mark '-' at the head means a deleted line.

```

- Since 15.03, hatohol-db-initiator doesn't require command line argument after hatohol DB is created. db-name, db-user and db-password are read from hatohol.conf by default.

### Setup of Hatohol Web Frontend
- Prepare a DB for Hatohol Web Frontend

Use the following commands in the MySQL command line tool to create DB and user.

    MariaDB> CREATE DATABASE hatohol_client DEFAULT CHARACTER SET utf8;
    MariaDB> GRANT ALL PRIVILEGES ON hatohol_client.* TO hatohol@localhost IDENTIFIED BY 'hatohol';

- Add tables into the DB

Use the following command to add tables used by Hatohol Web Frontend into the DB.

    # /usr/libexec/hatohol/client/manage.py syncdb

### Set up Hatohol Arm Plugin Interface 2 setting
You should setup RabbitMQ and HAP2 to retrieve data from Zabbix, Nagios, and Ceilometer etc. via Hatohol Arm Plugin 2(HAP2).
Please refer to the following document:

 - https://github.com/project-hatohol/hatohol/blob/master/doc/server/hap2/HowToUse.md

### Start of Hatohol server

    # systemctl enable hatohol
    # systemctl start hatohol

### Start of Hatohol Web

    # systemctl enable httpd
    # systemctl start httpd

Access with a web browser
------------------------
### Check of the setting of firewall and SELinux
By default, some security mechanisms such as SELinux and firewall block the access from other computers.
You have to deactivate them if needed.
> ** WARNING **
> You should do the following steps after you understand the security risk.

You can confirm the current SELinux status as follows:

    # getenforce
    Enforcing

If 'Enforcing' is replied, it is enabled. And you can disable it as follows:

    # setenforce 0
    # getenforce
    Permissive

> ** Tips **
> By editing /etc/selinux/config, you can disable it permanently.

As for firewall, an allowed port can be added by executing the following command.
The following example commands allows port 8000.

    # firewall-cmd --zone=public --add-port=80/tcp --permanent
    # firewall-cmd --zone=public --add-port=80/tcp

### View of Hatohol information
For example, if the Hatohol Web Frontend runs on computer: 192.168.1.1,
open the following URL from your Browser.

- http://192.168.1.1/hatohol

> ** Note **
> Currently the above pages have been checked with Google Chrome and Firefox.
> When using Internet Explorer, display layouts may collapse depending on the version of use.
