# MySQL Preparation, Security, and Monitoring.

- Manual installation and configuration of MySQL on a Linux server <br>
- Best practices for securing MySQL, including user management and access control <br>
- Monitoring MySQL by phpMyAdmin <br>
- Scheduled Backup <br>
- Docker-based installation and management of MySQL <br>


## Compatibility Note

This tutorial is designed to be compatible with various Linux distributions, including Ubuntu, CentOS, Debian, and others. While the examples and commands provided are based on an Ubuntu server, they can be easily adapted to your specific Linux distribution.

### Package Manager Variations

- For Ubuntu and Debian-based distributions, the package manager used is `apt`. If you're using a different distribution, such as CentOS or Fedora, you can replace the `apt` commands with the appropriate package manager for your distribution (e.g., `yum` or `dnf`).
- Example: Replace `apt install package-name` with `yum install package-name` or `dnf install package-name`.

### Firewall Command Variations

- In this tutorial, the firewall commands are demonstrated using `ufw`, the default firewall configuration tool for Ubuntu. If you're using a different distribution, you can substitute the `ufw` commands with the appropriate firewall management commands for your distribution (e.g., `iptables` or `firewalld`).
- Example: Replace `ufw allow port` with the appropriate command for your firewall management tool.

Please ensure to consult the documentation or resources specific to your Linux distribution for accurate commands and package management guidelines.


## Manual installation and configuration of MySQL on a Linux server

### Step 1 — Installing MySQL

To install it, make sure you have recently updated the package index on your server:

```shell
sudo apt update
```
Next, proceed with the installation of the mysql-server package:

```shell
sudo apt install mysql-server
```
Ensure that the server is running using the systemctl start command:

```shell
sudo systemctl start mysql.service
```

### Step 2 — Configuring MySQL

First, open up the MySQL prompt:

```shell
sudo mysql
```
Subsequently, execute the ALTER USER command provided below to modify the authentication method of the root user to utilize a password. In this particular instance, the command alters the authentication method to mysql_native_password:

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```
Once you have made this modification, you can exit the MySQL prompt.

```shell
exit
```
During this process, you will be presented with a sequence of prompts allowing you to customize your MySQL installation's security settings. The initial prompt will inquire if you wish to configure the Validate Password Plugin, which can be utilized to assess the strength of passwords for new MySQL users before accepting them as valid.
<br>
If you choose to enable the Validate Password Plugin, any MySQL user you create and who authenticates with a password will need to adhere to the password policy you choose. The policy ensures that passwords meet certain criteria for security purposes.

```
Output
Securing the MySQL server deployment.

Connecting to MySQL using a blank password.

VALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?

Press y|Y for Yes, any other key for No: Y

There are three levels of password validation policy:

LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary                  

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG:
 2
```

Whether or not you decide to configure the Validate Password Plugin, the subsequent prompt will ask you to specify a password for the MySQL root user. Please enter a strong password of your preference, and then confirm it by re-entering it:

```
Output
Please set the password for root here.


New password:

Re-enter new password:
```
Please take note that although you have successfully set a password for the root MySQL user, the current configuration does not require the user to authenticate with a password when connecting to the MySQL shell.
<br>
If you have utilized the Validate Password Plugin, you will receive feedback on the strength of the password you have set. Afterwards, the script will prompt you to confirm whether you wish to proceed with the password you have entered or if you prefer to enter a new one. If you are content with the strength of the password you have just entered, please input "Y" to proceed with the script:

```
Output
Estimated strength of the password: 100
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : Y
```
At this point, you can press "Y" followed by the "ENTER" key to accept the default values for all subsequent inquiries. By doing so, you will remove any anonymous users and the test database, disable remote root logins, and enforce these new rules, ensuring that MySQL promptly recognizes and applies the modifications you have implemented.

<br>
Note: After the security script has finished, you have the option to revert the authentication method of the root user back to the default, which is auth_socket. If you wish to authenticate as the root MySQL user using a password, execute the following command:

```shell
mysql -u root -p
```
To revert back to the default authentication method, utilize the following command:

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH auth_socket;
```

### Step 3 — Creating a Dedicated MySQL User and Granting Privileges

