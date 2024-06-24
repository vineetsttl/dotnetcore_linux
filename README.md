# .NET Core 6 Application Deployment on a Linux Server Using MobaXterm

This guide provides step-by-step instructions to host and deploy a .NET Core 6 application on a Linux server using MobaXterm from a Windows machine.

Here is a properly formatted step-by-step guide to host and deploy a .NET Core 6 application on a Linux server using MobaXterm from Windows:

---

## Prerequisites

1. **Install MobaXterm on your Windows machine:**
   - Download it from [MobaXterm's official website](https://mobaxterm.mobatek.net/).

2. **Have a Linux server available:**
   - Ensure you have SSH access to the server.

## Step-by-Step Guide

### 1. Develop and Publish Your .NET Core Application

1. **Publish the application:**
   ```sh
   dotnet publish --configuration Release
   ```
   This will create a publish folder with the necessary files to run your application. By default, this folder is located at `bin/Release/net6.0/publish/`.

### 2. Connect to Your Linux Server Using MobaXterm

1. **Open MobaXterm.**

2. **Start a new SSH session:**
   - Click on the "Session" icon.
   - Select "SSH".
   - Fill in the "Remote host" field with your server's IP address or hostname.
   - Click "OK".

3. **Login with your credentials:**
   - Provide your username and password or use an SSH key if configured.

### 3. Prepare the Linux Server

1. **Update package lists:**
   ```sh
   sudo apt update
   ```

2. **Install .NET Core Runtime on the Linux Server:**

   1. **Update package lists:**
      ```sh
      sudo apt update
      ```

   2. **Install the required packages:**
      ```sh
      sudo apt install wget apt-transport-https
      ```

   3. **Add the Microsoft package signing key and repository:**
      ```sh
      wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
      sudo dpkg -i packages-microsoft-prod.deb
      ```

   4. **Install the .NET 6.0 runtime:**
      ```sh
      sudo apt update
      sudo apt install dotnet-runtime-6.0
      ```

### 4. Configure Reverse Proxy (Optional but Recommended)

To make your application accessible via standard HTTP/HTTPS ports (80/443), configure a reverse proxy using Nginx:

1. **Install Nginx:**
   ```sh
   sudo apt install nginx
   ```

2. **Configure Nginx:**
   - Open the Nginx configuration file for editing:
     ```sh
     sudo nano /etc/nginx/sites-available/default
     ```

   - Modify the file to include the following configuration:
     ```nginx
     server {
         listen 80;
         server_name your_domain_or_ip;

         location / {
             proxy_pass http://localhost:5000;
             proxy_http_version 1.1;
             proxy_set_header Upgrade $http_upgrade;
             proxy_set_header Connection keep-alive;
             proxy_set_header Host $host;
             proxy_cache_bypass $http_upgrade;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
         }
     }
     ```

   - Save and close the file (Ctrl+X, then Y, then Enter).

3. **Restart Nginx:**
   ```sh
   sudo systemctl restart nginx
   ```

4. **Access your application:**
   - Navigate to `http://your_domain_or_ip` in your browser.

### 5. Run Your Application on the New Port

Start your .NET application on the new port:
```sh
dotnet MyWebApp.dll --urls "http://localhost:5001"
```

### 6. Configure Firewall (if necessary)

Ensure that port 5000 is open on your server to allow incoming connections.

1. **Check if UFW (Uncomplicated Firewall) is installed:**
   ```sh
   sudo ufw status
   ```

2. **If UFW is active, allow port 5000:**
   ```sh
   sudo ufw allow 5000
   ```

### 7. Check Nginx Status

1. **Check Nginx Status:**
   Run the following command to see the status of the nginx service:
   ```sh
   sudo systemctl status nginx.service
   ```

2. **Check Nginx Error Logs:**
   Run the following command to check the error logs:
   ```sh
   sudo journalctl -xeu nginx.service
   ```

### 8. Common Issues and Fixes

1. **Configuration Syntax Error:**
   If there is a syntax error in the Nginx configuration file, you can check for syntax errors using:
   ```sh
   sudo nginx -t
   ```

2. **Port Conflict:**
   Make sure that no other service is using port 80 or 443. You can check which processes are using these ports with:
   ```sh
   sudo lsof -i :80
   sudo lsof -i :443
   ```

### 9. Optional: Set Up Nginx as a Reverse Proxy (Recommended for Production)

To make your application accessible via standard HTTP/HTTPS ports (80/443), configure a reverse proxy using Nginx.

1. **Install Nginx:**
   ```sh
   sudo apt install nginx
   ```

2. **Configure Nginx:**
   - Open the Nginx configuration file for editing:
     ```sh
     sudo nano /etc/nginx/sites-available/default
     ```

   - Modify the file to include the following configuration:
     ```nginx
     server {
         listen 80;
         server_name _;

         location / {
             proxy_pass http://localhost:5000;
             proxy_http_version 1.1;
             proxy_set_header Upgrade $http_upgrade;
             proxy_set_header Connection keep-alive;
             proxy_set_header Host $host;
             proxy_cache_bypass $http_upgrade;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
         }
     }
     ```

   - Save and close the file (Ctrl+X, then Y, then Enter).

3. **Test the Nginx configuration:**
   ```sh
   sudo nginx -t
   ```

4. **Restart Nginx:**
   ```sh
   sudo systemctl restart nginx
   ```

5. **Access your application:**
   - Navigate to `http://192.168.21.163` in your browser.

### 10. Fixing Repository Issues for Ubuntu 22.04 (Jammy)

The error message indicates that the repository for Nginx you are trying to use is not available for the Ubuntu version "jammy" (22.04). Instead of using a PPA (Personal Package Archive) for Nginx, we can use the official Ubuntu repositories or the official Nginx repositories.

#### Step-by-Step Guide to Install Nginx Correctly

1. **Remove the problematic PPA:**
   ```sh
   sudo add-apt-repository --remove ppa:nginx/stable
   ```

2. **Add the official Nginx repository:**
   ```sh
   sudo apt update
   sudo apt install nginx
   ```

3. **Verify Nginx installation:**
   ```sh
   nginx -v
   ```

This should install Nginx successfully without errors.

---

By following these steps, you will have your .NET Core 6 application running on a Linux server and accessible from your browser.