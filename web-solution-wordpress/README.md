Here's the content extracted from the provided Markdown document:

---

# Web Solution with WordPress

## Step 1 — Prepare a Web Server

1. Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB. [Learn How to Add EBS Volume to an EC2 instance](https://www.youtube.com/watch?v=HPXnXkBzIHw)
> use `RedHat` linux option for the instance

![](./assests/Screenshot%202024-05-30%20at%203.14.15%20PM.png)

2. Attach all three volumes one by one to your Web Server EC2 instance
![](./assests/Screenshot%202024-05-30%20at%203.17.50%20PM.png)

3. Open up the Linux terminal to begin configuration
![](./assests/Screenshot%202024-05-30%20at%203.42.01%20PM.png)

   - Use `lsblk` command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in `/dev/` directory. Inspect it with `ls /dev/` and make sure you see all 3 newly created block devices there - their names will likely be `xvdf`, `xvdh`, `xvdg` or in my case `xvdbf`, `xvdbh`, `xvdbg`.
   ```sh
   lsblk
   ```
   ![](./assests/Screenshot%202024-05-30%20at%203.17.59%20PM.png)
4. Use `df -h` command to see all mounts and free space on your server
   ![](./assests/Screenshot%202024-05-30%20at%204.02.50%20PM.png)
5. Use `gdisk` utility to create a single partition on each of the 3 disks
   ```
   sudo gdisk /dev/xvdbf
   sudo gdisk /dev/xvdbg
   sudo gdisk /dev/xvdbh
   ```
   > This will prompt you to input some option, these option should be done in this order `n`, `p`, `w` and `y`

   `/dev/xvdbf`
   ![](./assests/Screenshot%202024-05-30%20at%203.20.20%20PM.png)

   `/dev/xvdbg`
   ![](./assests/Screenshot%202024-05-30%20at%203.21.01%20PM.png)
   
   `/dev/xvdbh`
   ![](./assests/Screenshot%202024-05-30%20at%203.21.30%20PM.png)
6. Use `lsblk` utility to view the newly configured partition on each of the 3 disks.
![](./assests/Screenshot%202024-05-30%20at%203.21.59%20PM.png)

7. Install `lvm2` package using `sudo yum install lvm2`. Run `sudo lvmdiskscan` command to check for available partitions.
```sh
sudo yum install lvm2
sudo lvmdiskscan
```
![](./assests/Screenshot%202024-05-30%20at%203.23.00%20PM.png)

8. Use `pvcreate` utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM
   ```
   sudo pvcreate /dev/xvdbf1
   sudo pvcreate /dev/xvdbg1
   sudo pvcreate /dev/xvdbh1
   ```
   ![](./assests/Screenshot%202024-05-30%20at%203.23.36%20PM.png)

9. Verify that your Physical volume has been created successfully by running `sudo pvs`
   ![](./assests/Screenshot%202024-05-30%20at%203.24.04%20PM.png)

10. Use `vgcreate` utility to add all 3 PVs to a volume group (VG). Name the VG `webdata-vg`
    ```
    sudo vgcreate webdata-vg /dev/xvdbh1 /dev/xvdbg1 /dev/xvdbf1
    ```
   ![](./assests/Screenshot%202024-05-30%20at%203.24.41%20PM.png)
11. Verify that your VG has been created successfully by running `sudo vgs`
   ![](./assests/Screenshot%202024-05-30%20at%203.25.05%20PM.png)
12. Use `lvcreate` utility to create 2 logical volumes. `apps-lv` (Use half of the PV size), and `logs-lv` (Use the remaining space of the PV size). 

> NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

    ```
    sudo lvcreate -n apps-lv -L 14G webdata-vg
    sudo lvcreate -n logs-lv -L 14G webdata-vg
    ```
   ![](./assests/Screenshot%202024-05-30%20at%203.25.52%20PM.png)

13. Verify that your Logical Volume has been created successfully by running `sudo lvs`
   ![](./assests/Screenshot%202024-05-30%20at%203.26.15%20PM.png)

13. Verify the entire setup
   ```sh
   sudo vgdisplay -v #view complete setup - VG, PV, and LV
   ```
   ![](./assests/Screenshot%202024-05-30%20at%203.26.50%20PM.png)

   and
   ```sh
   sudo lsblk 
   ```

   ![](./assests/Screenshot%202024-05-30%20at%203.27.07%20PM.png)   

14. Use `mkfs.ext4` to format the logical volumes with ext4 filesystem
    ```
    sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
    sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
    ```
    ![](./assests/Screenshot%202024-05-30%20at%203.28.18%20PM.png)

15. Create `/var/www/html` directory to store website files 
   ```sh
   sudo mkdir -p /var/www/html
   ```

16. Create `/home/recovery/logs` to store backup of log data 
   ```sh
   sudo mkdir -p /home/recovery/logs
   ```
17. Mount `/var/www/html` on `apps-lv` logical volume
    ```
    sudo mount /dev/webdata-vg/apps-lv /var/www/html/
    ```
   ![](./assests/Screenshot%202024-05-30%20at%203.29.30%20PM.png)

18. Use `rsync` utility to backup all the files in the log directory `/var/log` into `/home/recovery/logs/`
    ```
    sudo rsync -av /var/log/ /home/recovery/logs/
    ```

19. Mount `/var/log` on `logs-lv` logical volume. (Note that all the existing data on `/var/log` will be deleted. That is why step 15 above is very important)
    ```
    sudo mount /dev/webdata-vg/logs-lv /var/log
    ```

20. Restore log files back into `/var/log` directory
    ```
    sudo rsync -av /home/recovery/logs/ /var/log
    ```

   ![](./assests/Screenshot%202024-05-30%20at%203.30.40%20PM.png)

21. Update `/etc/fstab` file so that the mount configuration will persist after restart of the server. use `blkid` to see the UUID of the device
   ```sh
   sudo blkid
   ```
   ![](./assests/Screenshot%202024-05-30%20at%203.31.31%20PM.png)

   Open the `/etc.fstab` file and include the UUID for `/dev/mapper/webdata-vg-logs--1v:` and `/dev/mapper/webdata--vg-apps--1v:`. include the ids as seen below
   ![](./assests/Screenshot%202024-05-30%20at%203.38.10%20PM.png)

22. Test the configuration and reload the daemon
    ```
    sudo mount -a
    sudo systemctl daemon-reload
    ```
   ![](./assests/Screenshot%202024-05-30%20at%203.39.41%20PM.png)

### Step 2 — Prepare the Database Server

Launch a second RedHat EC2 instance that will have a role - 'DB Server'. Repeat the same steps as for the Web Server, but instead of creating `apps-lv`, create `db-lv` and mount it to `/db` directory instead of `/var/www/html/`. This ensures that the database server has dedicated storage for its operations.

This step ensures that your database server is properly configured and has dedicated storage for its operations. Having separate storage for the database ensures better performance and easier management of resources.
![](./assests/Screenshot%202024-05-30%20at%203.41.41%20PM.png)

![](./assests/Screenshot%202024-05-30%20at%203.45.49%20PM.png)

![](./assests/Screenshot%202024-05-30%20at%203.42.01%20PM.png)

Repeat Every Step taking note of **Point 12** in [Step 1](#step-1--prepare-a-web-server) as seen below
![](./assests/Screenshot%202024-05-30%20at%203.54.50%20PM.png)
![](./assests/Screenshot%202024-05-30%20at%203.57.40%20PM.png)
![](./assests/Screenshot%202024-05-30%20at%204.01.47%20PM.png)
![](./assests/Screenshot%202024-05-30%20at%204.01.56%20PM.png)
### Step 3 — Install WordPress on your Web Server EC2

1. **Update the repository:**
   ```bash
   sudo yum -y update
   ```
   ![](./assests/Screenshot%202024-05-30%20at%204.03.45%20PM.png)

2. **Install wget, Apache and its dependencies:**
   ```bash
   sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
   ```
   ![](./assests/Screenshot%202024-05-30%20at%204.09.18%20PM.png)
3. **Start Apache:**
   ```bash
   sudo systemctl enable httpd
   sudo systemctl start httpd
   ```
   ![](./assests/Screenshot%202024-05-30%20at%204.08.59%20PM.png)

4. **Install PHP and its dependencies:**
   ```bash
   sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
   sudo yum install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm
   ```
   ![](./assests/Screenshot%202024-05-30%20at%204.34.11%20PM.png)

   ![](./assests/Screenshot%202024-05-30%20at%204.34.29%20PM.png)

   ```sh
   sudo yum-config-manager --enable remi
   sudo yum module list php
   sudo yum module reset php
   sudo yum module enable php:remi-7.4
   sudo yum install php php-opcache php-gd php-curl php-mysqlnd
   sudo systemctl start php-fpm
   sudo systemctl enable php-fpm
   setsebool -P httpd_execmem 1
   ```
![](./assests/Screenshot%202024-05-30%20at%205.57.53%20PM.png)

![](./assests/Screenshot%202024-05-30%20at%205.58.02%20PM.png)

![](./assests/Screenshot%202024-05-30%20at%206.01.21%20PM.png)

![](./assests/Screenshot%202024-05-30%20at%206.03.02%20PM.png)

5. **Restart Apache:**
   ```bash
   sudo systemctl restart httpd
   ```
![](./assests/Screenshot%202024-05-30%20at%206.04.39%20PM.png)

6. **Download WordPress and copy it to `/var/www/html`:**
   ```bash
   mkdir wordpress
   cd wordpress
   sudo wget http://wordpress.org/latest.tar.gz
   sudo tar xzvf latest.tar.gz
   sudo rm -rf latest.tar.gz
   cp wordpress/wp-config-sample.php wordpress/wp-config.php
   cp -R wordpress /var/www/html/
   ```
![](./assests/Screenshot%202024-05-30%20at%206.04.47%20PM.png)

![](./assests/Screenshot%202024-05-30%20at%206.06.15%20PM.png)


7. **Configure SELinux Policies:**
   ```bash
   sudo chown -R apache:apache /var/www/html/wordpress
   sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
   sudo setsebool -P httpd_can_network_connect=1
   ```
![](./assests/Screenshot%202024-05-30%20at%206.07.54%20PM.png)

### Step 4 — Install MySQL on your DB Server EC2

```bash
sudo yum update
sudo yum install mysql-server
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```
![](./assests/Screenshot%202024-05-30%20at%206.12.49%20PM.png)
![](./assests/Screenshot%202024-05-30%20at%206.12.57%20PM.png)

### Step 5 — Configure DB to work with WordPress

```sql
sudo mysql
CREATE DATABASE wordpress;
CREATE USER 'myuser'@'<Web-Server-Private-IP-Address>' IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```
![](./assests/Screenshot%202024-05-30%20at%206.13.57%20PM.png)

![](./assests/Screenshot%202024-05-30%20at%206.16.13%20PM.png)

> Remeber to update the `inbound rule` for the security group on the AWS to allow MySQl at port `3306` to be accesss through the `Web Server` ip

![](./assests/Screenshot%202024-05-30%20at%206.18.51%20PM.png)

### Step 6 — Configure WordPress to connect to a remote database

1. **Install MySQL client and test the connection from your Web Server to your DB server:**
   ```bash
   sudo yum install mysql
   sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
   ```
![](./assests/Screenshot%202024-05-30%20at%206.21.52%20PM.png)

2. **Verify if you can execute `SHOW DATABASES;` command successfully.**
![](./assests/Screenshot%202024-05-30%20at%206.21.52%20PM.png)

3. **Visit the remote server on your browser, using your EC2 IPv4 address at the path `/wordpress`

if you get this error
![](./assests/Screenshot%202024-05-30%20at%206.55.24%20PM.png)

3. **Change permissions and configuration so Apache could use WordPress.**
Open the file `/var/www/html/wordpress/wp-config.php` and update your database information
```sh
sudo vi /var/www/html/wordpress/wp-config.php
```

![](./assests/Screenshot%202024-05-30%20at%207.57.35%20PM.png)

4. **Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2.**
![](./assests/Screenshot%202024-05-30%20at%206.18.51%20PM.png)

5. **Access your WordPress site from the browser using `http://<Web-Server-Public-IP-Address>/wordpress/`.**
![](./assests/Screenshot%202024-05-30%20at%207.54.57%20PM.png)
![](./assests/Screenshot%202024-05-30%20at%207.55.08%20PM.png)
![](./assests/Screenshot%202024-05-30%20at%207.56.37%20PM.png)

These steps should guide you through the installation and configuration of WordPress on your EC2 instances.