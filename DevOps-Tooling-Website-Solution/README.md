# DevOps Tooling Website Solution

## Introduction

This project involves implementing a solution that consists of the following components:

-   **Infrastructure**: AWS
-   **Web Server Linux**: Red Hat Enterprise Linux 9
-   **Database Server**: Ubuntu Linux + MySQL
-   **Storage Server**: Red Hat Enterprise Linux 9 + NFS Server
-   **Programming Language**: PHP
-   **Code Repository**: GitHub

The diagram below shows the architecture of the solution.

![3 tier](./assets/3-Tier.png)

## Step 1 - Prepare NFS Server

1. **Spin up an EC2 instance with RHEL Operating System, use RedHat on AWS**

    ![NFS](./assets/Screenshot%202024-06-08%20at%202.01.45%20PM.png)

2. **Configure Logical Volume Management (LVM) on the server**

    - Format the LVM as xfs.
    - Create 3 Logical Volumes: `lv-opt`, `lv-apps`, `lv-logs`.
    - Create mount points on the `/mnt` directory for the logical volumes as follows:

        - Mount `lv-apps` on `/mnt/apps` for web servers.
        - Mount `lv-logs` on `/mnt/logs` for web server logs.
        - Mount `lv-opt` on `/mnt/opt` for the Jenkins server in the next project.

    - **Create 3 volumes in the same AZ as the NFS Server EC2, each of 10GB, and attach them to the NFS Server.**

        ![nfs volumes](./assets/Screenshot%202024-06-08%20at%202.13.20%20PM.png)

    - **Open the Linux terminal to begin configuration.**

        ```bash
        ssh -i "my-devec2key.pem" ec2-user@18.224.38.96
        ```

        ![NFS ssh](./assets/Screenshot%202024-06-08%20at%202.14.02%20PM.png)

    - **Use `lsblk` to inspect what block devices are attached to the server.**

        ```bash
        lsblk
        ```

        ![List Block](./assets/Screenshot%202024-06-08%20at%202.14.40%20PM.png)

    - **Use `gdisk` to create a single partition on each of the 3 disks.**

        ```bash
        sudo gdisk /dev/xvdf
        sudo gdisk /dev/xvdg
        sudo gdisk /dev/xvdh
        ```

        ![partition](./assets/Screenshot%202024-06-08%20at%202.15.29%20PM.png)
        ![partition](./assets/Screenshot%202024-06-08%20at%202.16.31%20PM.png)
        ![partition](./assets/Screenshot%202024-06-08%20at%202.17.00%20PM.png)

    - **Use `lsblk` to view the newly configured partitions.**

        ```bash
        lsblk
        ```

        ![View partitions](./assets/Screenshot%202024-06-08%20at%202.17.20%20PM.png)

    - **Install `lvm2` package.**

        ```bash
        sudo yum install lvm2 -y
        ```

        ![Install lvm](./assets/Screenshot%202024-06-08%20at%202.18.03%20PM.png)

    - **Use `pvcreate` to mark each of the 3 disks as physical volumes (PVs) for LVM.**

        ```bash
        sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
        sudo pvs
        ```

        ![PVs](./assets/Screenshot%202024-06-08%20at%202.18.41%20PM.png)

    - **Use `vgcreate` to add all 3 PVs to a volume group (VG) named `webdata-vg`.**

        ```bash
        sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
        sudo vgs
        ```

        ![VG](./assets/Screenshot%202024-06-08%20at%202.18.56%20PM.png)

    - **Use `lvcreate` to create 3 logical volumes: `lv-apps`, `lv-logs`, and `lv-opt`.**

        ```bash
        sudo lvcreate -n lv-apps -L 9G webdata-vg
        sudo lvcreate -n lv-logs -L 9G webdata-vg
        sudo lvcreate -n lv-opt -L 9G webdata-vg
        sudo lvs
        ```

        ![LV](./assets/Screenshot%202024-06-08%20at%202.19.44%20PM.png)

    - **Verify the entire setup.**

        ```bash
        sudo vgdisplay -v   #view complete setup, VG, PV and LV
        lsblk
        ```

        ![VG display](./assets/Screenshot%202024-06-08%20at%202.21.18%20PM.png)
        ![List Block](./assets/Screenshot%202024-06-08%20at%202.21.43%20PM.png)

    - **Format the logical volumes with the xfs filesystem.**

        ```bash
        sudo mkfs -t xfs /dev/webdata-vg/lv-apps
        sudo mkfs -t xfs /dev/webdata-vg/lv-logs
        sudo mkfs -t xfs /dev/webdata-vg/lv-opt
        ```

        ![filesystem](./assets/Screenshot%202024-06-08%20at%202.22.36%20PM.png)

    - **Create mount points in the `/mnt` directory.**

        ```bash
        sudo mkdir /mnt/apps
        sudo mkdir /mnt/logs
        sudo mkdir /mnt/opt
        sudo mount /dev/webdata-vg/lv-apps /mnt/apps
        sudo mount /dev/webdata-vg/lv-logs /mnt/logs
        sudo mount /dev/webdata-vg/lv-opt /mnt/opt
        ```

        ![mount dir](./assets/Screenshot%202024-06-08%20at%202.24.12%20PM.png)

