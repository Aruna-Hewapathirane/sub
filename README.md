# How to Install Nginx on a VPS

To install Nginx on a VPS, connect to your server via SSH, install the `nginx` package using your Linux distribution's package manager, and open ports `80` and `443` in your firewall.

## 1. Connect to Your VPS

Log into your server as `root` or a user with `sudo` privileges:

```bash
ssh username@your_server_ip
```

## 2. Install Nginx

Run the commands that match your VPS operating system.

### Ubuntu / Debian

```bash
sudo apt update
sudo apt install nginx -y
```

### CentOS / Rocky Linux / AlmaLinux / RHEL

```bash
sudo dnf install nginx -y
```

## 3. Start and Enable the Service

Start Nginx immediately and configure it to launch automatically whenever your VPS reboots:

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

## 4. Adjust Your Firewall

Allow standard HTTP (`80`) and HTTPS (`443`) web traffic through your firewall.

### Ubuntu / Debian (UFW)

```bash
sudo ufw allow 'Nginx Full'
```

### Rocky / AlmaLinux / RHEL (firewalld)

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

> **Note:** If your VPS provider (such as AWS, DigitalOcean, or Oracle Cloud) uses an external network firewall or Security Group, ensure inbound rules for TCP ports `80` and `443` are also enabled in their dashboard.

## 5. Verify the Installation

1. **Check service status:** Run `sudo systemctl status nginx` and confirm it displays `active (running)`.
2. **Test in a browser:** Visit `http://your_server_ip` in any web browser. You should see the default **"Welcome to nginx!"** landing page.



# Setting Up `pascal.sahanaya.net` on a Namecheap VPS with Nginx

Here are the exact, ready-to-use steps to set up `pascal.sahanaya.net` on your Namecheap VPS with Nginx.

## Step 1. Update Namecheap DNS

1. Log in to Namecheap, go to **Domain List**, click **Manage** next to `sahanaya.net`, and select **Advanced DNS**.
2. Under **Host Records**, click **Add New Record**.
3. Set **Type** to `A Record`, **Host** to `pascal`, and **Value** to your VPS public IP address, then save.

## Step 2. Create the Web Folder and Nginx Server Block

Connect to your VPS via SSH and run the following commands. *(Skip the first three commands if you are proxying to an app port instead of serving static files.)*

```bash
# 1. Create the web directory (for static HTML/CSS/JS)
sudo mkdir -p /var/www/pascal.sahanaya.net/html
sudo chown -R USER:USER /var/www/pascal.sahanaya.net/html
echo "<h1>pascal.sahanaya.net is live</h1>" | sudo tee /var/www/pascal.sahanaya.net/html/index.html

# 2. Create the Nginx configuration file
sudo nano /etc/nginx/sites-available/pascal.sahanaya.net
```

Paste one of the configurations below into the editor:

### Option A: Static Website or Files

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name pascal.sahanaya.net;
    root /var/www/pascal.sahanaya.net/html;
    index index.html index.htm;

    location / {
        try_files uri uri/ =404;
    }
}
```

### Option B: Reverse Proxy (Node.js, Python, Docker, etc.)

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name pascal.sahanaya.net;

    location / {
        proxy_pass http://127.0.0.1:3000; # Replace 3000 with your app's port
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host \$host;
        proxy_cache_bypass \$http_upgrade;
    }
}
```

Save and exit nano (`Ctrl + O`, `Enter`, then `Ctrl + X`).

## Step 3. Enable the Site and Install SSL

Run these commands to activate `pascal.sahanaya.net` and secure it with HTTPS:

```bash
sudo ln -s /etc/nginx/sites-available/pascal.sahanaya.net /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
sudo certbot --nginx -d pascal.sahanaya.net
```
