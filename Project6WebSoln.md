## Documentation for Project 6: IMPLEMENT A CLIENT SERVER ARCHITECTURE USING MYSQL DATABASE MANAGEMENT SYSTEM (DBMS)

**Preparing prerequisites**

- Create a new EC2 Instance of t2.nano family with RedHat 20.04 LTS (HVM) image.

- Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.
Learn How to Add EBS Volume to an EC2 instance here.

- Attach all three volumes one by one to your Web Server EC2 instance

- Open up the Linux terminal to begin configuration.

- Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg:

`lsblk`

![lsblk Status](./image/lsblk-status.PNG)

- Use df -h command to see all mounts and free space on your server:

`df -h`

![df-h Status](./image/df-h-output.PNG)

- Use gdisk utility to create a single partition on each of the 3 disks:

`gdisk`

- Steps to partition:

- Run command in line 34 to display the disk

`lsblk`

- In the command line:

`sudo gdisk /dev/xvdf` repeat for xvdg and xvdh

- Command (? for help): type n after the colon to define as new

- Press enter until this displays "Hex code or GUID (L to show codes, Enter = 8300):"

- To change the default Linux file system to LVM, type 8e00 after the colon in line 42

The output should display as :

- Changed type of partition to 'Linux LVM'

- Command (? for help): p to display the action was done correctly.

- Command (? for help): w to write the action

- Do you want to proceed? (Y/N): Y to confirm

![Xvdf Partition Output](./image/xvdf-part1-output.PNG)

![Xvdg Partition Output](./image/xvdg-part1-outpu.PNG)

![Xvdh Partition Output](./image/xvdh-part1-outpu.PNG)

`sudo gdisk /dev/xvdf`

![Gdisk Status](./image/gdisk-output.PNG)

- Use lsblk utility to view the newly configured partition on each of the 3 disks.

`lsblk`

![lsblk New Status](./image/lsblk-new-output.PNG)

- Install lvm2 package using sudo yum install lvm2. Run sudo lvmdiskscan command to check for available partitions:

`sudo yum install lvm2 -y`

![Lvm2 Status](./image/lvm2-install-status.PNG)

- To confirm lvm is install:

`which lvm`

![Lvm Status](./image/lvm-success.PNG)

`sudo lvmdiskscan`

![Lvmdiskscan Status](./image/lvmdiskscan-status.PNG)

- Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM:

`sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

![Pvcreate Physical Volumes](./image/pvcreate-volume-output.PNG)

`sudo pvs`

![Sudo Pvs Check](./image/sudo-pvs-output.PNG)

- Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg:

`sudo vgcreate vg-webdata /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

![Vgcreate Output](./image/vgcreate-create-output.PNG)

- Verify that your VG has been created successfully by running:

`sudo vgs`

![Vgs](./image/vgs-output.PNG)

- Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs:

`sudo lvcreate -n apps-lv -L 14G vg-webdata`

![Vgs](./image/apps-lv-output.PNG)

`sudo lvcreate -n logs-lv -L 14G vg-webdata`

![Vgs](./image/logs-lv-output.PNG)

- Verify that your Logical Volume has been created successfully by running:

`sudo lvs`

![Sudo Lvs Output](./image/sudo-lvs-status.PNG)

- Verify the entire setup:

`sudo vgdisplay -v #view complete setup - VG, PV, and LV`

![View Complete Output](./image/entire-output.PNG)

`sudo lsblk`

![Sudo lsblk Output](./image/sudo-lsblk-output.PNG)

- Use mkfs.ext4 to format the logical volumes with ext4 filesystem:

`sudo mkfs -t ext4 /dev/vg-webdata/apps-lv`

![Mkfs Applv Output](./image/mkfs-apps-lv-output.PNG)

`sudo mkfs -t ext4 /dev/vg-webdata/logs-lv`

![Mkfs loglv Output](./image/mkfs-logs-lv-output.PNG)

- Create /var/www/html directory to store website files:

`sudo mkdir -p /var/www/html`

