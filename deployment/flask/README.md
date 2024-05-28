# Deploying a Flask App with Nginx and with/without Docker

![Deploying a Flask App with Nginx and with/without Docker](image.png)

Flask is a lightweight and versatile web framework for Python. It is designed with simplicity and flexibility in mind, making it an excellent choice for both beginners and experienced developers who want to create robust web applications quickly and efficiently. Flask follows a minimalist approach, providing only the essential components needed to build web applications, while allowing developers to extend its functionality through a wide array of extensions.


> Note: Before starting this lab, ensure you have completed the following prerequisites:
> - Setting up the client-server environment as described in the [Environment Setup](../../Prerequisites.md) section. Proper DNS configuration and hostname setup are essential for the exercises in this lab.
> - Installing Nginx as outlined in the [Nginx Installation Guide](../../nginx/README.md).
> - Alternatively, if you prefer to use Docker, make sure Docker is installed and running on your server. You can refer to the [Docker Installation Guide](../../../docker/README.md) for detailed instructions.

In this lab, we will learn how to deploy a Flask App  on an Ubuntu server using Nginx. We will cover how to transfer your project to the server using SCP or a GitHub repository.For this lab, we will use the subdomain flask.server.io and a demo project named [dome.zip](demo.zip) is attached in the same directory as this lab for your convenience.


## Deploying a Flask App Using Nginx without Docker

### Step 1: Create the Website Folder

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
#### Create the Website Folder:
1. Create a new directory for your project:

```bash
sudo mkdir -p /workdir/flask.server.io

sudo chown -R $USER:$USER /workdir/flask.server.io
sudo chmod -R 755 /workdir/flask.server.io

cd /workdir/flask.server.io
```

### Step 2: Transfer the flask App Files
You can transfer your flask app files to the server using SCP or by cloning a GitHub repository.

#### Using SCP:

1. Open a terminal on your local machine.

2. Use the scp command to transfer the files. Replace <local-path> with the path to your project directory and <user> and <server-ip> with your server's username and IP address:

```bash
scp -r <local-path> <user>@<server-ip>:/workdir/flask.server.io
```
Example:

```bash
scp -r ./demo.zip serverio@192.168.1.10:/workdir/flask.server.io
```

#### Using GitHub:

1. Navigate to the web root directory:

```bash
cd /workdir/flask.server.io
```
2. Clone your repository:
```bash
git clone https://github.com/yourusername/flask-website.git flask-website
```



### Step 3: Install Python

```bash
sudo apt install python3-pip python3-dev
```

Now, Install the virtual environment manager

```bash
sudo apt install python3.10-venv
```

### Step 4: Setup your Flask App

1. Create a virtual environment for your Flask app and activate it:
```bash
cd /workdir/flask.server.io
python3 -m venv env
source env/bin/activate
```

2. Extract the project files
```bash
sudo unzip demo.zip
sudo rm demo.zip
```

3. Install Flask and Gunicorn inside the virtual environment:

```bash
pip install flask gunicorn
```
### Step 5: Creating the systemd service 
Next, create the systemd service unit file. Creating a systemd unit file will allow Ubuntu’s init system to automatically start Gunicorn and serve the Flask application whenever the server boots.

Create a unit file ending in .service within the /etc/systemd/system directory to begin:

```bash
sudo nano /etc/systemd/system/flask.server.io.service
```

```bash
[Unit]
Description=Gunicorn instance to serve flask.server.io
After=network.target

[Service]
User=serverio
Group=www-data
WorkingDirectory=/workdir/flask.server.io
Environment="PATH=/workdir/flask.server.io/env/bin"
ExecStart=/workdir/flask.server.io/env/bin/gunicorn --bind 0.0.0.0:5000 app:app

[Install]
WantedBy=multi-user.target
```

This service file defines how to run a Flask application using Gunicorn as a daemon (background service) on a systemd-based Linux system. Let's break down each section:

[Unit]

- Description: This line provides a human-readable description of the service. Here, it indicates it's a Gunicorn instance serving a Flask application named flask.server.io.
- After=network.target: This line specifies that this service should be started after the network.target is reached. This ensures that network interfaces are up and running before the service attempts to listen for connections.

[Service]
- User: This line defines the user under which the service will run. Here, it's set to serverio. This user should have the necessary permissions to access the application files and libraries.
- Group: This line specifies the group to which the service process will belong. Here, it's set to www-data. This is commonly used for web services as it might provide access to resources needed by the application.
- WorkingDirectory: This line defines the working directory where the service will be executed. Here, it's set to /workdir/flask.server.io, indicating the directory containing the Flask application code.
- Environment: This line sets an environment variable named PATH. The value specifies the path to the directory containing the Gunicorn executable within the virtual environment (/workdir/flask.server.io/env/bin). This ensures Gunicorn can be found when the service is started.
- ExecStart: This line defines the command to be executed to start the service. Here, it uses /workdir/flask.server.io/env/bin/gunicorn, which is the Gunicorn executable located in the virtual environment. The command arguments are:
  - --bind 0.0.0.0:5000: This tells Gunicorn to listen on all network interfaces (0.0.0.0) on port 5000.
  - app:app: This defines the application object (usually defined in your Flask application file) to be served by Gunicorn.