After installing MySQL, a root user account is automatically created, providing you with comprehensive control over the MySQL server, including databases, tables, and users. However, it is recommended to avoid using this account for non-administrative tasks. Instead, it is advisable to create a new user account with appropriate privileges. The following steps explain how to use the root MySQL user to create a new user account and assign the necessary privileges:


```shell
sudo mysql
```
Or if you are using a password for root, and enter the root user's password when prompted:

```shell
mysql -u root -p
```
Create a new user account by running the following command:

```sql
CREATE USER 'new_username'@'host' IDENTIFIED BY 'password';
```

Replace 'new_username' with the desired username and 'password' with the password for the new user.  The 'host' portion refers to the host or location from which the user is allowed to connect. The 'host' can be specified using different options depending on your requirements. Here are some common options for specifying the host:
<br>

1. Specific IP Address: You can specify a specific IP address or IP range to restrict user access. For example:

    - '192.168.0.100' to allow connections from a specific IP address.
    - '192.168.0.%' to allow connections from a range of IP addresses within the specified subnet.

2. Any Host ('%'): You can use '%' to allow connections from any host. This is less restrictive but should be used with caution, especially in production environments.

3. Localhost ('localhost'): You can specify 'localhost' to restrict connections to the local machine only. This means the user can only connect from the same machine where the database server is running.

<br>

Grant necessary privileges to the new user account. For example, if you want to grant all privileges on a specific database, use the following command:

```sql
GRANT ALL PRIVILEGES ON database_name.* TO 'new_username'@'localhost';
```
Or On all databases using ON \*.\*
```sql
GRANT ALL PRIVILEGES ON *.* TO 'new_username'@'localhost';
```

To grant specific privileges to a specific database in SQL, you can use the GRANT statement. Here's an example SQL query to grant SELECT privileges to a user for a specific database:

```sql
GRANT SELECT ON database_name.* TO 'new_username'@'localhost';
```

Once you have granted the privileges, run the following command to refresh MySQL and make the changes take effect:

```sql
FLUSH PRIVILEGES;
```

### Step 4 — Testing MySQL

Irrespective of the installation method, MySQL should have started running automatically. To verify the status of MySQL, you can check it using the following command:

```shell
systemctl status mysql.service
```

### Allow mysql to access anywhere at port 3306

If you are using cloud hosting services, you need to allow inbound traffic on port 3306 in the Security Group configuration. This will enable external connections to reach the MySQL server. Make sure to add a rule to the Security Group allowing incoming traffic on port 3306 to ensure proper connectivity.

![Alt text](./Inbound%20rule.PNG?raw=true "MySQl server Inbound rule")

After updating the Security Group to allow inbound traffic on port 3306, you also need to update the firewall settings on your SSH server to allow the MySQL traffic. This step ensures that the SSH server allows incoming connections on port 3306 for MySQL. Make sure to configure the firewall settings accordingly to ensure seamless communication between the SSH server and MySQL.

```shell
sudo ufw allow 3306/tcp
```

In the next step, you will need to update the MySQL configuration. This involves modifying the configuration file to customize various settings based on your requirements. 