![Create Directory Website](./image/directory-website-files-output.PNG)

- Create /home/recovery/logs to store backup of log data:

`sudo mkdir -p /home/recovery/logs`

![Create Directory Home Recovery](./image/directory-home-recovery-output.PNG)

- Mount /var/www/html on apps-lv logical volume:

`sudo mount /dev/vg-webdata/apps-lv /var/www/html/`

![Mount Webdata Apps](./image/mount-webdata-apps-lv.PNG)

- To check mount status:

`df -h`

![Mount Success](./image/mount-success-output.PNG)

- Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system):

`sudo rsync -av /var/log/. /home/recovery/logs/`

![Mount Success](./image/rsync-home-recovery.PNG)

`sudo ls -l /home/recovery/logs`

![Check Success](./image/check-create-status.PNG)

- Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why steps 146 & 148 above is very important):

`sudo mount /dev/vg-webdata/logs-lv /var/log`

![Mount Webdata logs](./image/mount-webdata-logs-lv.PNG)

- Restore log files back into /var/log directory:

`sudo rsync -av /home/recovery/logs/. /var/log`

![Restore logs Directory](./image/restore-log-directory.PNG)

- To check the success status of the step 190:

`sudo ls -l /var/log`

![Restore logs Directory Success](./image/check-success-status-restore-log-directory.PNG)

- Update /etc/fstab file so that the mount configuration will persist after restart of the server:

**UPDATE THE /ETC/FSTAB FILE**

-The UUID of the device will be used to update the /etc/fstab file;

`sudo blkid`

![Sudo blkid](./image/sudo-blkid-output.PNG)

- Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.

`sudo vi /etc/fstab`

![Sudo Etc Fstab](./image/sudo-etc-fstab.PNG)

- Test the configuration and reload the daemon:

`sudo mount -a`

![Sudo Mount Success](./image/sudo-mount-a.PNG)

`sudo systemctl daemon-reload`

![System Reload Success](./image/system-reload-success.PNG)

- Verify your setup by running df -h, output must look like this:

`df -h`

![Verify Setup](./image/verify-setup-output.PNG)

**Prepare the Database Server**

- Launch an EC2 instance that will serve as "DB_Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.
Learn How to Add EBS Volume to an EC2 instance here.

- Attach all three volumes one by one to your Web Server EC2 instance

- Open up the Linux terminal to begin configuration.

- Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg:

`lsblk`

![lsblk Status DB Server](./image/lsblk-status-dbserver.PNG)

`sudo gdisk /dev/xvdf` repeat for xvdg and xvdh

- Command (? for help): type n after the colon to define as new

- Press enter until this displays "Hex code or GUID (L to show codes, Enter = 8300):"

- To change the default Linux file system to LVM, type 8e00 after the colon in line 42

The output should display as :

- Changed type of partition to 'Linux LVM'

- Command (? for help): p to display the action was done correctly.

- Command (? for help): w to write the action

- Do you want to proceed? (Y/N): Y to confirm

- Use lsblk utility to view the newly configured partition on each of the 3 disks.

`lsblk`

![lsblk New Status DBServer](./image/lsblk-new-dbserver-output.PNG)

- Install lvm2

`sudo yum install lvm2 -y`

![Lvm2 Dbserver Output](./image/lvm2-install-dbserver-status.PNG)

- Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM:

`sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

![Pvcreate Physical Volumes Dbserver](./image/pvcreate-volume-dbserver-output.PNG)

- Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg:

`sudo vgcreate vg-database /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

![Vgcreate Dbserver Output](./image/vgcreate-create-dbserver-output.PNG)

- Verify that your VG has been created successfully by running:

`sudo vgs`

![Vgs Dbserver Output](./image/vgs-dbserver-output.PNG)

- Create logical Volume:

`sudo lvcreate -n db-lv -L 20G vg-database`

![Lv Create Output](./image/db-lv-output.PNG)

- Verify that your Logical Volume has been created successfully by running:

`sudo lvs`