3. **Install NFS Server, configure it to start on reboot, and ensure it is up and running.**

    ```bash
    sudo yum update -y
    sudo yum install nfs-utils -y
    sudo systemctl start nfs-server.service
    sudo systemctl enable nfs-server.service
    sudo systemctl status nfs-server.service
    ```

    ![install nfs](./assets/Screenshot%202024-06-08%20at%202.24.42%20PM.png)
    ![install nfs](./assets/Screenshot%202024-06-08%20at%202.28.56%20PM.png)
    ![start nfs](./assets/Screenshot%202024-06-08%20at%202.29.37%20PM.png)

4. **Export the mounts for Webservers' subnet CIDR to connect as clients.**

    - Set up permissions to allow the Web Servers to read, write, and execute files on NFS.

        ```bash
        sudo chown -R nobody: /mnt/apps
        sudo chown -R nobody: /mnt/logs
        sudo chown -R nobody: /mnt/opt
        sudo chmod -R 777 /mnt/apps
        sudo chmod -R 777 /mnt/logs
        sudo chmod -R 777 /mnt/opt
        sudo systemctl restart nfs-server.service
        ```

        ![permissions](./assets/Screenshot%202024-06-08%20at%202.31.49%20PM.png)

    - Configure access to NFS for clients within the same subnet.

        ```bash
        sudo vi /etc/exports

        /mnt/apps 172.31.0.0/16(rw,sync,no_all_squash,no_root_squash)
        /mnt/logs 172.31.0.0/16(rw,sync,no_all_squash,no_root_squash)
        /mnt/opt 172.31.0.0/16(rw,sync,no_all_squash,no_root_squash)

        sudo exportfs -arv
        ```

        > NB use your CIDR subnet ip

    ![nfs port](./assets/Screenshot%202024-06-08%20at%204.06.35%20PM.png)
    ![nfs port](./assets/Screenshot%202024-06-08%20at%204.06.54%20PM.png)

5. **Check which port is used by NFS and open it using the security group.**

    ```bash
    rpcinfo -p | grep nfs
    ```

    ![](./assets/Screenshot%202024-06-08%20at%202.42.10%20PM.png)

    **Note**: For NFS Server to be accessible from the client, the following ports must be opened: TCP 111, UDP 111, UDP 2049, NFS 2049. Set the Web Server subnet CIDR as the source.

    ![nfs security rule](./assets/Screenshot%202024-06-08%20at%204.04.58%20PM.png)

## Step 2 - Configure the Database Server

1. **Launch an Ubuntu EC2 instance for the DB Server.**

    ![ec2 db detail](./assets/Screenshot%202024-06-08%20at%205.28.20%20PM.png)

2. **Access the instance to begin configuration.**

    ```bash
    ssh -i "my-devec2key.pem" ubuntu@18.116.87.242
    ```

    ![ssh](./assets/Screenshot%202024-06-08%20at%202.45.48%20PM.png)

3. **Update and upgrade Ubuntu.**

    ```bash
    sudo apt update && sudo apt upgrade -y
    ```

    ![update ubuntu](./assets/Screenshot%202024-06-08%20at%202.45.58%20PM.png)

4. **Install MySQL Server.**

    - Install MySQL server.

        ```bash
        sudo apt install mysql-server
        ```

        ![install mysql](./assets/Screenshot%202024-06-08%20at%202.47.27%20PM.png)

    - Run MySQL secure installation script.

        ```bash
        sudo mysql_secure_installation
        ```

5. **Create a database named `tooling`.**

6. **Create a database user named `webaccess`.**

7. **Grant permission to `webaccess` user on `tooling` database from the webservers' subnet CIDR.**

    ```sql
    sudo mysql

    CREATE DATABASE tooling;
    CREATE USER 'webaccess'@'172.31.0.0/16' IDENTIFIED WITH mysql_native_password BY 'Admin123$';
    GRANT ALL PRIVILEGES ON tooling.* TO 'webaccess'@'172.31.0.0/16' WITH GRANT OPTION;
    FLUSH PRIVILEGES;
    show databases;

    use tooling;
    select host, user from mysql.user;
    exit
    ```

    ![create db](./assets/Screenshot%202024-06-08%20at%202.48.49%20PM.png)
    ![create db](./assets/Screenshot%202024-06-08%20at%206.33.50%20PM.png)

8. \*\*Set Bind Address and

port (3306) for MySQL in the configuration file.\*\*

-   Edit MySQL config file to allow connections from the webservers subnet.

    ```bash
    sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
    ```

-   Change the `bind-address` to allow connections from any IP address.

    ```ini
    bind-address = 0.0.0.0
    ```

    ![bind address](./assets/Screenshot%202024-06-08%20at%202.54.54%20PM.png)