```shell
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

Update bind-address     =         127.0.0.1 to

```console
bind-address            = 0.0.0.0
```

By updating the bind-address configuration to 0.0.0.0, MySQL will be able to accept connections from any IP address. Please note that this change can potentially introduce security risks, so it's crucial to implement proper security measures such as firewall rules and authentication mechanisms to protect your MySQL server.
<br>
To restrict MySQL server access to specific IP addresses, you can modify the bind-address configuration to specify the desired IP addresses. This helps enhance security by allowing connections only from the specified IP addresses.
<br>
Here's how you can configure MySQL to allow connections from specific IP addresses:

```
bind-address = 192.168.0.100, 10.0.0.50
```

Restart the MySQL service for the changes to take effect.

```
sudo service mysql restart
```

## PhpMyAdmin

To host PhpMyAdmin, you will need a web server such as Nginx or Apache. In this case, we will use Nginx as the web server.

### Step 1 – Installing Nginx


As Nginx is readily available in the default repositories of Ubuntu, it is possible to install it using the apt packaging system from these repositories.
<br>
To begin, as this is our initial interaction with the apt packaging system in this session, we will update the local package index to ensure access to the latest package listings. Following that, we can proceed with the installation of Nginx by executing the following command:

```
sudo apt install nginx
```

### Step 2 - Download phpMyAdmin 

While phpMyAdmin is available in the Ubuntu 22.04 software repository, it is often recommended to install the latest version from the upstream package. You can download the latest version using the following command:
```shell
wget https://www.phpmyadmin.net/downloads/phpMyAdmin-latest-all-languages.zip
```
This command will download the latest version of phpMyAdmin in a compressed tarball format. After downloading, you can proceed with the installation and configuration of phpMyAdmin.
<br>
To install the `unzip` utility, which is necessary for extracting the contents of the downloaded phpMyAdmin tarball, you can use the following command:

```shell
sudo apt install unzip
```

Running this command will install `unzip` on your Ubuntu system, enabling you to extract the contents of the downloaded phpMyAdmin package.

<br>
To unzip the `phpMyAdmin-latest-all-languages.zip` file, you can use the following command:

```shell
unzip phpMyAdmin-latest-all-languages.zip
```

Make sure you are in the same directory where the `phpMyAdmin-latest-all-languages.zip` file is located. Running this command will extract the contents of the zip file into the current directory.
<br>
To move the extracted phpMyAdmin files to the `/var/www/` directory, you can use the `mv` command. Assuming you are in the same directory where the phpMyAdmin files are located, you can run the following command:

```shell
sudo mkdir -p /var/www/
sudo mv phpMyAdmin-X.X.X-all-languages /var/www/phpmyadmin
```

This command will move the entire `phpMyAdmin-X.X.X-all-languages` directory to the `/var/www/` directory. Ensure you have the necessary permissions to perform this operation by running the command with `sudo` (superuser) privileges.
<br>
Please replace "X.X.X" with the actual version number you have downloaded. This command will move the extracted phpMyAdmin files to the /var/www/phpmyadmin directory.

<br>

To change the ownership of the `/var/www/phpmyadmin` directory to the web server user (typically `www-data`), you can use the following command:

```shell
sudo chown -R www-data:www-data /var/www/phpmyadmin
```

Running this command with superuser privileges (`sudo`) will recursively change the ownership of all files and directories within `/var/www/phpmyadmin` to the `www-data` user and group. This ensures that the web server has the necessary permissions to access and serve the phpMyAdmin files.
<br>
To create the PhpMyAdmin configuration file, you can copy the default configuration file `config.sample.inc.php` to `config.inc.php` using the following command:

```shell
sudo cp config.sample.inc.php config.inc.php
```

This command will make a copy of the `config.sample.inc.php` file and name it `config.inc.php`. The `config.inc.php` file will serve as the configuration file for PhpMyAdmin. You can then modify the settings in `config.inc.php` to customize the behavior of PhpMyAdmin according to your requirements.

<br>
If you have the MySQL server installed on a different server and need to connect PhpMyAdmin to that server, you can modify the `config.inc.php` file to specify the IP address of the MySQL server.
<br>
Here's how you can change the `controlhost` option in the `config.inc.php` file:

1. Open the `config.inc.php` file in a text editor:
   ```shell
   sudo nano config.inc.php
   ```

2. Search for the line that contains the following configuration setting:
   ```php
   $cfg['Servers'][$i]['controlhost'] = 'localhost';
   ```

3. Modify the `'localhost'` value to the IP address of the MySQL server. For example:
   ```php
   $cfg['Servers'][$i]['controlhost'] = '192.168.0.100';
   ```

4. Save the changes to the `config.inc.php` file and exit the text editor.

By changing the `controlhost` option to the IP address of the MySQL server, PhpMyAdmin will connect to the specified server for managing the databases. Make sure you have the correct IP address and necessary network connectivity to establish a connection to the MySQL server.


### Step 3 - Install Required and Recommended PHP Modules.

To install the PHP modules required or recommended by phpMyAdmin, you can run the following command:

```shell
sudo apt install php-imagick php-phpseclib php-php-gettext php8.1-common php8.1-mysql php8.1-gd php8.1-imap php8.1-curl php8.1-zip php8.1-xml php8.1-mbstring php8.1-bz2 php8.1-intl php8.1-gmp php8.1-fpm
```

Executing this command with superuser privileges (`sudo`) will install the specified PHP modules on your Ubuntu system. These modules are commonly required or recommended for running phpMyAdmin smoothly.

### Step 4 - Create Nginx Server Block for phpMyAdmin

There are three hosting strategies for PhpMyAdmin:

1. Path: In this strategy, PhpMyAdmin is hosted under a path location, such as http://your_server_ip_or_domain/PhpMyAdmin. However, this strategy is not recommended due to potential security concerns.

2. Port Mapping: In this strategy, PhpMyAdmin is hosted under a specific port, such as http://your_server_ip_or_domain:3333. This strategy is recommended as it allows separate access to PhpMyAdmin using a different port, which can enhance security.

3. Subdomain: In this strategy, PhpMyAdmin is hosted under a subdomain, such as http://phpmyadmin.your_domain.com. This strategy is highly recommended as it provides a dedicated subdomain for PhpMyAdmin, making it easier to manage and access.

<br>
When choosing a hosting strategy for PhpMyAdmin, consider the security implications and ease of use for your specific setup. Port mapping and subdomain strategies are generally preferred due to better isolation and ease of management.


In this course, we will focus on using the port mapping and subdomain strategies for hosting PhpMyAdmin.

<br>
To create an Nginx server block for hosting PhpMyAdmin using the port mapping strategy (specifically under port 8080), you can follow these steps:

1. Open the Nginx configuration file for PhpMyAdmin using the Nano text editor:
   ```shell
   sudo nano /etc/nginx/conf.d/phpmyadmin.conf
   ```

2. Copy and paste the following configuration into the opened file:
   ```
    server {
    listen 8080;
    listen [::]:8080;
    root /var/www/phpmyadmin/;
    index index.php index.html index.htm index.nginx-debian.html;

    access_log /var/log/nginx/phpmyadmin_access.log;
    error_log /var/log/nginx/phpmyadmin_error.log;

    location / {

        autoindex on;
        autoindex_exact_size on;
    }

    location ~ ^/(doc|sql|setup)/ {
        deny all;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        include snippets/fastcgi-php.conf;
    }

    location ~ /\.ht {
        deny all;
    }
    }
   ```

3. Save the changes to the `phpmyadmin.conf` file and exit the Nano text editor.


This configuration sets up an Nginx server block for PhpMyAdmin, listening on port 8080. It specifies the root directory as `/var/www/phpmyadmin/` and includes various location blocks for handling PHP scripts, denying access to sensitive directories, and other necessary settings.

After saving the configuration, you need to restart the Nginx service for the changes to take effect:
```shell
sudo service nginx restart
```

Once Nginx restarts, you should be able to access the PhpMyAdmin web interface using the URL `http://your_server_ip_or_domain:8080`.

