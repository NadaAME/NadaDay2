# installing nagios

```bash
LOGFILE=/tmp/nagios-installation.log
echo "DATE: `date`" > $LOGFILE
echo "Nagios Core 4.x Installation Start" >> $LOGFILE
echo "Stopping firewalld" >> $LOGFILE
systemctl stop firewalld >> $LOGFILE
echo "Disable firewalld service" >> $LOGFILE
systemctl disable firewalld >> $LOGFILE

echo "Disable SELinux" >> $LOGFILE
cat /etc/selinux/config |grep SELINUX=e >> $LOGFILE
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
cat /etc/selinux/config |grep SELINUX=d >> $LOGFILE

echo "Installing prerequisites" >> $LOGFILE
yum install -y wget httpd php gcc glibc glibc-common gd gd-devel make net-snmp perl perl-devel openssl >> $LOGFILE

echo "Downloading Nagios Core Package..." >> $LOGFILE
cd
wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.4.3.tar.gz

echo "Creating Nagios User" >> $LOGFILE
useradd nagios
echo "Creating Group" >> $LOGFILE
groupadd nagcmd
echo "Adding nagios to group" >> $LOGFILE
usermod -a -G nagcmd nagios

echo "Nagios package compeling" >> $LOGFILE
tar -xvf nagios-4.4.3.tar.gz
cd nagios-4.4.3 >> $LOGFILE
./configure --with-command-group=nagcmd >> $LOGFILE
sleep 30
make all >> $LOGFILE
make install >> $LOGFILE
make install-init >> $LOGFILE
make install-config >> $LOGFILE
make install-commandmode >> $LOGFILE 
make install-webconf >> $LOGFILE

echo "Copying Evenhandler Files" >> $LOGFILE
cp -rvf contrib/eventhandlers/ /usr/local/nagios/libexec/
echo "chaning permissions of eventhandlers" >> $LOGFILE
chown -R nagios:nagcmd /usr/local/nagios/libexec/eventhandlers

echo "Generating Nagiosadmin password" >> $LOGFILE
htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin

echo "Starting Web Service" >> $LOGFILE
systemctl start httpd.service  >> $LOGFILE
systemctl enable httpd.service  >> $LOGFILE
systemctl status httpd.service >> $LOGFILE
echo "Starting Nagios Service" >> $LOGFILE
systemctl start nagios.service >> $LOGFILE
systemctl enable nagios.service >> $LOGFILE
systemctl status nagios.service >> $LOGFILE

echo "Downloading Nagios Plugins" >> $LOGFILE
cd
wget https://nagios-plugins.org/download/nagios-plugins-2.2.1.tar.gz >> $LOGFILE

echo "Extracting Nagios Plugins" >> $LOGFILE
tar -xvf nagios-plugins-2.2.1.tar.gz >> $LOGFILE

echo "Installing Nagios Plugins" >> $LOGFILE
cd nagios-plugins-2.2.1 >> $LOGFILE 
./configure --with-nagios-user=nagios --with-nagios-group=nagcmd  >> $LOGFILE
make  >> $LOGFILE
make install >> $LOGFILE

echo "Nagios core and Nagios Plugins installed successfully" >> $LOGFILE
echo "DATE: `date`" >> $LOGFILE
```

# **Installing the Nagios Plugins**

Nagios needs plugins to operate properly. The official Nagios Plugins package contains over 50 plugins that allow you to monitor basic services such as uptime, disk usage, swap usage, NTP, and others.

```bash
sudo yum install openssl-devel -y
```

```bash
cd ~
curl -L -O https://nagios-plugins.org/download/nagios-plugins-2.2.1.tar.gz

#Extract the NRPE archive and navigate into the extracted directory:
tar zxf nagios-plugins-2.2.1.tar.gz
cd nagios-plugins-2.2.1

#Next configure their installation:
./configure

#Now build and install the plugins:
make
sudo make install
```

# Installing the check_nrpe Plugin

```bash
#Download it to your home directory with curl:
cd ~
curl -L -O https://github.com/NagiosEnterprises/nrpe/releases/download/nrpe-4.1.0/nrpe-4.1.0.tar.gz

#Extract the NRPE archive:
tar zxf nrpe-4.1.0.tar.gz
cd nrpe-4.1.0/

#Configure
./configure

#Now build and install check_nrpe plugin:
make check_nrpe
sudo make install-plugin
```

# **Configuring Nagios**

```bash
sudo vi /usr/local/nagios/etc/nagios.cfg

#Uncomment line
#cfg_dir=/usr/local/nagios/etc/servers

```

Now create the directory that will store the configuration file for each server that you will monitor:

```bash
sudo mkdir /usr/local/nagios/etc/servers
```

