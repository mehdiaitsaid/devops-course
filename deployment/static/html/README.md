# Deploying a Static Website with Nginx and with/without Docker

![Deploying a Static Website with Nginx with/without Docker](image.png)

A static website is a collection of web pages with fixed content. Each page is coded in HTML and displays the same information to every visitor. Static websites are ideal for showcasing content that doesn't change frequently, such as personal blogs, portfolios, and informational sites. They are easy to create, secure, and can be hosted on any web server, making them a popular choice for many web projects. In this lab, we will learn how to deploy a static website using Nginx, a powerful and efficient web server.


> Note: Before starting this lab, ensure you have completed the following prerequisites:
> - Setting up the client-server environment as described in the [Environment Setup](../../Prerequisites.md) section. Proper DNS configuration and hostname setup are essential for the exercises in this lab.
> - Installing Nginx as outlined in the [Nginx Installation Guide](../../nginx/README.md).
> - Alternatively, if you prefer to use Docker, make sure Docker is installed and running on your server. You can refer to the [Docker Installation Guide](../../../docker/README.md) for detailed instructions.

In this lab, we will learn how to deploy a static website on an Ubuntu server using Nginx. We will cover how to transfer your project to the server using SCP or a GitHub repository.For this lab, we will use the subdomain static.server.io and a demo project named [dome.zip](demo.zip) is attached in the same directory as this lab for your convenience.

## Deploying a Static Website Using Nginx without Docker

### Step 1: Connect to the Server and Create the Website Folder
#### Connect to the Server:

1. Open a terminal on your local machine.

2. Use SSH to connect to your Ubuntu server. Replace <user> and <server-ip> with your server’s username and IP address:

```bash
ssh <user>@<server-ip>
```

Example:

```bash
ssh serverio@192.168.1.10
```
2. Create the Website Folder:

Once connected to the server, create the directory structure for your static website. This structure will include a specific folder for your domain static.server.io. Run the following commands:

```bash
sudo mkdir -p /var/www/static.server.io/html
```

3. Set Permissions:

Next, assign ownership of the directory with the $USER environment variable:

```bash
sudo chown -R $USER:$USER /var/www/static.server.io/html
```

The permissions of your web roots should be correct if you haven’t modified your umask value, which sets default file permissions. To ensure that your permissions are correct and allow the owner to read, write, and execute the files while granting only read and execute permissions to groups and others, you can input the following command:

```bash
sudo chmod -R 755 /var/www/static.server.io/html
```

### Step 2: Transfer the Static Website Files
You can transfer your static website files to the server using SCP or by cloning a GitHub repository.

#### Using SCP:

1. Open a terminal on your local machine.

2. Use the scp command to transfer the files. Replace <local-path> with the path to your project directory and <user> and <server-ip> with your server's username and IP address:

```bash
scp -r <local-path> <user>@<server-ip>:/var/www/html/static-site
```
Example:

```bash
scp -r ./demo.zip serverio@192.168.1.10:/var/www/static.server.io/html
```

3. Connect to server and unzip the project files:

```bash
sudo apt install unzip
cd /var/www/static.server.io/html
unzip demo.zip
```

#### Using GitHub:

1. Navigate to the web root directory:

```bash
cd /var/www/static.server.io
```
2. Clone your repository:
```bash
git clone https://github.com/yourusername/static-website.git static-website
```

### Step 3: Configure Nginx
1. Create a new Nginx configuration file for your static site:

```bash
sudo nano /etc/nginx/sites-available/static.server.io
```

2. Add the following configuration to the file:

```nginx
server {
    listen 80;
    server_name static.server.io www.static.server.io;

    # Define the root directory for your static files
    root /var/www/static.server.io/html;

    # Configure access to static files
    location / {
        try_files $uri $uri/ =404;
    }

    # Additional configuration options can be added here

    # Define access log and error log locations
    access_log /var/log/nginx/static.server.io.access.log;
    error_log /var/log/nginx/static.server.io.error.log;

    # Add any other server-specific configurations here

}
```

**Explanation of the Nginx Server Configuration**

This Nginx configuration sets up a basic HTTP server for the domain `static.server.io` and its `www` subdomain. It serves static files from `/var/www/static.server.io/html` and logs access and error information to specified log files. The `try_files` directive ensures that only existing files or directories are served; otherwise, a `404` error is returned.

- **Server Block:**
   - The `server` block defines the configuration for handling requests to your server.

- **Listening Port:**
   - `listen 80;`
   - Specifies that the server will listen on port 80, the default port for HTTP traffic.

- **Server Name:**
   - `server_name static.server.io www.static.server.io;`
   - Defines the domain names that this server block should respond to. Here, it will respond to requests for `static.server.io` and `www.static.server.io`.

- **Root Directory:**
   - `root /var/www/static.server.io/html;`
   - Specifies the root directory where the static files for the website are located. Nginx will serve files from this directory.

- **Location Block:**
   - `location / { try_files $uri $uri/ =404; }`
   - The `location /` block defines how to handle requests for the root URL and its subpaths.
   - The `try_files` directive checks for the existence of the requested file (`$uri`) or directory (`$uri/`). If neither is found, it returns a `404 Not Found` error.

- **Access Log:**
   - `access_log /var/log/nginx/example.com.access.log;`
   - Defines the location of the access log file, where Nginx will log details of every request processed by this server block.