<br>
To host PhpMyAdmin using the subdomain strategy, you can configure an Nginx server block with the following configuration:

```
server {
  listen 80;
  listen [::]:80;
  server_name phpmyadmin.your_example.com;
  root /var/www/phpmyadmin/;
  index index.php index.html index.htm index.nginx-debian.html;

  access_log /var/log/nginx/phpmyadmin_access.log;
  error_log /var/log/nginx/phpmyadmin_error.log;

  location / {
    try_files $uri $uri/ /index.php;
  }

  location ~ ^/(doc|sql|setup)/ {
    deny all;
  }

  location ~ \.php$ {
    fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
    include snippets/fastcgi-php.conf;
  }

  location ~ /\.ht {
    deny all;
  }
}
```

Here's how you can use this configuration:

1. Open the Nginx configuration file for PhpMyAdmin using the Nano text editor:
   ```shell
   sudo nano /etc/nginx/conf.d/phpmyadmin.conf
   ```

2. Copy and paste the above configuration into the opened file.

3. Modify the `server_name` directive to match your desired subdomain. For example, replace `phpmyadmin.your_example.com` with `phpmyadmin.your_domain.com`.

4. Save the changes to the `phpmyadmin.conf` file and exit the Nano text editor.

After saving the configuration, you need to restart the Nginx service for the changes to take effect:
```shell
sudo service nginx restart
```

Once Nginx restarts, you should be able to access the PhpMyAdmin web interface using the subdomain URL `http://phpmyadmin.your_example.com`. Make sure to replace `your_example.com` with your actual domain.