Next, add a new command to your Nagios configuration that lets you use the `check_nrpe` command in Nagios service definitions. Open the file `/usr/local/nagios/etc/objects/commands.cfg` in your editor.

Add the following to the end of the file to define a new command called `check_nrpe`:

```bash
sudo vi /usr/local/nagios/etc/objects/commands.cfg
...
define command{
        command_name check_nrpe
        command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
}
```

Start Nagios and enable it to start when the server boots:

```bash
sudo systemctl restart nagios
```

Now open nagios on `http://nagios_server_public_ip/nagios`
Username - **nagiosadmin**

Password - which you gave

## Installing Nagios Plugins and NRPE Daemon on a [Host]

```bash
#First create a nagios user which will run the NRPE agent:
sudo useradd nagios

#Install libraries
sudo apt update
sudo apt install autoconf gcc libmcrypt-dev make libssl-dev wget dc build-essential gettext -y

#Download Nagios Plugins to your home directory with curl:
cd ~
curl -L -O https://nagios-plugins.org/download/nagios-plugins-2.2.1.tar.gz

#Extract the Nagios Plugins archive and change to the extracted directory:
tar zxf nagios-plugins-2.2.1.tar.gz
cd nagios-plugins-2.2.1
./configure

#compile the plugins:
make
sudo make install

#install NRPE daemon
cd ~
curl -L -O https://github.com/NagiosEnterprises/nrpe/releases/download/nrpe-4.1.0/nrpe-4.1.0.tar.gz
tar zxf nrpe-4.1.0.tar.gz
cd nrpe-4.1.0/

#Configure NRPE:
./configure

#Now build and install NRPE and its startup script with these commands:
make nrpe
sudo make install-daemon
sudo make install-config
sudo make install-init
```

Update the NRPE configuration file and add some basic checks that Nagios can monitor.

monitor the disk usage of this server

```bash
df -h
```

**Updating nrpe config**

```bash
sudo vi /usr/local/nagios/etc/nrpe.cfg

#Locate these settings and alter them appropriately:
...
server_address=second_ubuntu_server_private_ip
...
allowed_hosts=127.0.0.1,::1,your_nagios_server_public_ip
...
command[**/dev/sda1**]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p **/dev/sda1**
...
```

Start NRPE

```bash
sudo systemctl start nrpe.service
sudo systemctl status nrpe.service

```

check the communication with the remote NRPE server. Run the following command on the Nagios server

```bash
/usr/local/nagios/libexec/check_nrpe -H <SLAVEIP>
```

## **Monitoring Hosts with Nagios**

Slave is on ubuntu 22

```bash
#Replace the highlighted word, monitored_server_host_name with the name of your host:
sudo vi /usr/local/nagios/etc/servers/dojotest-nagios-host1.cfg
```

```bash
#Monitor service up/down
define host {
        use                             <linux-server>
        host_name                       <your_monitored_server_host_name>
        alias                           <your_monitored_server_host_alias>
        address                         <your_monitored_server_private_ip>
        max_check_attempts              5
        check_period                    24x7
        notification_interval           30
        notification_period             24x7
}

#Monitor load average
define service {
        use                             generic-service
        host_name                       <your_monitored_server_host_name>
        service_description             Load average
        check_command                   check_nrpe!check_load
}

#Monitor disk usage
define service {
        use                             generic-service
        host_name                       <your_monitored_server_host_name>
        service_description             /dev/vda1 free space
        check_command                   check_nrpe!check_vda1
}
```

```bash
# HOST defination for monitoring

define host {
    use                     linux-server
    host_name               chandra.devops.learn
    alias                   Linux Server centos 7
    address                 172.31.17.193
}

# Service definations For Linux Host Using check_nrpe

define service {

    use                     local-service
    host_name               chandra.devops.learn
    service_description     PING
    check_command           check_ping!100.0,20%!500.0,60%
}

define service {

    use                     local-service
    host_name               chandra.devops.learn
    service_description     Root Partition
    check_command           check_nrpe!check_sda
}

define service {
    use                     local-service
    host_name               chandra.devops.learn
    service_description     Current Users
    check_command           check_nrpe!check_users
}

define service {
    use                     local-service
    host_name               chandra.devops.learn
    service_description     Total Processes
    check_command           check_nrpe!check_total_procs
}

define service {
    use                     local-service
    host_name               chandra.devops.learn
    service_description     Current Load
    check_command           check_nrpe!check_load
}

define service {
    use                     local-service
    host_name               chandra.devops.learn
    service_description     Zombie Process count
    check_command           check_nrpe!check_zombie_procs
}

define service {
    use                     local-service
    host_name               chandra.devops.learn
    service_description     SSH
    check_command           check_ssh
    notifications_enabled   0
}
```

```bash
sudo systemctl restart nagios
```