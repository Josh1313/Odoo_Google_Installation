# Google Cloud Setup for Odoo with Docker and Nginx

---

### Step 1: Check Disk Space
```bash
df -h
```

### Step 2: Update System Packages
```bash
sudo apt update
sudo apt-get update
```

### Step 3: Install Unzip
```bash
sudo apt-get install unzip -y
```

### Step 4: Install Docker
```bash
sudo apt install docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

### Step 5: Add User to Docker Group
```bash
whoami
sudo usermod -aG docker user
exit
```
Login again and verify the user is added to the Docker group.

---

### Step 6: Install Docker Compose
```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

### Step 7: Install Git
```bash
sudo apt install -y git
```

### Step 8: Clone Odoo Repository
```bash
git clone https://github.com/Josh1313/Odoo.git
cd Odoo
git switch Development/Production
```

### Step 9: Start Docker Compose
```bash
docker-compose up
sudo docker-compose up -d
```

### Step 10: Install ReportLab Fonts
```bash
wget http://www.reportlab.com/ftp/fonts/pfbfer.zip
unzip pfbfer.zip -d pfbfer_fonts
sudo mkdir -p /usr/lib/python3/dist-packages/reportlab/fonts
sudo cp pfbfer_fonts/* /usr/lib/python3/dist-packages/reportlab/fonts/
```

### Step 11: Restart Docker Compose
```bash
docker-compose down
docker-compose up
```

### Step 12: Configure Odoo
```bash
sudo nano ./config/odoo.conf
```
Add the following configuration:
```ini
[options]
addons_path = /mnt/extra-addons
admin_passwd = admin
db_host = db
db_port = 5432
db_user = odoo
db_password = datapathfinder
proxy_mode = True
```
Save and exit with `CTRL+O`, `ENTER`, `CTRL+X`.

---

### Step 13: Install Certbot and Nginx
```bash
sudo apt install certbot python3-certbot-nginx -y
sudo apt install -y nginx
```

### Step 14: Configure Nginx for Odoo
```bash
sudo nano /etc/nginx/sites-available/Odoo
```
Add the following configuration:
```nginx
server {
    listen 8081;
    server_name googleodoo.zapto.org;

    location / {
        proxy_pass http://localhost:9069;
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
        proxy_buffering off;
        proxy_cache off;

        # Headers for WebSocket support
        proxy_set_header Connection 'Upgrade';
        proxy_set_header Upgrade $http_upgrade;

        # Additional headers for forwarding client info
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
Save and exit with `CTRL+O`, `ENTER`, `CTRL+X`.

### Step 15: Enable Nginx Site and Test Configuration
```bash
sudo ln -s /etc/nginx/sites-available/Odoo /etc/nginx/sites-enabled/
sudo nginx -t
```

### Step 16: Set Up SSL with Certbot
```bash
sudo certbot --nginx -d googleodoo.zapto.org
```

### Step 17: Restart Nginx
```bash
sudo systemctl restart nginx

