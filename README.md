# Odoo-Web-Application---Ecoquant.ai
Migrate the existing Odoo setup from the old server (website-server-ecoquant-cj) to a new web server, including Docker containers, PostgreSQL database, file storage, and Nginx configuration.
# ODOO Installation
 # Steps to install ODOO and host it on AWS EC2 Instance
 
Login to AWS and Launch an Instance.
 Make sure you have enabled ports 80,443,8069(odoo),5432(postgres)
 After adding all the settings launch the instance.
 # Commands to execute after SSH into the VM
  sudo apt-get update
  sudo apt upgrade
  sudo apt install docker-compose
 # Create a directory and name it as odoo at path /home/ubuntu/odoo
 Enter the odoo directory and create a file named docker-compose.yml and paste the below contents.
  a. sudo vim docker-compose.yml
 
# Create a file named .env and paste the below contents
 a. sudo vim .env
 
# To start the docker container via docker compose use below command, make sure you are inside the folder where file is present.
 sudo docker-compose up -d
# To check if ODOO is running properly on the server, use the command below and you will get status - 200 if it is working fine.
  curl --head http://localhost:8069
 1 version: '3'
 2 services:
 3  odoo:
 4    image: odoo:16.0
 5    env_file: .env
 6    depends_on:
 7      - postgres
 8    ports:
 9      - "0.0.0.0:8069:8069"  
10    volumes:
 11      - data:/var/lib/odoo
 12      - ./addons:/mnt/extra-addons
 13
 14  postgres:
 15    image: postgres:13
 16    env_file: /home/ubuntu/odoo/.env
 17    volumes:
 18      - db:/var/lib/postgresql/data/pgdata
 19    ports:
 20      - "0.0.0.0:5432:5432"
 21 volumes:
 22  data:
 23  db:
 # Create a file named .env and paste the below contents
     a. sudo vim .env
 1 # postgresql environment variables
 2 POSTGRES_DB=postgres
 3 POSTGRES_PASSWORD=admin
 4 POSTGRES_USER=odoo
 5 PGDATA=/var/lib/postgresql/data/pgdata
 6
 7 # odoo environment variables
 8 HOST=postgres
 9 USER=odoo
 10 PASSWORD=admin
 # To start the docker container via docker compose use below command, make sure you are inside the folder where file is present.
    sudo docker-compose up -d
 # To check if ODOO is running properly on the server, use the command below and you will get status - 200 if it is working fine.
   curl --head http://localhost:8069
 # Installing and Configuring Nginx
 Note : Before following this process make sure you have added A Record in DNS Hosted Zone 
(For Ex - AWS Route 53) (“A Record” added with Domain name and IP Address of Instance  )
 # Commands to install nginx on Ubuntu
 1.sudo apt update
 2.sudo apt install nginx
 # Allow public traffic to ports 
 80 and 443 (HTTP and HTTPS) using the Nginx Full UFW application profile
 # Creating and configuring config file for nginx.
  sudo nano /etc/nginx/sites-available/odoo.conf
  server {
  listen       80;
    listen       [::]:80;
    server_name  iqodoo.evalue8.it; #(example)
    access_log  /var/log/nginx/odoo.access.log;
    error_log   /var/log/nginx/odoo.error.log;
    location / {
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-Host $host;
      proxy_set_header X-Forwarded-Proto https;
      proxy_pass http://localhost:8069
      }
      }
 #  Save and close the file, then enable the configuration by linking it into /etc/nginx/sites-enabled/
    sudo ln -s /etc/nginx/sites-available/odoo.conf /etc/nginx/sites-enabled/
 # Use nginx -t to verify that the configuration file syntax is correct  
      sudo nginx -t
 #  Reload the nginx service with command
      sudo systemctl reload nginx.service
 # Installing Certbot and Setting Up TLS Certificates
  Install Certbot and its Nginx plugin
  sudo apt install certbot python3-certbot-nginx
  sudo certbot --nginx -d iqodoo.evalue8.it
