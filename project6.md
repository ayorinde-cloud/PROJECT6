# IMPLEMENTATION OF PROJECT6

## WEB SOLUTION WITH WORDPRESS

## Step 1 — Prepare a Web Server

### 1 -  Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.

![Web Server EC2](./images/ EC2_WEB_SERVER.MPG)

### 2 - Attach all three volumes one by one to your Web Server EC2 instance

![NEW DISK VOLUMNS](./images/VOLUMES_ADDED_EC2.MPG)

### 3 - Use lsblk command to inspect what block devices are attached to the server.

`sudo lsblk`

![lsblk](./images/lsblk.MPG)

`ls /dev/`

![ls dev](./images/ls_dev.MPG)

### 4 - Use df -h command to see all mounts and free space on your server

`sudo df –h`

![DF -H](./images/WEB_SERVER_FREE_SPACE.MPG)

### 5 - Use gdisk utility to create a single partition on each of the 3 disks

sudo gdisk /dev/nvme1n1

sudo gdisk /dev/nvme2n1

sudo gdisk /dev/nvme3n1

![PARTITION 1](./images/ DISK1_partition1.MPG)

![PARTITION 2](./images/ DISK2_partition1.MPG)

![PARTITION 3](./images/ DISK3_partition1.MPG)

### 6 - Use lsblk utility to view the newly configured partition on each of the 3 disks

`sudo lsblk`

![AVAILABLE PARTITION](./images/ web-server_partitions.MPG)

### 7 - Install lvm2 package using sudo yum install lvm2. Run sudo lvmdiskscan command to check for available partitions.

sudo yum install lvm2

sudo lvmdiskscan

![AVAILABLE PARTITION](./images/ LVMDISK_SCAN.MPG)

### 8 -  Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

sudo pvcreate /dev/nvme1n1p1

sudo pvcreate /dev/nvme2n1p1

sudo pvcreate /dev/nvme3n1p1

### 9 - Verify that your Physical volume has been created successfully by running 

`sudo pvs`

![PHYSICAL VOLUMN](./images/PVs.MPG)

### 10 - Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg

`sudo vgcreate webdata-vg /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1`

### 11 - Verify that your VG has been created successfully by running 

`sudo vgs`

![VGs CREATED](./images/WEB-SERVER_VGs.MPG)

### 12 - Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

`sudo lvcreate -n apps-lv -L 14G webdata-vg`

`sudo lvcreate -n logs-lv -L 14G webdata-vg`

### 13 - Verify that your Logical Volume has been created successfully by running

`sudo lvs`

![LVs CREATED](./images/WEB-SERVER_LVs.MPG)

### 14 - Verify the entire setup

`sudo vgdisplay -v #view complete setup - VG, PV, and LV`

![ENTIRE_DISK_SETUP](./images/VG_PV_LV.MPG)

### 15 - Use mkfs.ext4 to format the logical volumes with ext4 filesystem

`sudo mkfs -t ext4 /dev/webdata-vg/apps-lv`

`sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`

![LOGICAL VOLUMES FORMATED](./images/LVS_FORMATED.MPG)

### 16 Create /var/www/html directory to store website files

`sudo mkdir -p /var/www/html`

### 17 - Create /home/recovery/logs to store backup of log data

`sudo mkdir -p /home/recovery/logs`

### 18 - Mount /var/www/html on apps-lv logical volume

`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

### 19 - Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)

`sudo rsync -av /var/log/. /home/recovery/logs`

### 20 - Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted.) 


`sudo mount /dev/webdata-vg/logs-lv /var/log`

### 21 - Restore log files back into /var/log directory

`sudo rsync -av /home/recovery/logs/. /var/log`

`sudo lsblk`

![DISK SETUP](./images/ENTIRE_DISK_SETUP2.MPG)

### 22 - Update /etc/fstab file so that the mount configuration will persist after restart of the server

`sudo blkid`

![DISK SETUP](./images/BLKID.MPG)

### 23 - Update /etc/fstab in this format using your own UUID

`sudo vi /etc/fstab`

![fstab](./images/ADDING_UUID.MPG)

### 24 - Test the configuration and reload the daemon

`sudo mount –a`

`sudo systemctl daemon-reload`

### 25 - Verify your setup by running

`df –h`

![DISK STATUS](./images/df -h_output.MPG)

## Step 2 — Prepare the Database Server
### Launch a second RedHat EC2 instance that will have a role – ‘DB Server’
Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.

![DATABASE_EC2](./images/DATABASE_EC2_INSTANCE_RUNNING.MPG)

![Disk Volumn](./images/DISK_VOLUMNS_CREATED.MPG)

![Disk Volumn](./images/physical_volunms_created.MPG)

![Disk Volumn](./images/disk_volume_status.MPG)

![DATABASE_SERVER_UUID](./images/DATABASE_SERVER_UUID.MPG)

![DB_SERVER_STATUS](./images/DB_SERVER_STATUS.MPG)

## Step 3 — Install WordPress on your Web Server EC2

### 1 - Update the repository

`sudo yum -y update`

### 2 - Install wget, Apache and it’s dependencies

`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

