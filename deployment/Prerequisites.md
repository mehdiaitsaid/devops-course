# Prerequisites
Before we begin configuring a web server, there are a few prerequisites to ensure a smooth learning experience. You will need two machines: one to act as the server and the other as the client. The client machine can be a Windows machine. Additionally, we need to configure DNS settings. If a DNS server is not available, we can use the hosts file to achieve the same result.

## Requirements
1. Server Machine:
    - This machine will host the Nginx server.
    - Ensure you have administrative access to make configuration changes.

2. Client Machine:
    - This machine will be used to access the server.
    - A Windows machine is suitable for this purpose.

## DNS Configuration
To resolve domain names to the server’s IP address, we can configure DNS. If you do not have access to a DNS server, you can use the hosts file on both the client and the server machines. This is an old-school but effective solution for local network setups.


> Note: For this lab, we will use server.io as the hostname for the server. You can name it anything else as you prefer.


### Configuring the Hosts File

#### 1. Change the Hostname of Your Server:

You need to change the hostname of your server to server.io or your chosen hostname. This can be done using the hostnamectl command.

```
$ sudo hostnamectl set-hostname server.io
```

Ensure that your /etc/hosts file has an entry 127.0.0.1 server.io

#### 2. Determine Your Server's IP Address:

Identify the IP address of your server machine. For this example, we will use 192.168.1.10.

#### 3. Edit the Hosts File:

On both the client and server machines, you need to modify the hosts file to map domain names to the server’s IP address.

On Windows:

- Open Notepad as an administrator.

- Open the file C:\Windows\System32\drivers\etc\hosts.

- Add the following lines to the file:

```
192.168.1.10	server.io
192.168.1.10	static.server.io
192.168.1.10	laravel.server.io
192.168.1.10	spring.server.io
192.168.1.10	flask.server.io
192.168.1.10	express.server.io
192.168.1.10	phpmyadmin.server.io
192.168.1.10	php.server.io
192.168.1.10	java.server.io
```

On Linux/Mac:

- Open a terminal.
- Edit the hosts file using a text editor with superuser privileges, such as sudo nano /etc/hosts.
- Add the same lines as above.