## Automate daily backups of your MySQL database  

To install `mysqldump` and `zip` on your system for performing backups, you can use the following commands:

1. Install `mysqldump`:
   ```shell
   sudo apt install mysql-client
   ```

   This command installs the MySQL client package, which includes the `mysqldump` utility.

2. Install `zip`:
   ```shell
   sudo apt install zip
   ```

   This command installs the `zip` utility, which is used to compress the backup files.

By running these commands with superuser privileges (`sudo`), you can install `mysqldump` and `zip` on your system. Once installed, you can proceed with creating backups using `mysqldump` and compressing them with `zip`.

<br>
To create a backup directory named "backup" in the root of your server, you can use the following command:

```shell
sudo mkdir /backup
```

Running this command with superuser privileges (`sudo`) will create the "backup" directory directly under the root ("/") directory. This directory can be used to store your MySQL database backups and other relevant backup files.

<br>

To create a shell script file for performing backups, open the file using the `nano` text editor with the following command:

```shell
sudo nano /backup/script.sh
```

This command opens the `script.sh` file (located in the `/backup` directory) using the `nano` text editor.
<br>

Inside the `script.sh` file, you can include the backup code. Here's an example script that uses `mysqldump` to backup a MySQL database and then compresses the backup file using `zip`:

```shell
#!/bin/bash
now=$(date +%d%m%Y-%H:%M:%S)
filename=$1
backupfilename=$1-$now
mysqldump  --single-transaction   --no-tablespaces  -u [Database Username] -p[Database Password] [Database Name] > /backup/backup$backupfilename.sql
zip -r /backup/backup$backupfilename.zip /backup/backup$backupfilename.sql
rm /backup/backup$backupfilename.sql
```

Please replace `[Database Username]`, `[Database Password]`,  and `[Database Name]` with the appropriate values specific to your MySQL database.
<br>

Save the changes to the `script.sh` file and exit the `nano` editor.
<br>
Make the script executable: Set the executable permission on the script file to allow it to be executed. You can use the chmod command to achieve this. For example:

```shell
chmod +x /backup/script.sh
```
You can now use this shell script (`script.sh`) to perform backups by running it manually or scheduling it as a cron job.
<br>
Set up a cron job: Use the cron job scheduler to automate the execution of the backup script. The cron job can be configured to run at a specific time or interval. Open the crontab file for editing by running:

```shell
crontab -e
```

Add an entry to the crontab file: In the crontab file, add an entry that specifies the schedule and the path to the backup script. For example, to run the backup script every day at 2 AM, you can add the following entry:

```
0 2 * * * /path/to/backup_script.sh
```

Save and exit the crontab file: After adding the cron job entry, save the file and exit the text editor.

## Docker-based installation and management of MySQL

There are several advantages to using Docker for the installation and management of MySQL:

1. **Isolation and Portability**: Docker allows you to encapsulate the MySQL database and its dependencies within a container. This isolation ensures that the MySQL installation and configuration do not interfere with other applications or the host system. It also provides portability, allowing you to easily move the containerized MySQL instance across different environments, ensuring consistency and eliminating potential compatibility issues.

2. **Easy Deployment**: With Docker, deploying a MySQL instance becomes simpler and more efficient. Docker images contain all the necessary components and configurations, making it easy to spin up new MySQL containers quickly. This saves time and effort compared to manual installation, especially in scenarios where you need to deploy multiple instances or recreate environments for testing or development purposes.

3. **Version Control and Reproducibility**: Docker enables you to specify the exact version of MySQL and other dependencies through Docker images and Dockerfiles. This ensures that the same MySQL version can be consistently used across different environments, including development, staging, and production. It enhances version control and allows for easy replication of the exact environment configuration.

4. **Scalability and Resource Optimization**: Docker provides scalability options by allowing you to run multiple MySQL containers simultaneously. With Docker's container orchestration tools like Docker Swarm or Kubernetes, you can scale the number of MySQL containers up or down based on demand. This flexibility enables efficient resource utilization, as you can allocate resources dynamically to meet specific workload requirements.