9. **Restart MySQL to apply changes.**

    ```bash
    sudo systemctl restart mysql
    sudo systemctl status mysql
    ```

    ![restart mysql](./assets/Screenshot%202024-06-08%20at%202.55.25%20PM.png)

10. **Open port 3306 on the database security group to allow access from the webserver subnet CIDR.**

## Step 3 - Prepare the Web Server

1. **Launch 3 RHEL EC2 instances for the web servers.**

    ![web1](./assets/Screenshot%202024-06-08%20at%205.39.05%20PM.png)

2. **Install NFS client on both servers and mount the NFS directory.**

    - SSH into each web server instance.

        ```bash
        ssh -i "my-devec2key.pem" ec2-user@3.140.255.174
        ssh -i "my-devec2key.pem" ec2-user@3.129.221.24
        ```

        ![ssh web1](./assets/Screenshot%202024-06-08%20at%203.09.27%20PM.png)

    - Install NFS client on both servers.

        ```bash
        sudo yum install nfs-utils nfs4-acl-tools -y
        sudo systemctl start nfs-client.target
        sudo systemctl enable nfs-client.target
        sudo systemctl status nfs-client.target
        ```

        ![install nfs](./assets/Screenshot%202024-06-08%20at%203.09.41%20PM.png)

    - Mount NFS shares from the NFS server to the web servers.

        ```bash
        sudo mkdir /var/www
        sudo mkdir /var/log/httpd
        sudo mount -t nfs4 -o rw,sync,noatime,hard,intr,nolock 18.224.38.96:/mnt/apps /var/www
        sudo mount -t nfs4 -o rw,sync,noatime,hard,intr,nolock 18.224.38.96:/mnt/logs /var/log/httpd
        ```

        ![mount nfs](./assets/Screenshot%202024-06-08%20at%203.14.10%20PM.png)
        ![](./assets/Screenshot%202024-06-08%20at%204.08.58%20PM.png)

    - Verify the mounts.

        ```bash
        df -h
        ```

        ![mount nfs](./assets/Screenshot%202024-06-08%20at%203.14.10%20PM.png)

3. **Install Apache and PHP on both web servers.**

    - Install Apache.

        ```bash
        sudo yum install httpd -y
        ```

        ![install apache](./assets/Screenshot%202024-06-08%20at%204.12.07%20PM.png)

    - Start and enable Apache.

        ```bash
        sudo systemctl start httpd
        sudo systemctl enable httpd
        sudo systemctl status httpd
        ```

        ![start apache](./assets/Screenshot%202024-06-08%20at%204.40.45%20PM.png)

    - Install PHP.

        ```bash
        sudo yum install php php-mysqlnd -y
        ```

        ![install php](./assets/Screenshot%202024-06-08%20at%204.38.45%20PM.png)

4. **Clone the repository from GitHub to the web servers.**
   fork this [repo](https://github.com/StegTechHub/tooling.git) on github and clone your forked version.

    ```bash
    sudo yum install git -y
    cd /var/www
    sudo git clone https://github.com/<your-github-username>/tooling.git
    ```

    ![clone repo](./assets/Screenshot%202024-06-08%20at%206.04.51%20PM.png)

    ```sh
    cd tooling
    ls
    ```

    ![](./assets/Screenshot%202024-06-08%20at%206.05.22%20PM.png)
    
    - copy the content of `tooling/html` to `/var/www/html`
    ```sh
      sudo cp -R html/. /var/www/html/
    ```
    ![](./assets/Screenshot%202024-06-08%20at%206.06.55%20PM.png)

5. **Configure the database connection in the application.**

    - Update the database connection settings in the application configuration file.

        ```bash
        sudo vi /var/www/html/functions.php
        ```

    - Set the following database connection parameters.

        ```php
        $db = mysqli_connect('18.116.87.242', 'webaccess', 'Admin123$', 'tooling');
        ```
    ![](./assets/config-function-php.png)

    ```sh
    sudo mysql -h 172.31.8.129 -u webaccess -p tooling < tooling-db.sql
    ```
  ![](./assets/tooling-script-db.png)
  - Access the database server from Web Server
  ```sh
    sudo mysql -h 172.31.8.129 -u webaccess -p
  ```
  ![](./assets/access-db-web1.png)

  - Create in MyQSL a new admin user with username: myuser and password: password

  ```sql
    INSERT INTO users(id, username, password, email, user_type, status) VALUES (2, 'myuser', '5f4dcc3b5aa765d61d8327deb882cf99', 'user@mail.com', 'admin', '1');
  ```

  ![](./assets/insert-rows.png)
6. **Test the application.**

    - Open the browser and navigate to the public IP address of the web servers.

  ![web app](./assets/Screenshot%202024-06-15%20at%208.54.03%20AM.png)
  ![](./assets/Screenshot%202024-06-15%20at%208.53.35%20AM.png)

## Conclusion

The project is successfully implemented with a fully functional 3-tier architecture, including an NFS server for shared storage, a database server for data management, and web servers serving the application.