[Install]
- WantedBy=multi-user.target: This line specifies that the service should be automatically started when the system reaches the multi-user.target. This target indicates the system is ready for multi-user login and is a common place for services to be launched.



With that, your systemd service file is complete. Save and close it now.

You can now start the Gunicorn service that you created and enable it so that it starts at boot:

```bash
sudo systemctl start flask.server.io
```


### Step 6: Configure Nginx

![Reverse Proxy](image-2.png)

1. Create a new Nginx configuration file for your Flask App :

```bash
sudo nano /etc/nginx/sites-available/flask.server.io
```

2. Add the following configuration to the file:

```nginx
server {
    listen 80;
    server_name flask.server.io www.flask.server.io;

    location / {
        proxy_pass http://localhost:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Define access log and error log locations
    access_log /var/log/nginx/flask-app-proxy.access.log;
    error_log /var/log/nginx/flask-app-proxy.error.log;
}
```

### Nginx Configuration for flask Boot Application

This Nginx configuration file sets up a server to handle HTTP requests for a flask  application hosted under the `flask.server.io` domain. The Nginx server acts as a reverse proxy, forwarding incoming requests to the flask application running on localhost at port 3000.

**Configuration Breakdown**

- **Server Block:**
    - `listen 80;`
      - The server listens on port 80, which is the default port for HTTP traffic.
    - `server_name flask.server.io www.flask.server.io;`
      - The server responds to requests for the domain names `flask.server.io` and `www.flask.server.io`.

- **Location Block:**
    - `location / { ... }`
      - Defines how requests to the root URL and its subpaths should be handled.

- **Proxy Settings:**
    - `proxy_pass http://localhost:5000;`
      - Forwards all incoming requests to the flask application running on localhost at port 5000.
    - `proxy_set_header Host $host;`
      - Sets the `Host` header in the forwarded request to the value of the original request's host header.
    - `proxy_set_header X-Real-IP $remote_addr;`
      - Sets the `X-Real-IP` header to the client’s IP address.
    - `proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;`
      - Sets the `X-Forwarded-For` header to include the client’s IP address. This header helps in tracking the original client's IP address when requests are forwarded through proxies.
    - `proxy_set_header X-Forwarded-Proto $scheme;`
      - Sets the `X-Forwarded-Proto` header to the scheme (HTTP or HTTPS) used in the original request. This is useful for applications that need to know the protocol used by the client.

- **Log Configuration:**
    - `access_log /var/log/nginx/flask-app-proxy.access.log;`
      - Specifies the location of the access log file, which records details about each request handled by the server.
    - `error_log /var/log/nginx/flask-app-proxy.error.log;`
      - Specifies the location of the error log file, which records any errors encountered while processing requests.


```bash
sudo ln -s /etc/nginx/sites-available/flask.server.io /etc/nginx/sites-enabled/
```

4. Test the Nginx configuration for syntax errors:

```bash
sudo nginx -t
```
5. Reload Nginx to apply the changes:

```bash
sudo systemctl reload nginx
```

### Step 6: Verify the Deployment
Open a web browser on your client machine.
Navigate to http://flask.server.io to see your deployed flask App.

![alt text](image-1.png)

## Deploying a Flask App Using Nginx with Docker

### Step 1: Make Docker image
1. Inside the working  directory (`/workdir/flask.server.io`), create a `Dockerfile` with the following content:


```Dockerfile
FROM python:3.10
EXPOSE 5000
WORKDIR /app
#COPY requirements.txt .  : if you have requirements.txt you can use this config
#RUN pip install -r requirements.txt
RUN pip install flask gunicorn
COPY . .
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

## Step 2: Build and Run the Docker Container

1. Build the Docker image:
```bash
sudo docker build -t flask.server.io .
```

2. Run the Docker container:
```bash
sudo docker run -d -p 5000:5000 --name flask.server.io-container flask.server.io
```
- This command maps port 5000 on your server to port 5000 in the container.

### Step 3: Configure Nginx 

In this case we will use the same configuration for the section without docker

1. Create a new Nginx configuration file for your Flask App :

```bash
sudo nano /etc/nginx/sites-available/flask.server.io
```

2. Add the following configuration to the file:

```nginx
server {
    listen 80;
    server_name flask.server.io www.flask.server.io;

    location / {
        proxy_pass http://localhost:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Define access log and error log locations
    access_log /var/log/nginx/flask-app-proxy.access.log;
    error_log /var/log/nginx/flask-app-proxy.error.log;
}
```

```bash
sudo ln -s /etc/nginx/sites-available/flask.server.io /etc/nginx/sites-enabled/
```

4. Test the Nginx configuration for syntax errors:

```bash
sudo nginx -t
```
5. Reload Nginx to apply the changes:

```bash
sudo systemctl reload nginx
```

### Step 6: Verify the Deployment
Open a web browser on your client machine.
Navigate to http://flask.server.io to see your deployed flask App.