![Sudo Lvs Dbserver Output](./image/sudo-lvs-dbserver-status.PNG)

- Create a directory called db:

`sudo mkdir /db`

![Db Directory Create](./image/db-directory-create.PNG)

- Use mkfs.ext4 to format the logical volumes with ext4 filesystem:

`sudo mkfs -t ext4 /dev/vg-database/db-lv`

![Mkfs Dblv Output](./image/mkfs-db-lv-output.PNG)

- Mount /var/www/html on apps-lv logical volume:

`sudo mount /dev/vg-database/db-lv /db`

![Mount Webdata Db](./image/mount-webdata-db-lv.PNG)

- To check mount status:

`df -h`

![Mount Dbserver Success](./image/mount-dbserver-success-output.PNG)

-The UUID of the device will be used to update the /etc/fstab file;

`sudo blkid`

![Sudo blkid Dbserver](./image/sudo-dbserver-blkid-output.PNG)

- Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.

`sudo vi /etc/fstab`

![Sudo Etc Fstab](./image/sudo-dbserver-etc-fstab.PNG)

- Test the configuration and reload the daemon:

`sudo mount -a`

![Sudo Mount Dbserver Success](./image/sudo-mount-a.PNG)

`sudo systemctl daemon-reload`

![System Reload Dbserver Success](./image/system-reload-success.PNG)

- Verify your setup by running df -h, output must look like this:

`df -h`

![Verify Dbserver Setup](./image/verify-dbserver-setup-output.PNG)

- Install WordPress on your Web Server EC2.

- Update the repository:

`sudo yum -y update`

![Sudo Webserver Yum Update](./image/yum-update-output.PNG)

- Install wget, Apache and it’s dependencies:

`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

![Yum Php](./image/yum-php-output.PNG)

- Start Apache:

`sudo systemctl enable httpd`

![Enable Httpd](./image/httpd-enable-output.PNG)

`sudo systemctl start httpd`

![Start Httpd](./image/httpd-start-output.PNG)

- To install PHP and it’s depemdencies:

`sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

![Epel Output](./image/epel-release-output.PNG)

- Install Yum Utils:

`sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm -y`

![Epel Output](./image/epel-release-output.PNG)

`sudo yum module list php`

![Php Module List](./image/module-list-php.PNG)

`sudo yum module reset php`

![Php Module List](./image/module-reset-php.PNG)

`sudo yum module enable php:remi-8.0`

- Finally, install PHP, PHP-FPM (FastCGI Process Manager) and associated PHP modules using the command:

`sudo dnf install php php-opcache php-gd php-curl php-mysqlnd`

![Php Process Manager](./image/php-pmanager-output.PNG)

- To verify the version installed to run:

`sudo dnf install php php-opcache php-gd php-curl php-mysqlnd`

![Php Version](./image/php-version.PNG)

- PHP 8.0 installed. Equally important, we need to start and enable PHP-FPM on boot-up:

`sudo systemctl start php-fpm`

`sudo systemctl enable php-fpm`

![Php Start Enable](./image/php-start-enable-output.PNG)

- To check its status execute the command:

`sudo systemctl status php-fpm`

![Php Status Check](./image/php-success-output.PNG)

- To instruct SELinux to allow Apache to execute the PHP code via PHP-FPM run:

`sudo setsebool -P httpd_execmem 1`

![Setse Bool](./image/setsebool-output.PNG)

- Restart Apache:

`sudo systemctl restart httpd`

`sudo systemctl status httpd`

![Htppd Active Status](./image/httpd-status-output.PNG)

- Download wordpress and copy wordpress to var/www/html:

`mkdir wordpress`

`cd wordpress`

![Cd to Wordpress](./image/mk-cd-wordpress.PNG)

`sudo wget http://wordpress.org/latest.tar.gz`

![Wordpress Output](./image/wordpress-org-output.PNG)

- Extract Wordpress -

`sudo tar xzvf latest.tar.gz`

![Wordpress Extract Output](./image/wordpress-extract.PNG)

