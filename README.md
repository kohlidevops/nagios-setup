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
