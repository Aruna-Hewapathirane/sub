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