5. **Simplified Maintenance and Updates**: Docker simplifies the process of maintaining and updating MySQL instances. With Docker, you can easily stop, start, or restart containers, perform upgrades, and apply patches without affecting other components or the host system. It provides a clean separation between the containerized MySQL instance and the underlying infrastructure, making maintenance tasks more manageable and reducing the risk of system-wide disruptions.

6. **Reproducible Development Environments**: Docker-based MySQL installations ensure consistency and reproducibility in development environments. Developers can share the same MySQL container configuration across the team, eliminating compatibility issues and reducing the time spent on environment setup. This promotes collaboration and streamlines the development process.

Overall, Docker-based installation and management of MySQL offer advantages such as isolation, portability, ease of deployment, version control, scalability, simplified maintenance, and reproducible development environments. These benefits make Docker a popular choice for running and managing MySQL instances in various scenarios, from local development environments to production deployments.

<br>

### Step 1 - Install Docker

To ensure a smooth installation process, certain dependencies are required. You can install these dependencies by running the following command:

```shell
sudo apt install apt-transport-https curl gnupg-agent ca-certificates software-properties-common -y
```
Once the dependencies are installed, you can proceed with the installation of Docker. We will install the Docker Community Edition (Docker CE), an open-source and free version of Docker.
<br>
To do so, we will add the GPGK key

```shell
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Once added, add the Docker repository as follows.


```shell
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
```
After adding the GPG key and repository, you can install Docker and its associated packages by running the following command:

```shell
sudo apt install docker-ce docker-ce-cli containerd.io -y
```

This command installs Docker along with all the necessary additional packages, libraries, and dependencies that are required for Docker and its associated packages to function properly.

After the command successfully completes, it is recommended to add the currently logged-in user to the `docker` group. This allows you to run Docker commands without needing to use `sudo` each time.

```shell
sudo usermod -aG docker $USER
newgrp docker
```

Install Docker Compose using the package manager:

```shell
sudo apt install docker-compose
```

To verify that Docker is installed, run the command:


```shell
docker version
```

```console
OUTPUT
Client: Docker Engine - Community
 Version:           24.0.4
 API version:       1.43
 Go version:        go1.20.5
 Git commit:        3713ee1
 Built:             Fri Jul  7 14:50:57 2023
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          24.0.4
  API version:      1.43 (minimum version 1.12)
  Go version:       go1.20.5
  Git commit:       4ffc614
  Built:            Fri Jul  7 14:50:57 2023
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.21
  GitCommit:        3dce8eb055cbb6872793272b4f20ed16117344f8
 runc:
  Version:          1.1.7
  GitCommit:        v1.1.7-0-g860f061
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```
To manage the Docker service on a Linux system, you can use commands to start, stop, restart, enable, and disable the Docker service. Here are some commonly used commands:

1. **Start Docker Service:**
   ```
   sudo systemctl start docker
   ```

2. **Stop Docker Service:**
   ```
   sudo systemctl stop docker
   ```

3. **Restart Docker Service:**
   ```
   sudo systemctl restart docker
   ```

4. **Enable Docker Service to Start on Boot:**
   ```
   sudo systemctl enable docker
   ```

5. **Disable Docker Service from Starting on Boot:**
   ```
   sudo systemctl disable docker
   ```

6. **Check Docker Service Status:**
   ```
   sudo systemctl status docker
   ```

7. **View Docker Service Logs:**
   ```
   sudo journalctl -u docker
   ```

These commands assume that you are using a Linux distribution with systemd as the init system. If you are using a different init system, the commands may vary. Make sure to adjust the commands according to your specific Linux distribution and init system.

By using these commands, you can easily start, stop, restart, enable, and disable the Docker service on your Linux system, allowing you to manage the Docker daemon effectively.

To start setting up your MySQL server, you can create an environment variable in a `.env` file. This file will store configuration variables for your application.

Here's an example of how you can create the `.env` file and set the environment variables for your MySQL server:

1. Open a text editor and create a new file named `.env`.

2. In the `.env` file, add the following lines:
   ```
   MYSQL_DB=ng_db
   MYSQL_ROOT_PASSWORD=password
   ```
   These lines define the environment variables `MYSQL_DB` with the value `ng_db` and `MYSQL_ROOT_PASSWORD` with the value `password`. Modify these values as needed for your specific setup.

3. Save the changes to the `.env` file.

By defining these environment variables in the `.env` file, you can easily reference and use them during the setup of your MySQL server or any other applications that require these configurations.

This Docker Compose file creates a MySQL server with PHPMyAdmin and a scheduled backup. Let's break down the file and explain each section:

```yaml
version: '3'
```
This line specifies the version of the Docker Compose file format being used. In this case, it's version 3.

```yaml
services:
  mysql:
    image: mysql:5.7
    container_name: mysql_server
    ports:
      - "3306:3306"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /opt/mysql/data:/var/lib/mysql
      - /opt/mysql/backup/latest.${MYSQL_DB}.sql.gz:/docker-entrypoint-initdb.d/database.sql.gz
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DB}
    restart: unless-stopped