### 3 - Start the apache service

`sudo systemctl enable httpd`

`sudo systemctl start httpd`

### 4 - Install PHP and it’s Depemdencies

`sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

`sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

`sudo yum module list php`

`sudo yum module reset php`

`sudo yum module enable php:remi-7.4`

`sudo yum install php php-opcache php-gd php-curl php-mysqlnd`

`sudo systemctl start php-fpm`

`sudo systemctl enable php-fpm`

`sudo setsebool -P httpd_execmem 1`

### 5 - Restart Apache

`sudo systemctl restart httpd`

### 6 - Download wordpress and copy wordpress to var/www/html

`mkdir wordpress`

`cd wordpress`

`sudo wget http://wordpress.org/latest.tar.gz`

`sudo tar xzvf latest.tar.gz`

`sudo rm -rf latest.tar.gz`

`sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php`

`sudo cp -R wordpress /var/www/html/`

### 7 - Configure SELinux Policies

`sudo chown -R apache:apache /var/www/html/wordpress`

`sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress –R`

`sudo setsebool -P httpd_can_network_connect=1`

![Wordpress](./images/SELinux_codes.MPG)

## Step 4 — Install MySQL on your DB Server EC2

### 1 - Install MySQL on your DB Server EC2

`sudo yum update`

`sudo yum install mysql-server`

`sudo systemctl restart mysqld`

`sudo systemctl status mysqld`

![MySQL Status](./images/Mysql_Status.MPG)

`sudo systemctl restart mysqld`

`sudo systemctl enable mysqld`

### 2 - ON DATABASE SERVER INSTANCE ADD A NEW INBOUND RURE TO USE PORT 3306.

![MySQL](./images/Mysql_port3306.MPG)

### 3 - Create a user for the wordpress server to connect to the database on Database-server

`sudo mysql`

`mysql  > CREATE DATABASE wordpress;`

`mysql > CREATE USER `myuser2`@`172.31.47.47` IDENTIFIED BY 'mypass2';`

`mysql > GRANT ALL ON wordpress.* TO 'myuser2'@'172.31.47.47';`

`mysql > FLUSH PRIVILEGES;`

`mysql > SHOW DATABASES;`

`mysql >exit`

![MySQL](./images/Mysql_database.MPG)

## 5 - Open MySQL Port 3306 on the DB-server EC2 to allow traffic between the App-server and DB-server. For extra security, we will only be allowing the IP address of the App-server.

![MySQL PORT 3306](./images/wordpress_server_IP_Address.MPG)

## 6 - Configure WordPress to connect to remote database.

### 1 - Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client

`sudo yum install mysql`

`sudo mysql -u myuser2 -p -h 172.31.43.179`

`mysql > SHOW DATABASES;`

`mysql >exit`

![MySQL](./images/Connecting_mysql_server_from_mysql_client_private_IP_Address.MPG)

### 2 - Change permissions and configuration so Apache could use WordPress: Create a configuration file for wordpress in order to point client requests to the wordpress directory.

`sudo vi /etc/httpd/conf.d/wordpress.conf`

![Wordpress](./images/wordpress.conf_file.MPG)

### 3 - Apply the changes, restart Apache

`sudo systemctl restart httpd`

### 4 - Go to https://api.wordpress.org/secret-key/1.1/salt/ to generate new Authentication unique keys and salts, and update the /wordpress/wp-config.php file with these keys and the username, database, and password Wordpress would use to access the database.

![WP Authentication Key](./images/wordpress_secret-key.MPG)

`sudo vi /var/www/html/wordpress/wp-config.php`

![Wordpress](./images/wp-config.php_file.MPG)

![Wordpress](./images/Edit_wordpress_secret-key.MPG)

### 5 - configure SELinux for wordpress

`sudo semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/html/wordpress/.*?"`

`sudo yum provides /usr/sbin/semanage`

`sudo yum install policycoreutils-python-utils`

### 6 - Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)

![Mysql](./images/Mysql_port3306 and port80.MPG)

### 7 Access from your browser the link to your WordPress : http://13.42.46.226/wordpress/ and install it.

[Wordpress](http://13.51.70.92/wordpress/wp-admin)

![Wordpress](./images/wordpress_wellcome_page.MPG)

![Wordpress](./images/wordpress_installed.MPG)
