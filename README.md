# Server Monitoring.
Server Monitoring is a process to monitor server's system resources like CPU Usage, Memory Consumption, I/O, Network, URL Services etc.Server Monitoring helps understanding server's system resource usage which can help you better your capacity planning and provide a better end-user experience. It helps determine exactly when a program is on track and when changes may be needed.

> In Server monitoring we monitor servers by diffrent commands and tools.
> We use diffrent commands on terminal to monitor system resources for example.
```sh
ps ,top , netstat , iostat and many more.
```
<img src="https://github.com/adsingh007/nagios-overview.md/blob/main/top.png" > <br/>


## Tech
We use diffrent DevOps tools for monitoring the servers and services in IT infrastructure. It provides high productivity , Higher-performing system and help us to reduce downtime of any services. 
Here we are using tools which are mentioned below.
- Nagios 
- Grafana


# Nagios
## Overview

Nagios is an open source monitoring system for computer systems. It was designed to run on the Linux operating system and can monitor devices running Linux, Windows and Unix operating systems. Nagios monitors  entire IT infrastructure to ensure systems, applications, services, and business processes are functioning properly. In the event of a failure, Nagios can alert technical staff of the problem, allowing them to begin remediation processes before outages affect business processes, end-users, or customers.
## Features :-  <img src="https://github.com/adsingh007/nagios-overview.md/blob/main/nagios.png" align="right" width="60%">
- Powerful script APIs allow easy monitoring of applications, services, and systems
- Multi-user access to web interface allows to view infrastructure status
- Alerts can be delivered to technical staff via email or SMS
- User-specific views ensures clients see only their infrastructure components.
- Open Source Software
## Installation process
### Process to be followed for Installing Nagios 4.4.5
1.	Nagios and its plugins will be installed under /usr/local/nagios directory.
2.	Nagios will be configured to monitor few services of your local machine (Disk Usage, CPU Load, Current Users, Total Processes, etc.)
3.	Nagios web interface will be available at http://localhost/nagios

<details>
    
### Step 1: Install Required Dependencies
We need to install Apache, PHP and some libraries like gcc, glibc, glibc-common and GD libraries and its development libraries before installing Nagios 4.4.5 with the source. And to do so, we can use yum default package installer.
- yum install -y httpd httpd-tools php gcc glibc glibc-common gd gd-devel make net-snmp
### Step 2: Create Nagios User and Group
- useradd nagios
- groupadd nagcmd
Next, add both the nagios user and the apache user to the nagcmd group using usermod command.
- usermod -G nagcmd nagios
- usermod -G nagcmd apache
### Step 3: Download Nagios Core 4.4.5 and Nagios Plugin 2.2.1
- mkdir /root/nagios
- cd /root/nagios
Now download latest Nagios Core 4.4.5 and Nagios plugins 2.2.1 packages with wget command.
- wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.4.5.tar.gz
- wget https://nagios-plugins.org/download/nagios-plugins-2.2.1.tar.gz
### Extract Nagios Core and its Plugins
- tar -xvf nagios-4.4.5.tar.gz
- tar -xvf nagios-plugins-2.2.1.tar.gz
#### Configure Nagios Core.
Now, first we will configure Nagios Core and to do so we need to go to Nagios directory and run configure file and if everything goes fine, it will show the output in the end as sample output. Please see below.
- cd nagios-4.4.5/
- ./configure --with-command-group=nagcmd
- make all
- make install
- make install-init
- make install-commandmode
- make install-config
### Step 5: Customizing Nagios Configuration
Open the "contacts.cfg" file with your choice of editor and set the email address associated with the nagiosadmin contact definition to receiving email alerts.
-  vi /usr/local/nagios/etc/objects/contacts.cfg
### Step 6: Install and Configure Web Interface for Nagios
- make install-webconf
In this step, we will be creating a password for “nagiosadmin”. After executing this command, please provide a password twice and keep it remember because this password will be used when you login in the Nagios Web interface.
- htpasswd -s -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
- systemctl start httpd.service
### Step 7: Compile and Install Nagios Plugin
- cd /root/nagios
- cd nagios-plugins-2.2.1/
- ./configure --with-nagios-user=nagios --with-nagios-group=nagios
- make
- make install
### Step 8: Verify Nagios Configuration Files
we are all done with Nagios configuration and its time to verify it
- /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
### Step 9: Add Nagios Services to System Startup
- systemctl enable nagios
- systemctl enable httpd
### Step 10: Login to the Nagios Web Interface
Your Nagios is ready to work, please open it in your browser with “http://Your-server-IP-address/nagios” and
Provide the username “nagiosadmin” and password.
## Now Configure NRPE on client side.
Note : Before doing any changes in the Nagios Server please take the backup of all configuration files .
### Step1: Copy nrpe.tgz on the remote Server where nrpe agent to be installed 
### Step 2: Extract tar file 
- tar xvfz nrpe.tgz
- /home/user/nrpe
Now go to the extracted nrpe location
- ls 
You will find two files in that location.
- mv nagios /usr/local
Move nagios file to the /usr/local location.
- cd /usr /local 
Now go the the /usr/location
- chown -R nagios:nagios nagios
Now change the ownership of nagios folder.
### Step 3: For nrpe service 
- cd /home /user/ nrpe
- mv nrpe.service /usr/lib/systemd/system
Moved nrpe.service to the system service installed packages 
- cd /usr / lib /systemd /system
- chmod 600 nrpe.service
Changed the owner-ship of nrpe.service for read and execution
- systemctl start nrpe.service
Now start the nrpe service on client side.
#### Config file to add host & services in nagios server.

```sh
define host {

    use                     linux-server            ; Name of host template to use
                                                    ; This host definition will inherit all variables that are defined
                                                    ; in (or inherited by) the linux-server host template definition.
    host_name               redhat_clint.server (client host name)
    alias                   redhat_clint.server 
    address                 192.168.0.123 (client ip)
    notifications_enabled   1
    check_interval          2
    retry_interval          2
    max_check_attempts      5
    notification_period     24x7
    notification_interval   120
    notification_options    d,u,r
    contact_groups          admins,critical
}
define service {

    use                     local-service           ; Name of service template to use
    host_name               redhat_clint.server
    hostgroup_name
    service_description     Root Partition
    check_command           check_disk
    check_interval          1
    check_period            24x7
    retry_interval          2
    max_check_attempts      3
    contact_groups          admins,critical
}

define service {

    use                     local-service           ; Name of service template to use
    host_name               redhat_clint.server
    hostgroup_name
    service_description     Average load
    check_command           check_load
    check_interval          1
    check_period            24x7
    retry_interval          2
    max_check_attempts      3
    contact_groups          admins,critical
}

```

</details>