```
This section defines the MySQL service. It pulls the `mysql:5.7` Docker image, sets the container name to `mysql_server`, maps port 3306 from the host to the container, and mounts volumes for time synchronization and data storage. The `${MYSQL_DB}` and `${MYSQL_ROOT_PASSWORD}` variables are used to set the MySQL database name and root password, respectively. The `restart: unless-stopped` option ensures that the MySQL service restarts automatically unless explicitly stopped.

```yaml
  phpmyadmin:
    image: phpmyadmin
    container_name: mysql_phpmyadmin
    restart: always
    expose:
      - "8001"
    ports:
      - "8001:80"
    environment:
      PMA_HOST: mysql_server
      PMA_PORT: 3306
      UPLOAD_LIMIT: 1000M
```
This section defines the PHPMyAdmin service. It uses the `phpmyadmin` Docker image, sets the container name to `mysql_phpmyadmin`, and ensures that the service restarts always. The service is exposed on port 8001 within the container, and the port is mapped to port 80 on the host. The environment variables `PMA_HOST` and `PMA_PORT` specify the MySQL server host and port to connect to, and `UPLOAD_LIMIT` sets the maximum upload limit for PHPMyAdmin.

```yaml
  mysql-cron-backup:
    image: fradelg/mysql-cron-backup
    container_name: mysql_backup
    depends_on:
      - mysql
    volumes:
      - /opt/mysql/backup:/backup
    environment:
      MYSQL_HOST: mysql_server
      MYSQL_USER: root
      MYSQL_PASS: ${MYSQL_ROOT_PASSWORD}
      MAX_BACKUPS: 240
      INIT_BACKUP: 0
      CRON_TIME: '* * * * *'
      GZIP_LEVEL: 9
    restart: unless-stopped
```
This section defines the MySQL backup service using the `fradelg/mysql-cron-backup` Docker image. It sets the container name to `mysql_backup` and depends on the `mysql` service. The volume mapping `/opt/mysql/backup:/backup` ensures that the backup files are stored on the host machine. Environment variables such as `MYSQL_HOST`, `MYSQL_USER`, and `MYSQL_PASS` specify the MySQL server details for the backup script. `MAX_BACKUPS` sets the maximum number of backups to retain, `INIT_BACKUP` specifies whether to create an initial backup when the container starts, `CRON_TIME` defines the schedule for backup (every minute in this example), and `GZIP_LEVEL` sets the compression level for backups.


This Docker Compose file allows you to run a MySQL server with PHPMyAdmin and schedule backups. It provides an integrated environment for managing databases and automating backup tasks.

This is the full docker-compose.yml file:

```yaml
version: '3'
services:
  mysql:
    image: mysql:5.7
    container_name: mysql_server
    ports:
      - "3306:3306"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /opt/mysql/data:/var/lib/mysql
      # If there is not scheme, restore the last created backup (if exists)
      - /opt/mysql/backup/latest.${MYSQL_DB}.sql.gz:/docker-entrypoint-initdb.d/database.sql.gz
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DB}
    restart: unless-stopped
  phpmyadmin:
    image: phpmyadmin
    container_name: mysql_phpmyadmin
    restart: always
    expose:
      - "8001"
    ports:
      - "8001:80"
    environment:
      PMA_HOST: mysql_server
      PMA_PORT: 3306
      UPLOAD_LIMIT: 1000M
  mysql-cron-backup:
    image: fradelg/mysql-cron-backup
    container_name: mysql_backup
    depends_on:
      - mysql
    volumes:
      - /opt/mysql/backup:/backup
    environment:
      MYSQL_HOST: mysql_server
      MYSQL_USER: root
      MYSQL_PASS: ${MYSQL_ROOT_PASSWORD}
      MAX_BACKUPS: 240
      INIT_BACKUP: 0
      # Every 6 hours
      CRON_TIME: '* * * * *'
      # Make it small
      GZIP_LEVEL: 9
    restart: unless-stopped

