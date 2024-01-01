# How to setup Nagios Server in Ubuntu22 to monitoring infrastructure and how to add host to Nagios Server

## To Launch Instance

To launch EC2 ubuntu22 instance and SSH to machine.

![image](https://github.com/kohlidevops/nagios-setup/assets/100069489/0841fd3d-492d-47bd-a8fd-c8952ad60ed7)

### To update the System

```
sudo apt update -y
sudo apt upgrade -y
```

### Install Packages Required by Nagios Core

```
sudo apt install -y autoconf bc gawk dc build-essential gcc libc6 make wget unzip apache2 php libapache2-mod-php libgd-dev libmcrypt-dev make libssl-dev snmp libnet-snmp-perl gettext
```

### Download and Extract Nagios Core

```
mkdir nagioscore
cd nagioscore
sudo wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.4.13.tar.gz
sudo tar -xvf nagios-4.4.13.tar.gz
cd nagios-4.4.13
```

### Compile and Install Nagios

```
sudo ./configure --with-httpd-conf=/etc/apache2/sites-enabled
sudo make all
```

#### create the Nagios user and group, and add the www-data Apache user to the nagios group

```
sudo make install-groups-users
sudo usermod -a -G nagios www-data
```

#### To install Nagios binaries, service daemon script, and the command mode

```
sudo make install
sudo make install-daemoninit
sudo make install-init
sudo make install-commandmode
sudo make install-config
```

#### To install the Apache configuration for Nagios and activate the mod_rewrite and mode_cgi modules

```
sudo make install-webconf
sudo a2enmod rewrite cgi
sudo systemctl restart apache2.service
```

#### Create a new Nagios Admin user named nagiosadmin

```
sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
password : *****
password : *****
```

### Install and Configure Nagios Plugins and NRPE Plugins

```
sudo apt install monitoring-plugins nagios-nrpe-plugin -y
cd /usr/local/nagios/etc/
sudo vim nagios.cfg
#uncomment the below line
cfg_dir=/usr/local/nagios/etc/servers
save and exit
sudo vim resource.cfg
#add below line
$USER1$=/usr/lib/nagios/plugins
```
![image](https://github.com/kohlidevops/nagios-setup/assets/100069489/41386315-8981-4d14-ab89-8c1ae0f32a55)

```
sudo vim objects/contacts.cfg
```
![image](https://github.com/kohlidevops/nagios-setup/assets/100069489/0a3bb114-6cc9-435e-aeb0-e27d7beed885)

```
sudo vim objects/commands.cfg
add below contents at the end of file

define command{
        command_name check_nrpe
        command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
}
```

### Access Nagios Core Server via the web interface

```
sudo systemctl restart apache2
sudo systemctl restart nagios
sudo systemctl enable apache2
sudo systemctl enable nagios
sudo systemctl status apache2
sudo systemctl status nagios
```

![image](https://github.com/kohlidevops/nagios-setup/assets/100069489/adedb38b-7b4f-41b0-813b-22011399c276)

![image](https://github.com/kohlidevops/nagios-setup/assets/100069489/7a779764-36c0-42e7-ad60-8958577c4875)

# How to Add Linux Host to Nagios Server to Monitoring Linux Host

## To launch EC2 Instance for host

![image](https://github.com/kohlidevops/nagios-setup/assets/100069489/0137d781-1385-4d78-ac62-2f45070e3116)

To launch EC2 ubuntu22 instance and SSH to machine

```
sudo apt-get update -y
sudo apt-get upgrade -y
```

### To install NRPE and Nagios Plugin on your Linux host

```
sudo apt install nagios-nrpe-server nagios-plugins -y
```

### Configure Nagios NRPE Agent

```
sudo vim /etc/nagios/nrpe.cfg
edit and update
server_address = 43.204.142.198 #Public IP of your Linux host
allowed_hosts = 127.0.0.1,65.0.71.195 #Public IP of Nagios Server
save and exit
```

#### To restart NRPE service

```
sudo systemctl restart nagios-nrpe-server
sudo systemctl enable nagios-nrpe-server
```

## Install NRPE on the Nagios Server

```
sudo apt install nagios-nrpe-server -y
```

### Add Linux Host on Nagios Server

```
cd /usr/local/nagios/etc/
sudo chmod 775 servers
sudo chown nagios:nagios servers
cd servers
```

### Create a new configuration file named linuxhost-1.cfg for the Linux host that will be added to Nagios

```
sudo vim linuxhost-1.cfg
#add below content

define host{
use                     linux-server
host_name               linuxhost-1
alias                   LinuxHost-1
address                 43.204.142.198  #Public IP of host
}

define service{
use                             local-service
host_name                       linuxhost-1
service_description             Root / Partition
check_command                   check_nrpe!check_disk
}

define service{
use                             local-service
host_name                       linuxhost-1
service_description             /mnt Partition
check_command                   check_nrpe!check_mnt_disk
}

define service{
use                             local-service
host_name                       linuxhost-1
service_description             Current Users
check_command                   check_nrpe!check_users
}

define service{
use                             local-service
host_name                       linuxhost-1
service_description             Total Processes
check_command                   check_nrpe!check_total_procs
}

define service{
use                             local-service
host_name                       linuxhost-1
service_description             Current Load
check_command                   check_nrpe!check_load
}

save and exit

sudo chmod 664 linuxhost-1.cfg
sudo chown nagios:nagios linuxhost-1.cfg
```

#### Make ensure the configuration is correct

```
sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```
![image](https://github.com/kohlidevops/nagios-setup/assets/100069489/221a078b-0887-4599-832d-ccdcb41d9609)

### Restart Nagios server

```
sudo systemctl restart nagios
```

### Access the Nagios portal to check

![image](https://github.com/kohlidevops/nagios-setup/assets/100069489/ffe83d0e-f920-4e20-b504-7bfd1a6bd35a)
