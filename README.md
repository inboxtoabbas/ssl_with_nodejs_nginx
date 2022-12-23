# ssl_with_nodejs_nginx
How to install SSL on specific domain or subdomain for NodeJs Application

# Node.js Deployment

> Steps to deploy a Node.js app to DigitalOcean using PM2, NGINX as a reverse proxy and an SSL from LetsEncrypt

## 1. Login AWS Console
Its always recommended to login via IAM User ID & Password

## 2. Create a new Instance
 Launch an EC2 instance Ubuntu 20.4 version with SSH, HTTP & HTTPS

## 3. Install Node/NPM
-----
cd ~
curl -sL https://deb.nodesource.com/setup_16.x -o /tmp/nodesource_setup.sh

Use nano or Vim  sudo nano /tmp/nodesource_setup.sh

sudo bash /tmp/nodesource_setup.sh

sudo apt install nodejs

node -v
Output
v16.6.1

## 4. Upload your API code thru SFTP / Clone your project from Github
I would suggest git to get your files on to the server

### 4.A Install Git on your EC2 instance
sudo apt install git
git --version 

git clone yourproject.git
-----

## 5. Install dependencies and test app
-----
cd yourproject
npm install
npm start (or whatever your start command)
# stop app
ctrl+C
-----
## 6. Setup PM2 process manager to keep your app running
-----
sudo npm i pm2 -g
pm2 start app (or whatever your file name)

## Other pm2 commands
pm2 show app
pm2 status
pm2 restart app
pm2 stop app
pm2 logs (Show log stream)
pm2 flush (Clear logs)

# To make sure app starts when reboot
pm2 startup ubuntu
---
### Now you should be able to access your app using your IP and the Port you mentioned in your app for e.g PORT 8000. Now we want to setup a firewall blocking that port and setup NGINX as a reverse proxy so we can access it directly using port 80 (http)

## 7. Setup ufw firewall
---
sudo ufw enable
sudo ufw status
sudo ufw allow ssh (Port 22)
sudo ufw allow http (Port 80)
sudo ufw allow https (Port 443)
----

## 8. Install NGINX and configure
----
sudo apt install nginx

sudo nano /etc/nginx/sites-available/default
----
Add the following to the location part of the server block
----
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:8000; #whatever port your app runs on
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

-----
# Check NGINX config
sudo nginx -t

# Restart NGINX
sudo service nginx restart
-----

### You should now be able to visit your IP with no port (port 80) and see your app. Now let's add a domain

## Create Load Balancer & Target Group

## 9. Add domain in Route53
In AWS -> Go to Route53 -> DNS Management -> Hosted Zone

Add an A record for your domain / Subdomain and pointed to your Application Load Balacer Public URL


10. Now lets add SSL with LetsEncrypt
----
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-nginx
sudo certbot --nginx -d xyz.com -d www.xyz.com

# Only valid for 90 days, test the renewal process with
certbot renew --dry-run


Now visit https://xyz.com and you should see your Node app
