# Deployment guide for eSchool Application (https://github.com/yurkovskiy)

## Overview

This document provides a comprehensive summary of the actions performed to deploy and configure the eSchool web application, along with troubleshooting tips and steps to access resources. 
This project is beginner project for a DevOps related role.

---

## Deployment Steps

### 1. **Provision Azure VMs**

- Two VMs were deployed using Terraform:
  - **VM1**: Hosted the eSchool web application.
  - **VM2**: Hosted the MySQL database server.

### 2. **Prepare VM2 (MySQL Server)**

1. **Install MySQL**:
   ```bash
   sudo apt update
   sudo apt install mysql-server -y
   ```
2. **Secure MySQL**:
   ```bash
   sudo mysql_secure_installation
   ```
3. **Create MySQL User and Database**:
   - Log in to MySQL: `sudo mysql`
   - Run:
     ```sql
     CREATE DATABASE eschool;
     CREATE USER 'eschooluser'@'%' IDENTIFIED BY 'Select_StrongPassword123';
     GRANT ALL PRIVILEGES ON eschool.* TO 'eschooluser'@'%';
     FLUSH PRIVILEGES;
     ```
4. **Update Firewall Rules**:
   - Allow traffic on port `3306` from VM1.
   ```bash
   sudo ufw allow from <VM1_PRIVATE_IP> to any port 3306
   sudo ufw enable
   ```

### 3. **Prepare VM1 (eSchool Application)**

1. **Install Required Software**:
   ```bash
   sudo apt update
   sudo apt install openjdk-8-jdk maven git -y
   ```
2. **Clone eSchool Repository**:
   ```bash
   git clone https://github.com/yurkovskiy/eSchool.git
   ```
3. **Update Configuration**:
   - Edit `application.properties` in the `eSchool` folder:
     ```properties
     spring.datasource.url=jdbc:mysql://<VM2_PRIVATE_IP>:3306/eschool
     spring.datasource.username=eschooluser
     spring.datasource.password=Select_StrongPassword123
     ```
4. **Build the Application**:
   ```bash
   mvn clean install
   ```
5. **Run the Application**:
   ```bash
   java -jar target/eschool.jar
   ```

### 4. **Configure Application Startupapplication.properties**

1. Create a systemd service:
   ```bash
   sudo nano /etc/systemd/system/eschool.service
   ```
   Add the following:
   ```ini
   [Unit]
   Description=eSchool Web Application
   After=network.target

   [Service]
   User=<YOUR_USERNAME>
   WorkingDirectory=/home/<YOUR_USERNAME>/eSchool
   ExecStart=/usr/bin/java -jar /home/<YOUR_USERNAME>/eSchool/target/eschool.jar
   Restart=always

   [Install]
   WantedBy=multi-user.target
   ```
2. Enable and Start the Service:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable eschool.service
   sudo systemctl start eschool.service
   ```
3. Verify the Application:
   ```bash
   systemctl status eschool.service
   ```

---

## Accessing Resources

### 1. **Access the Web Application**

- Open a browser and navigate to: `http://<VM1_PUBLIC_IP>:8081`
- Login credentials:
  - **Username**: `admin`
  - **Password**: `admin`

### 2. **Access MySQL Server from VM1**

- Connect using MySQL client:
  ```bash
  mysql -h <VM2_PRIVATE_IP> -u eschooluser -p
  ```

---

## Troubleshooting

### 1. **Application Not Starting**

- **Check logs**:
  ```bash
  sudo journalctl -u eschool.service
  ```
- **Possible Issue**: Port `8081` already in use.
  - Solution: Kill the process using the port:
    ```bash
    sudo netstat -tuln | grep 8081
    sudo kill <PID>
    ```

### 2. **Cannot Connect to MySQL from VM1**

- **Check Firewall Rules**:
  ```bash
  sudo ufw status
  ```
- **Verify MySQL Bind Address**:
  - Edit `/etc/mysql/mysql.conf.d/mysqld.cnf`:
    ```ini
    bind-address = 0.0.0.0
    ```
  - Restart MySQL:
    ```bash
    sudo systemctl restart mysql
    ```

### 3. **Service Fails to Start**

- Verify the `eschool.service` configuration file for correct paths and user.
- Run the application manually to debug:
  ```bash
  java -jar /home/<YOUR_USERNAME>/eSchool/target/eschool.jar
  ```

---

## Summary

The eSchool web application is deployed and accessible via `http://<VM1_PUBLIC_IP>:8081`. The MySQL database is configured and accessible from VM1. The application is set to start automatically on VM1 startup. For any issues, refer to the troubleshooting section.

---

## Notes

- Ensure to keep sensitive credentials secure.
- Monitor VM performance periodically and scale as needed.

