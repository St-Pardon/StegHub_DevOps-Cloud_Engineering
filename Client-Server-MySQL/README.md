## Setting Up a Client-Server Architecture with MySQL on AWS EC2

In today's cloud-centric world, leveraging cloud services to host your database servers can provide significant benefits in terms of scalability, availability, and cost-efficiency. Amazon Web Services (AWS) EC2 is a popular choice for hosting databases due to its flexibility and robust infrastructure. This article provides a comprehensive guide on setting up a client-server architecture with MySQL on an AWS EC2 instance.

### Step 1: Launch an EC2 Instance

The first step in setting up MySQL on AWS is to launch an EC2 instance. Follow these steps:

1. **Log in to AWS Management Console**: Navigate to the EC2 Dashboard and click on “Launch Instance”.
2. **Create 2 different EC2 instance**: One of the 2 will be used to set up the server and the other for the client
2. **Choose an Amazon Machine Image (AMI)**: Select a preferred Linux distribution, such as  Ubuntu.
![](./assets/Screenshot%202024-05-26%20at%203.04.29%20PM.png)
3. **Select an Instance Type**: For initial setups or small-scale use, `t2.micro` is a suitable choice and is eligible for the AWS Free Tier.
4. **Configure Instance Details**: Proceed with the default settings or customize according to your needs.
![](./assets/Screenshot%202024-05-26%20at%203.05.56%20PM.png)
5. **Configure Security Group**: Set up a security group to allow SSH access (port 22) to connect to your instance and also allow inboud on port 3306 attaching your client instance ip, this will be iused in communicating with the mysql server installed on the server. 
![](./assets/Screenshot%202024-05-26%20at%203.15.26%20PM.png)

### Step 2: Install MySQL Server

After launching your instance, connect to the one named `server` via SSH and install MySQL:

1. **Connect via SSH**:
   ```sh
   ssh -i /path/to/your-key-pair.pem ec2-user@your-ec2-public-dns
   ```
   ![](./assets/Screenshot%202024-05-26%20at%203.09.12%20PM.png)
2. **Update the Package Index**:
   ```sh
   sudo apt update -y   
   ```
   ![](./assets/Screenshot%202024-05-26%20at%203.07.39%20PM.png)
3. **Install MySQL Server**:
   ```sh
   sudo apt install mysql-server -y  
   ```
   ![](./assets/Screenshot%202024-05-26%20at%203.08.45%20PM.png)
### Step 3: Configure MySQL for Remote Access

By default, MySQL is configured to listen for connections only from localhost. To enable remote access:

1. **Start the MySQL Service**:
   ```sh
   sudo systemctl enable mysql  
   ```
   ![](./assets/Screenshot%202024-05-26%20at%203.08.54%20PM.png)
2. **Secure MySQL Installation**:
   ```sh
   sudo mysql_secure_installation
   ```
3. **Modify MySQL Configuration**:
   Open the MySQL configuration file and set it to listen on all IP addresses:
   ```sh
   sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
   ```
   Add or modify the following line:
   ```sh
   bind-address = 0.0.0.0
   ```
   ![](./assets/Screenshot%202024-05-26%20at%203.25.06%20PM.png)
4. **Restart MySQL Service**:
   ```sh
   sudo service mysqld restart   # For Amazon Linux
   sudo systemctl restart mysql  # For Ubuntu
   ```
   ![](./assets/Screenshot%202024-05-26%20at%203.24.44%20PM.png)
5. **Create a Remote User and Grant Privileges**:
   Log into MySQL:
   ```sh
   mysql -u root -p
   ```
   Inside the MySQL shell:
   ```sql
   CREATE USER 'clinetuser'@'%' IDENTIFIED With mysql_native_password BY 'Password1';
   GRANT ALL PRIVILEGES ON *.* TO 'clientuser'@'%';
   FLUSH PRIVILEGES;
   EXIT;
   ```
   ![](./assets/Screenshot%202024-05-26%20at%203.20.35%20PM.png)

### Step 4: Set Up Security Groups

To allow external connections to MySQL, modify your EC2 security group settings:

1. **Navigate to EC2 Dashboard**: Select Security Groups from the left-hand menu.
2. **Edit Inbound Rules**: Find your instance's security group and edit inbound rules to include:
   - **Type**: MySQL/Aurora
   - **Protocol**: TCP
   - **Port Range**: 3306
   - **Source**: Custom (specify your IP, client id or use `0.0.0.0/0` to allow all IPs, which is less secure)
   ![](./assets/Screenshot%202024-05-26%20at%203.15.26%20PM.png)

### Step 5: Connect from a Client

With MySQL configured for remote access, you can connect using a MySQL client tool or from the command line:

1. **Using Command Line**:
   ```sh
   mysql -h your-ec2-public-dns -u remote_user -p
   ```
2. **Update the instance**
   ```sh
   sudo apt update
   ```

2. **install mysql**
   ```sh
   sudo apt install mysql-server -y  
   ```
   > The 2 ssteps above were same as the server
4. **Start the MySQL Service**:
   ```sh
   sudo systemctl enable mysql  
   ```
5.  **Create a Remote User and Grant Privileges**:
   Log into MySQL:
   ```sh
   sudo mysql -u client -h 172.31.46.2 -p
   ```
   > use the password your created for the `clientuser` on the server
   ![](./assets/Screenshot%202024-05-26%20at%203.25.28%20PM.png)
   Inside the MySQL shell:
   ```sql
   SHOW databases;

   USE test;

   CREATE TABLE test(
    item_id INT AUTO_INCREMENT,
    content VARCHAR(255),
    PRIMARY KEY(item_id)
   );

   INSERT INTO test(content) VALUES ("Setting up react server"), ("Nginx and HAproxy load balancer");

   SELECT * FROM test;
   ```
![](./assets/Screenshot%202024-05-26%20at%203.25.56%20PM.png)
![](./assets/Screenshot%202024-05-26%20at%203.29.26%20PM.png)
![](./assets/Screenshot%202024-05-26%20at%203.29.58%20PM.png)

---
By following these steps, you can successfully set up a client-server architecture with MySQL on an AWS EC2 instance. This setup not only leverages the power of cloud computing but also ensures your database is scalable and accessible as needed.