# Mini-Project: Web Deployment Automation Tool

## Objective
The goal of this project is to create a **Web Deployment Automation Tool** that validates your understanding of **Web Deployment** concepts. The tool will streamline web application deployments using Python and Nginx on an Ubuntu server.

## Project Description
You are tasked with developing a Python-based tool that automates the deployment of web applications, manages server resources, supervises Nginx, and installs necessary dependencies.

---

## Functional Requirements
Your project must include the following functionalities:

### 1. System Health Monitoring
- Display essential server statistics:
  - **CPU**: Number of cores, usage per core, and total usage.
  - **Memory**: Total, used, and available RAM.
  - **Disk**: Partition usage and available space.
  - **Network**: Interface details, total data sent, and received.
- Use the `psutil` library for system monitoring.
- Format the output with color-coded sections for better readability.

### 2. Nginx Supervision
- Check if **Nginx** is installed.
  - Prompt the user to install Nginx if it's not found.
- Provide the following options for managing Nginx:
  - Start, stop, restart, reload, enable, or disable Nginx.
  - Check Nginx configuration syntax using `nginx -t`.

### 3. Manage Available Sites
- List all available sites from `/etc/nginx/sites-available/`.
- Add a new site:
  - Clone a GitHub repository into `/services/<domain-name>`.
  - Create a directory for the site if it doesn’t exist.
  - Generate an appropriate Nginx configuration file based on the site type:
    - **HTML**: Serves static files.
    - **PHP (Laravel)**: Configures PHP 8.3, Composer, and Laravel setup.
    - **Java (Spring Boot)**: Configures JAR deployment with proxy and systemd service.
    - **Python (Flask)**: Configures Flask with Gunicorn and systemd service.
    - **Node.js (Express)**: Manages Node.js apps with PM2 and proxy.

### 4. Manage Enabled Sites
- List all enabled sites from `/etc/nginx/sites-enabled/`.
- Provide the following management options:
  - Enable a site by creating a symbolic link from `/etc/nginx/sites-available/`.
  - Disable a site by removing the symbolic link.
  - Delete site configurations entirely.

### 5. Check and Install Dependencies
- Ensure the following runtimes are installed:
  - **PHP 8.3 + Composer**
  - **Java 17**
  - **Node.js 22**
  - **Flask** (Python web framework)
- Prompt users to install missing dependencies interactively.
- Use a progress indicator for installations.

---

## Deliverables
1. **Project Code**:
   - Modular Python code organized into separate files for each functionality.
   - Ensure the code is clean, readable, and includes comments for clarity.

2. **README.md**:
   - Include a detailed project description.
   - Provide setup instructions and usage examples.

3. **Video**:
   - Attach terminal video showcasing the tool's functionalities.

4. **Submission**:
   - Compress the entire project folder into a `.zip` file.
   - Name the file as `<yourname>_web_deployment_tool.zip`.
   - Send the zipped file via email with the subject:
     `DevOps and Cloud Mini-Project Submission - <Your Name>`.


---

## Example Video
Here’s an example video demonstrating the terminal output and tool functionalities:

[![Example Video](image.png)]()