`ls -l`

![Wordpress Folder](./image/wordpress-folder.PNG)

`cd wordpress/`

![Change to Wordpress Folder](./image/wordpress-wordpress-output.PNG)

`sudo cp -R wp-config-sample.php wp-config.php`

`ls`

![Line 469 & 471](./image/copy-paste-output.PNG)

`pwd`

`cd`

![Switch Wordpress Folder](./image/switch-to-wordpress-folder.PNG)

`sudo cp -R wordpress/ /var/www/html`

`cd /var/www/html`

![Cd to Html folder](./image/cd-to-html.PNG)

- To remove unexpected file:

`ls -l`

`sudo rm -rf wordpress/`

![To remove unexpected file in Html](./image/to-remove-unexpected-file.PNG)

- To change back to wordpress folder:

`cd ../..`

`cd`

`ls`

`cd wordpress`

![To remove unexpected file in Html](./image/change-back-to-wordpress.PNG)

- Display wordpress content

`ls -l wordpress`

![Ls l Wordpress](./image/ls-l-wordpress.PNG)

- To copy content of wordpress to html:

`sudo cp -R wordpress/. /var/www/html`

`sudo ls -l /var/www/html`

![Copy Wordpress to Html](./image/copy-wordpress-html.PNG)

- To change from wordpress folder to html:

`cd /var/www/html/`

- On the Webserver install mysql:

`sudo yum install mysql-server -y`

`sudo systemctl start mysqld`

`sudo systemctl enable mysqld`

`sudo systemctl status mysqld`

![Copy Wordpress to Html](./image/start-enable-status-webserver.PNG)000

- Install MySQL on your DB Server EC2:

`sudo yum update`

![Yum Update](./image/yum-update-db-output.PNG)

`sudo yum install mysql-server -y`

![Yum Mysql Server Install](./image/yum-db-mysql-install.PNG)

- Verify that the service is up and running by using sudo systemctl status mysqld, if it is not running, restart the service and enable it so it will be running even after reboot:

`sudo systemctl restart mysqld`

![Restart Mysqld](./image/restart-mysqld-output.PNG)

`sudo systemctl enable mysqld`

![Enable Mysqld](./image/enable-mysqld-output.PNG)

`sudo systemctl status mysqld`

![Status Mysqld Check](./image/start-enable-status-dataBserver.PNG)

`sudo mysql_secure_installation`

![Status Mysqld Check](./image/mysql-install-status.PNG)

- Configure DB to work with WordPress

`sudo mysql -u root -p`

![Mysql Launch](./image/mysql-launch-status.PNG)

- Create database:

`CREATE DATABASE wordpress;`

![Create Wordpress Db](./image/create-db-wordpress.PNG)

- Create User:

`CREATE DATABASE wordpress;`

`SHOW DATABASES;`

![Show Databases](./image/show-databases-output.PNG)

- Create user:

`CREATE USER myuser@<Web-Server-Private-IP-Address> IDENTIFIED BY 'mypass';`

`GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';`

`FLUSH PRIVILEGES;`

`Exit`

- To edit bind address:

`sudo vi /etc/my.cnf`

- Restart Mysql:

`sudo systemctl restart mysqld`

- Back to WebServer:

`sudo vi wp-config.php`

![Show Databases](./image/db-edit-output.PNG)

`sudo systemctl restart httpd`

- Disable Apache:

`sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup`

- Comuunication between webserver and dbserver:

![Show Databases](./image/webdserver-and-dbserver-communication-success.PNG)

- Chnage permissions and configurations so Apache can use WordPress:

`sudo chown -R apache:apache /var/www/html/`

Run `ls -l`

![Step 619-621](./image/apache-on-apache.PNG)

`sudo chcon -t httpd_sys_rw_content_t /var/www/html/ -R`

`sudo setsebool -P httpd_can_network_connect=1`

`sudo setsebool -P httpd_can_network_connect_db 1`

![Word](./image/wordpress-success-output.PNG)