- **Error Log:**
   - `error_log /var/log/nginx/example.com.error.log;`
   - Specifies the location of the error log file, where Nginx will log errors encountered while processing requests.

- **Additional Configuration:**
   - Comments `# Additional configuration options can be added here` and `# Add any other server-specific configurations here` are placeholders for any extra configurations you might want to include, such as security headers, gzip compression, etc.

3. Enable the configuration by creating a symbolic link to the sites-enabled directory:

```bash
sudo ln -s /etc/nginx/sites-available/static.server.io /etc/nginx/sites-enabled/
```

4. Test the Nginx configuration for syntax errors:

```bash
sudo nginx -t
```
5. Reload Nginx to apply the changes:

```bash
sudo systemctl reload nginx
```

### Step 4: Verify the Deployment
Open a web browser on your client machine.
Navigate to http://static.server.io to see your deployed static website.

![alt text](image-1.png)


## Deploying a Static Website Using Nginx with Docker

### Step 1: Create a Dockerfile for Your Static Website

1. Create a new directory for your project:

```bash
sudo mkdir -p /workdir/static.server.io

sudo chown -R $USER:$USER /workdir/static.server.io
sudo chmod -R 755 /workdir/static.server.io

cd /workdir/static.server.io
```

2. Inside this directory, create a `Dockerfile` with the following content:
```Dockerfile
# Use the official Nginx image from the Docker Hub
FROM nginx:alpine

# Copy static website files to the Nginx HTML directory
COPY . /usr/share/nginx/html

# Expose port 80
EXPOSE 80
```

3. Add your static website files (e.g., `index.html`, `styles.css`, etc.) to this directory using SCP or GitHub as explained in the previous section

## Step 2: Build and Run the Docker Container

1. Build the Docker image:
```bash
sudo docker build -t static.server.io .
```

2. Run the Docker container:
```bash
sudo docker run -d -p 8080:80 --name static.server.io-container static.server.io
```
- This command maps port 8080 on your server to port 80 in the container.

2. Show the list if the container:
```bash
sudo docker container ls
```

```bash
Output
...
483baed20cc4   static.server.io   "/docker-entrypoint.…"   18 seconds ago   Up 17 seconds   0.0.0.0:8080->80/tcp, :::8080->80/tcp   static.server.io-container
...
```

## Step 3: Configure Nginx as a Reverse Proxy

1. On your server, edit the Nginx configuration file to set up a reverse proxy. Open the configuration file:
```bash
sudo nano /etc/nginx/sites-available/static-site-proxy
```

2. Add the following configuration:
```nginx
server {
    listen 80;
    server_name static.server.io www.static.server.io;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Define access log and error log locations
    access_log /var/log/nginx/static-site-proxy.access.log;
    error_log /var/log/nginx/static-site-proxy.error.log;
}
```

**Explanation of the Nginx Server Configuration**

This Nginx configuration sets up a server to handle HTTP requests and proxy them to a Docker container running on port 8080. Here’s a detailed breakdown of the configuration:

- **Server Block:**
   - The `server` block defines the configuration for handling requests to your server.

- **Listening Port:**
   - `listen 80;`
   - Specifies that the server will listen on port 80, the default port for HTTP traffic.

- **Server Name:**
   - `server_name static.server.io www.static.server.io;`
   - Defines the domain names that this server block should respond to. Here, it will respond to requests for `static.server.io` and `www.static.server.io`.

- **Location Block:**
   - `location / { ... }`
   - The `location /` block defines how to handle requests for the root URL and its subpaths.

- **Proxy Pass:**
   - `proxy_pass http://localhost:8080;`
   - This directive passes requests from Nginx to the backend server running on `http://localhost:8080`. In this case, it's the Docker container serving the static website.

- **Proxy Headers:**
   - `proxy_set_header Host $host;`
     - Sets the `Host` header in the proxied request to the original host requested by the client.
   - `proxy_set_header X-Real-IP $remote_addr;`
     - Sets the `X-Real-IP` header in the proxied request to the IP address of the client making the request.
   - `proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;`
     - Adds the client’s IP address to the `X-Forwarded-For` header, which is a standard header used for identifying the originating IP address of a client connecting to a web server through an HTTP proxy or load balancer.
   - `proxy_set_header X-Forwarded-Proto $scheme;`
     - Sets the `X-Forwarded-Proto` header to the scheme (HTTP or HTTPS) used by the client to connect to the server.

- **Access Log:**
   - `access_log /var/log/nginx/static-site-proxy.access.log;`
   - Defines the location of the access log file, where Nginx will log details of every request processed by this server block.

- **Error Log:**
   - `error_log /var/log/nginx/static-site-proxy.error.log;`
   - Specifies the location of the error log file, where Nginx will log errors encountered while processing requests.




3. Enable the configuration by creating a symbolic link to the `sites-enabled` directory:
```bash
sudo ln -s /etc/nginx/sites-available/static-site-proxy /etc/nginx/sites-enabled/
```

4. Test the Nginx configuration for syntax errors:
```bash
sudo nginx -t
```

5. Reload Nginx to apply the changes:
```bash
sudo systemctl reload nginx
```

## Step 4: Verify the Deployment

1. Open a web browser on your client machine.
2. Navigate to `http://static.server.io` to see your deployed static website served through Nginx and Docker.

![alt text](image-2.png)