volumes:
  data:
```

To manage a Docker Compose file, you can use various Docker Compose commands to control the lifecycle of your services defined in the file. Here are some commonly used commands:

1. **Start Containers**:
   ```
   docker-compose up
   ```
   This command creates and starts the containers defined in the Docker Compose file. It also builds any required images if they don't already exist.

2. **Start Containers in Detached Mode**:
   ```
   docker-compose up -d
   ```
   This command starts the containers in detached mode, running them in the background. It allows you to continue using the terminal without being attached to the container logs.

3. **Stop Containers**:
   ```
   docker-compose down
   ```
   This command stops and removes the containers defined in the Docker Compose file. It also removes any networks and volumes associated with the containers, but it does not remove the images.

4. **View Container Logs**:
   ```
   docker-compose logs
   ```
   This command displays the logs of the running containers defined in the Docker Compose file. It shows the combined logs of all services.

5. **View Container Logs for a Specific Service**:
   ```
   docker-compose logs <service-name>
   ```
   This command displays the logs of a specific service defined in the Docker Compose file. Replace `<service-name>` with the actual name of the service.

6. **Scale Services**:
   ```
   docker-compose up --scale <service-name>=<num-instances>
   ```
   This command scales a service to the specified number of instances. Replace `<service-name>` with the name of the service you want to scale, and `<num-instances>` with the desired number of instances.

7. **Execute Command in a Running Service**:
   ```
   docker-compose exec <service-name> <command>
   ```
   This command allows you to execute a command in a running container of a specific service. Replace `<service-name>` with the name of the service, and `<command>` with the command you want to run.

These are just a few examples of Docker Compose commands to manage your services defined in the Docker Compose file. You can refer to the Docker documentation for more details and additional commands to suit your specific needs.


To access a shell within a running container by its name, you can use the `docker exec` command with the container's name. Here's the general syntax:

```shell
docker exec -it <container-name> <command>
```

Let's break down the command:

- `docker exec`: The command to execute a command within a running container.
- `-it`: These flags enable an interactive session with the container and allocate a pseudo-TTY.
- `<container-name>`: Replace this with the name of the container you want to access.
- `<command>`: Specify the command you want to execute within the container. Commonly used commands include `bash`, `sh`, or `sh -l` to start a shell session within the container.

Here's an example of accessing a shell within a running container named "my-container":

```shell
docker exec -it mysql_server  bash
```

This command will open a Bash shell within the "mysql_server" container, allowing you to interact with the container's filesystem and execute commands.

Make sure to replace `<container-name>` with the actual name of your container. If you're unsure about the container names, you can use the `docker ps` command to list all running containers and verify their names.


### MySQL Management

Congratulations on completing the MySQL management section of the course! You have gained valuable knowledge and practical experience in setting up, configuring, and managing a MySQL database. By now, you should be comfortable with tasks such as installation, securing the database, performing backups, and interacting with PHPMyAdmin.

Remember, mastering MySQL management is a continuous journey. There is always more to learn and explore in the world of databases. As you continue your learning path, consider diving deeper into advanced topics such as database optimization, replication, clustering, and high availability setups.

If you have any questions or need further guidance, don't hesitate to reach out to me. You can contact me on [LinkedIn](https://www.linkedin.com/in/mehdi-aitsaid/) or visit my [website](https://ait-said.ngcloud.ma/). I'm here to support you in your learning journey and help you overcome any challenges you may encounter.

Remember, practice is key to mastering MySQL management. Take the time to work on hands-on projects, experiment with different configurations, and explore real-world use cases. The more you practice, the more confident and proficient you will become.

Keep up the excellent work, and I look forward to seeing you succeed in your MySQL management endeavors. Happy learning!

Respectfully <br>
AIT SAID Mehdi <br>
Senior Software Architect, and Full-Stack Software Engineer

