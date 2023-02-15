# PROJECT-6
WEB SOLUTION WITH WORDPRESS


## WEB SOLUTION WITH WORDPRESS

**Step 1 — Prepare a Web Server**

- Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.

- Learn How to Add EBS Volume to an EC2 instance [here](https://www.youtube.com/watch?v=HPXnXkBzIHw)

<img width="519" alt="Creating volume" src="https://user-images.githubusercontent.com/115954100/218573209-8ddda957-acfd-4af0-8012-cf92a3b5fe49.png">

- Attach all three volumes one by one to your Web Server EC2 instance

<img width="505" alt="Creating volume 2" src="https://user-images.githubusercontent.com/115954100/218573744-981dbefa-dc32-408e-855b-0131fc32e756.png">

- Open up the Linux terminal to begin configuration

- Use lsblk [lsblk](https://man7.org/linux/man-pages/man8/lsblk.8.html) command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in **/dev/ directory**. Inspect it with ***ls /dev/*** and make sure you see all 3 newly created block devices there – their names will likely be ***xvdf, xvdh, xvdg***.

<img width="506" alt="Logical volumes created and verified" src="https://user-images.githubusercontent.com/115954100/218575935-d3a8bb4c-3cf7-4b3b-bca4-99a1b08ff742.png">

- Use `df -h` command to see all mounts and free space on your server

- Use `gdisk` utility to create a single partition on each of the 3 disks

- `sudo gdisk /dev/xvdf`

<img width="501" alt="Creating disk partition" src="https://user-images.githubusercontent.com/115954100/218576667-44861d66-a688-43b9-b339-9520c39eb25a.png">

- Use `lsblk` utility to view the newly configured partition on each of the 3 disks.

- <img width="336" alt="Newly Configured Partitions" src="https://user-images.githubusercontent.com/115954100/218577029-9eb3cecd-3dee-4034-9ef1-a5f60e8e46a0.png">

- Install lvm2 [lvm2](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)) package using `sudo yum install lvm2`. Run `sudo lvmdiskscan` command to check for available partitions.

- Note: Previously, in Ubuntu we used **apt** command to install packages, in RedHat/CentOS a different package manager is used, so we shall use **yum** command instead.

- Use [pvcreate](https://linux.die.net/man/8/pvcreate) utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

- `sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

- Verify that your Physical volume has been created successfully by running `sudo pvs`

<img width="469" alt="Physical volumes created and verified" src="https://user-images.githubusercontent.com/115954100/218651983-2ddcb7da-515e-430a-a755-342e40d80234.png">

- Use [vgcreate](https://linux.die.net/man/8/vgcreate) utility to add all 3 PVs to a volume group (VG). Name the VG **webdata-vg**

- `sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

- Verify that your VG has been created successfully by running `sudo vgs`

<img width="530" alt="Volume group created and verified" src="https://user-images.githubusercontent.com/115954100/218653319-9b99c8e5-cc3a-44c9-b98f-ffe9cf31ea76.png">

- Use [lvcreate](https://www.example.com) utility to create 2 logical volumes. **apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE:** apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

- `sudo lvcreate -n apps-lv -L 14G webdata-vg` and `sudo lvcreate -n logs-lv -L 14G webdata-vg`

- Verify that your Logical Volume has been created successfully by running `sudo lvs`

<img width="506" alt="Logical volumes created and verified" src="https://user-images.githubusercontent.com/115954100/218654733-a2af8e65-593c-4bca-85e8-546ba331d156.png">

- Verify the entire setup

- `sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk 
`

![lsblk3](https://user-images.githubusercontent.com/115954100/218656019-dffaca38-8aad-475d-af3a-c155073db4bd.png)

- Use **mkfs.ext4** to format the logical volumes with [ext4](https://en.wikipedia.org/wiki/Ext4) filesystem

- `sudo mkfs -t ext4 /dev/webdata-vg/apps-lv` and `sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`

- Create **/var/www/html** directory to store website files

- `sudo mkdir -p /var/www/html`

- Create **/home/recovery/logs** to store backup of log data

- `sudo mkdir -p /home/recovery/logs`

- Mount **/var/www/html** on **apps-lv** logical volume

- `sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

- Use [rsync](https://linux.die.net/man/1/rsync) utility to backup all the files in the log directory **/var/log** into **/home/recovery/logs** (This is required before mounting the file system)

- `sudo rsync -av /var/log/. /home/recovery/logs/`

- Mount **/var/log** on **logs-lv** logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very
important)

- `sudo mount /dev/webdata-vg/logs-lv /var/log`

- Restore log files back into **/var/log** directory

- `sudo rsync -av /home/recovery/logs/. /var/log`

- Update **/etc/fstab** file so that the mount configuration will persist after restart of the server.

- The UUID of the device will be used to update the **/etc/fstab** file;

- `sudo blkid`

<img width="507" alt="sudo blkid" src="https://user-images.githubusercontent.com/115954100/218827462-f7ddfcdf-9d1a-451c-9b71-279af208ab94.png">

- `sudo vi /etc/fstab`

- Update **/etc/fstab** in this format using your own UUID and rememeber to remove the leading and ending quotes.

<img width="504" alt="UUID" src="https://user-images.githubusercontent.com/115954100/218828944-a648eaf5-0be7-445a-9ff7-da28b79f3927.png">

- Test the configuration and reload the daemon

- `sudo mount -a`
- `sudo systemctl daemon-reload`

- Verify your setup by running **df -h**, output must look like this:

<img width="422" alt="Updating fstab and reload the deamon" src="https://user-images.githubusercontent.com/115954100/218831209-440fa4d5-dd8f-4ea1-a289-1d7ea932a3a0.png">

**Step 2 — Prepare the Database Server**

- Launch a second RedHat EC2 instance that will have a role – ‘DB Server’
Repeat the same steps as for the Web Server, but instead of **apps-lv** create **db-lv** and mount it to **/db** directory instead of **/var/www/html/**.

**Step 3 — Install WordPress on your Web Server EC2**

- Update the repository

- `sudo yum -y update`

- Install wget, Apache and it’s dependencies

- `sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

- Start Apache

- `sudo systemctl enable httpd`
- `sudo systemctl start httpd`

- To install PHP and it’s depemdencies

- Understands the redhat server version you are working with cause this will determine the **RHEL version of EPEL and Remirepo of the PHP dependencies to install**

```
 sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
 sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
 sudo yum module list php
 sudo yum module reset php
 sudo yum module enable php:remi-8.1
 sudo yum install php php-opcache php-gd php-curl php-mysqlnd
 sudo systemctl start php-fpm
 sudo systemctl enable php-fpm
 setsebool -P httpd_execmem 1
```

- Restart Apache

- `sudo systemctl restart httpd`

<img width="944" alt="PHP and Apache is running" src="https://user-images.githubusercontent.com/115954100/218851211-85673b47-2987-43da-9d82-b5c5dc2aa1c1.png">

- Download wordpress and copy wordpress to **var/www/html**

```
 mkdir wordpress
 cd wordpress
 sudo wget http://wordpress.org/latest.tar.gz
 sudo tar xzvf latest.tar.gz
 sudo rm -rf latest.tar.gz
 cp wordpress/wp-config-sample.php wordpress/wp-config.php
 cp -R wordpress /var/www/html/
```

- Configure SELinux Policies

```
 sudo chown -R apache:apache /var/www/html/wordpress
 sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
 sudo setsebool -P httpd_can_network_connect=1
```

**Step 4 — Install MySQL on your DB Server EC2**

```
sudo yum update
sudo yum install mysql-server -y
```

- Verify that the service is up and running by using `sudo systemctl status mysqld`, if it is not running, restart the service and enable it so it will be running even after reboot:

```
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```

- A password was set for the user using `sudo mysql_secure_installation`. A simple password was set

**Step 5 — Configure DB to work with WordPress**

- Now with a password set for root user

```
 sudo mysql -u root -p
 CREATE DATABASE wordpress;
 CREATE USER 'Godwin'@'%' IDENTIFIED WITH mysql_native_password BY 'GodwynE';
 GRANT ALL ON wordpress.* TO 'Godwin'@'%' WITH GRANT OPTION;
 FLUSH PRIVILEGES;
 SHOW DATABASES;
``` 

<img width="612" alt="Wordpress database created for DB server" src="https://user-images.githubusercontent.com/115954100/219131626-0722b275-fe00-42ed-8416-262d0cd85fe0.png">

- I exited using `exit`

- After this, I set up the bind address using `sudo vi /etc/my.cnf`

<img width="461" alt="Bind address" src="https://user-images.githubusercontent.com/115954100/218948504-2f5da033-af47-4e1f-8894-d21cec721531.png">

- I restarted the service using `sudo systemctl restart mysqld`

- Back to the **web server** wp-config.php was edited using `sudo vi wp-config.php`

<img width="560" alt="wp-config php" src="https://user-images.githubusercontent.com/115954100/218951780-5c91122a-828b-41bd-a886-9c925172489d.png">

- I restarted the service using `sudo systemctl restart httpd`

- In order to see the default page of wordpress, the default web page of Apache was disabled using

- `sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup`

**Step 6 — Configure WordPress to connect to remote database**

- **Hint**: Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server **ONLY** from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32

<img width="815" alt="Inbound rules" src="https://user-images.githubusercontent.com/115954100/218948642-94c5f851-e2bc-4514-814c-bba63157c9c2.png">

- I tested to see that I can connect from my Web Server to my DB server using

- `sudo mysql -h private IP of the database server -u username -p`

- I successfully execute `SHOW DATABASES`; command to see a list of existing databases.

<img width="952" alt="Using mysql server to connect with the database" src="https://user-images.githubusercontent.com/115954100/219132035-4446d1f0-939b-4ca2-ae96-44a3a22b2e91.png">

- I changed permissions and configuration so Apache could use WordPress by using the following commands:

```
 sudo chown -R apache:apache /var/www/html/
 sudo chcon -t httpd_sys_rw_content_t /var/www/html/ -R
 sudo setsebool -P httpd_can_network_connect=1
 sudo setsebool -P httpd_can_network_connect_db 1
```

- Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)

- Try to access from your browser the link to your WordPress

- `http://<Web-Server-Public-IP-Address>/wordpress/

![wordpress login](https://user-images.githubusercontent.com/115954100/219142252-1cd9ce73-0ded-46c3-90ee-195442bb646f.png)

<img width="923" alt="wordpress installed and logged in" src="https://user-images.githubusercontent.com/115954100/219142315-cbc8b4d4-4229-400c-bf07-291c04a3489c.png">

- **PROJECT COMPLETED**
