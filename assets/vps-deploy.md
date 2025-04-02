# Deploy Next.js Apps on VPS

This streamlined guide helps you deploy several Next.js apps on one VPS, each with its own domain and HTTPS.

## Prerequisites

- A VPS with Ubuntu 20.04/22.04
- SSH access with sudo privileges
- Multiple domain names pointing to your VPS
- Next.js apps in Git repositories

## 1. Server Setup (One-Time)

```bash
# Connect to your VPS
ssh username@vps_ip

# Update system & install essentials
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl wget unzip nginx

# Install Node.js via nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
nvm install --lts

# Install PM2 globally
sudo npm install pm2@latest -g

# Configure firewall
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

## 2. Deploy Each App (Repeat Per App)

### 2.1 Setup App Directory

```bash
# For App 1
APP_NAME="app1"
APP_DIR="/var/www/$APP_NAME"
sudo mkdir -p "$APP_DIR"
sudo chown $USER:$USER "$APP_DIR"
cd "$APP_DIR"
git clone "your-git-repo-url" .

# Install & build
npm install
npm run build
```

### 2.2 Configure App Port

Edit `package.json` to set a unique port:

```json
{
  "scripts": {
    "start": "next start -p 3001"  // Unique port for App 1 if more than one app is hosted
  }
}
```

### 2.3 Start With PM2

```bash
cd "$APP_DIR"
pm2 start npm --name "$APP_NAME" -- start
```

## 3. Configure Nginx For Each Domain

### 3.1 Create Config File

```bash
sudo nano /etc/nginx/sites-available/app1.com
```

Add configuration:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name 88.69.106.143 app1.com www.app1.com;

    location / {
        proxy_pass http://localhost:3001;  # App 1 port
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 3.2 Enable Configuration

```bash
sudo ln -s /etc/nginx/sites-available/app1.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
sudo service restart nginx
```

## 4. Add SSL (HTTPS)

```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Get certificate for each domain
sudo certbot --nginx -d app1.com -d www.app1.com
```

## 5. Setup PM2 Persistence

```bash
# Configure PM2 to start on boot
pm2 startup
# Run the command PM2 provides
pm2 save
```

## 6. Monitoring & Updates

```bash
# View status
pm2 list

# View logs
pm2 logs app1

# Update an app
cd /var/www/app1
git pull origin main
npm install
npm run build
pm2 restart app1
```

That's it! Repeat steps 2-4 for each additional Next.js app (using unique ports and domain names).

---

## Contribution
Want to suggest VPS deployment methods? Open a pull request!
