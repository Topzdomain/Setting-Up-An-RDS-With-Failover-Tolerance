<h2 align="center"> SETTING-UP AN RDS INSTANCE WITH FAILOVER TOLENCE</h2>

The purpose of this project is to demonstrate the creation of a Relational Database Service (RDS) Instance running MySQL and setting up a standby of the database in a different Availability Zone (AZ). 

Failover tolerance, often simply referred to as failover, is a critical concept in system design that ensures high availability and reliability by automatically transferring the workload to a standby system in the event of a failure. This process helps maintain uninterrupted service and minimizes downtime.

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/network-architecture.png" height="50%" width="70%"/>
</p>
<h5 align="center"> Network Architecture</h5>

<h2 align="left"> Creating the VPC</h2>

Based on the network architecture, I created a VPC with a CIDR of 192.168.0.0/16.

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/vpc.png" height="40%" width="60%"/>
</p>

<h2 align="left"> Creating Internet Gateway</h2>

Next, I created the internet gateway which I attached to the vpc.

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/igw.png" height="40%" width="60%"/>
</p>


<h2 align="left"> Creating Subnets</h2>

Next, I created the two public subnets in different AZs, Subnet A in us-east-1a and Subnet B in us-east-1c. After creation, I edited the subnet settings to auto-assign IPv4 address to both public subnets.

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/public-subnet-A.png" height="40%" width="60%"/>
</p>

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/public-subnet-B.png" height="40%" width="60%"/>
</p>

Configuring Subnet Settings to Auto-Assign IPv4 Address to Subnets

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/editing-subnet-settings.png" height="60%" width="80%"/>
</p>

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/enable-auto-assign-ip.png" height="40%" width="60%"/>
</p>

Click the save button after ticking the box to enable the auto-assign public IPv4 address

<h2 align="left"> Creating and Configuring Route Table</h2>

A default route is usually created after a vpc is created. I edited the name to "Main-Route" and edited the routing table to send all internet-bound traffic through the IGW. I also associated both subnets with the main route.

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/edit-route-table.png" height="60%" width="80%"/>
</p>

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/associating-subnets.png" height="60%" width="80%"/>
</p>

<h2 align="left"> Configuring Security Group</h2>

I configured the default security group created by the VPC to allow SSH and HTTP traffic from IP address. Also, after I create the EC2 instance, I'll need to allow MySQL/Aurora traffic from the EC2 private IP address. The reason why I allowed all SSH and HTTP traffic from the internet is because I'll terminate all instances and set-up immediately after the lab.

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/sg-configuration.png" height="60%" width="80%"/>
</p>


<h2 align="left"> Launching An RDS Instance</h2>

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/db-creation-method.png" height="40%" width="60%"/>
</p>

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/engine-option.png" height="70%" width="50%"/>
</p>

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/template-n-availability.png" height="45%" width="60%"/>
</p>

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/setting.png" height="60%" width="45%"/>
</p>

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/instance-configuration.png" height="45%" width="60%"/>
</p>

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/connectivity-1.png" height="45%" width="60%"/>
</p>

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/connectivity-2.png" height="45%" width="60%"/>
</p>

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/connectivity-3.png" height="45%" width="60%"/>
</p>


<h2 align="left"> Launching The EC2 Instance</h2>

The next step is to configure and launch an EC2 Instance in the same subnet and availability zone as the RDS. During the configuration, I installed Apache2 (HTTPd) and downloaded the phpMyAdmin tar file into the /var/www/html directory. After launching the EC2 Instance, I re-configured the security group to allow MySQL traffic from the EC2 Instance's private IP.

As part of configuring the EC2 Instance to be able to communicate with the database, I logged into the EC2 Instance using SSH, extracted the phpMyAdmin package, renamed it (phpMyAdmin-latest-all-languages) as phpMyAdmin (as a directory) after deleting the tar file, navigated into the phpMyAdmin directory, copied the sample configuration file (config.sample.inc.php) into an actual configuration file (config.inc.php) and edited it to use the endpoint of the database as localhost.

user-data

```commandline
#!/bin/bash
yum update -y
amazon-linux-extras install php7.2 -y
yum install httpd -y
cd /var/www/html
wget https://www.phpmyadmin.net/downloads/phpMyAdmin-latest-all-languages.tar.gz
systemctl enable httpd
systemctl start httpd
```

phpMyAdmin configuration

```commandline
sudo -i
cd /var/www/html
tar xvf phpMyAdmin-latest-all-languages.tar.gz
rm -rf phpMyAdmin-latest-all-languages.tar.gz
mv phpMyAdmin-latest-all-languages phpMyAdmin
cd phpMyAdmin
cp config.sample.inc.php config.inc.php
```

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/ec2-ami.png" height="60%" width="45%"/>
</p>

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/instance-type.png" height="40%" width="55%"/>
</p>

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/network-settings.png" height="40%" width="55%"/>
</p>

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/user-data.png" height="40%" width="40%"/>
</p>

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/sg-re-configuration.png" height="60%" width="80%"/>
</p>
<h5 align="center"> SG Re-Configuration</h5>

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/logging-into-ec2-instance.png" height="45%" width="65%"/>
</p>
<h5 align="center"> Logging into EC2 Instance for phpMyAdmin Configuration</h5>

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/editing-localhost-phpmyadmin.png" height="60%" width="80%"/>
</p>
<h5 align="center"> Editing Localhost for phpMyAdmin Configuration File</h5>


<h2 align="left"> Logging into the Database Using phpMyAdmin</h2>

Now that phpMyAdmin which is linked to the primary database has been completely configured, using the EC2 Instance's public IP (http://44.222.200.255/phpMyAdmin) and the login credentials that I set-up during the database creation, I logged into the phpMyAdmin dashboard.

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/phpmyadmin-login-page.png" height="60%" width="40%"/>
</p>
<h5 align="center"> phpMyAdmin Login Page</h5>

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/phpmyadmin-dashboard.png" height="80%" width="100%"/>
</p>
<h5 align="center"> phpMyAdmin Dashboard</h5>


<h2 align="left"> Test and Validation</h2>

To test and validate the set-up, I'll reboot the RDS with failover, simulating failure of the database. The stand-by RDS on the other availability zone should take over as the primary database without any downtime on the phpMyAdmin as the failover takes seconds. Sometimes, reloading the phpMyAdmin dashboard is necessary.

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/rds-us-east-1a.png" height="70%" width="95%"/>
</p>
<h5 align="center"> Primary RDS in US-East-1a</h5>

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/rds-reboot-dialogue-box.png" height="70%" width="95%"/>
</p>

<p align="center">
<img src="https://github.com/Topzdomain/Setting-Up-An-RDS-With-Failover-Tolerance/blob/main/screenshots/stand-by-rds.png" height="70%" width="95%"/>
</p>
<h5 align="center"> StandBy RDS in US-East-1c Taking Over as the Primary RDS</h5>





