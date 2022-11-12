# Web Server Setup

## Prerequisites

- An Ubuntu 22.10 server with 4GB RAM and 2 CPUs.
- An Ubuntu Desktop to connect to server
- A domain name
- github repos

## Technology Stack

### Server

- UFW
- Nginx
- PostgreSQL
- Nodejs
- npm
- ElasticSearch
- pm2

### Desktop Clients

- DBeaver

## Contents

[1. Initial Server Setup](#1-initial-server-setup)

- [1.1 Create an SSH key pair](#11-create-an-ssh-key-pair)
- [1.2 Login as root](#12-login-as-root)
- [1.3 Create a new user](#13-create-a-new-user)
- [1.4 Set Up Basic Firewall](#14-set-up-basic-firewall)
- [1.5 SSH directly into the created account](#15-ssh-directly-into-the-created-account)

[2. Nginx Web Server](#2-nginx-web-server)

- [2.1 Install Nginx](#21-install-nginx)
- [2.2 Adjust the Firewall](#22-adjust-the-firewall)
- [2.3 Check your Web Server](#23-check-your-web-server)
- [2.4 Nginx Process management](#24-nginx-process-management)
- [2.5 Modify default server block to proxy API requests](#25-modify-default-server-block-to-proxy-api-requests)
- [2.6 Set up Server Blocks](#26-set-up-server-blocks)
- [2.7 Nginx configurations](#27-nginx-configurations)

[3. PostgreSQL Database](#3-postgresql-database)

- [3.1 Install PostgreSQL](#31-install-postgresql)
- [3.2 Create a New Role](#32-create-a-new-role)
- [3.3 Create the Production Database](#33-create-the-production-database)
- [3.4 Set password for the ident user](#34-set-password-for-the-ident-user)
- [3.5 Backup and restore Production Database](##35-backup-and-restore-production-database)
- [3.6 Shell script to restore production database locally](##36-shell-script-to-restore-production-database-locally)

[4. Dbeaver DB Client](#4-dbeaver-db-client)

- [4.1 Remote connection](#41-remote-connection)
- [4.2 Configure SSH tunnel](#42-configure-ssh-tunnel)
- [4.3 Specify DB details](#43-specify-db-details)

[5. Node.js and npm](#5-nodejs-and-npm)

- [5.1 Install Node.js](#51-install-nodejs)
- [5.2 Install npm](#52-install-npm)
- [5.3 Set Node.js NODE_ENV](#53-set-nodejs-node_env)

[6. Elastic Search](#6-elastic-search)

- [6.1 Install OpenJDK 11](#61-install-openjdk-11)
- [6.2 Install Elasticsearch](#62-install-elasticsearch)
- [6.3 Configure Elasticsearch](#63-configure-elasticsearch)
- [6.4 Create the application index](#64-create-the-application-index)
- [6.5 Elasticsearch API](#65-elasticsearch-api)

[7. pm2 Node.js process manager](#7-pm2-nodejs-process-manager)

- [7.1 Install pm2](#71-install-pm2)

### Add

- DNS setup in amazon
- workflows for CI/CD
- server blocks for subdomain
- environment variable substitution

- add all .env in .bash_profile
- also for config.json

- nginx basic auth

## References

- [https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04)
- [https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04)
- [https://www.digitalocean.com/community/tutorials/how-to-install-postgresql-on-ubuntu-20-04-quickstart](https://www.digitalocean.com/community/tutorials/how-to-install-postgresql-on-ubuntu-20-04-quickstart)
- [https://github.com/dbeaver/dbeaver/issues/17537](https://github.com/dbeaver/dbeaver/issues/17537)
- [https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-20-04](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-20-04)
- [https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-on-ubuntu-22-04](https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-on-ubuntu-22-04)
- [https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-elasticsearch-on-ubuntu-20-04](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-elasticsearch-on-ubuntu-20-04)

# 1. Initial Server Setup

## 1.1 Create an SSH key pair

1. In your local computer create a new key pair

```
ssh-keygen -t rsa -b 2048 -m PEM
```

\*`PEM` format: valid for DBeaver

2. Enter file in which to save the key

```
id_rsa_do_mistymap
```

3. Copy the contents of the **.pub** file and give it the same name

```
cat ~/.ssh/id_rsa_do_mistymap.pub
```

---

## 1.2 Login as root

1. Find your server's public IP address

2. Connect as root with ssh

```
ssh root@207.154.205.22
```

---

## 1.3 Create a new user

Connected as root

1. Add user

```
adduser ioannis
```

2. Grant Administrative Privileges

Make the create user a **superuser** / give **root** privileges:

```
usermod -aG sudo ioannis
```

---

## 1.4 Set Up Basic Firewall

Make sure only connections to certain services are allowed

1. Check applications that registered their profiles with UFW upon installation.

```
ufw app list
```

2. Allow SSH connections to be able to log back in next time

```
ufw allow OpenSSH
```

3. Enable UFW

```
ufw enable
```

4. Check allowed SSH connections

```
ufw status
```

---

## 1.5 SSH directly into the created account

1. Copy root user's ssh file and directory structure to the new user account

```
rsync --archive --chown=ioannis:ioannis ~/.ssh /home/ioannis
```

> This will copy the root user’s .ssh directory, preserve the permissions, and modify the file owners, all in a single command.

[Contents](#contents)

---

# 2 Nginx Web Server

## 2.1 Install Nginx

1. Add package

```
sudo apt update
sudo apt install nginx
```

---

## 2.2 Adjust the Firewall

1. List the application configurations available to ufw:

```
sudo ufw app list
```

2. Enable Nginx Full

```
sudo ufw allow 'Nginx Full'
```

3. Verify the change

```
sudo ufw status
```

---

## 2.3 Check your Web Server

1. check with the `systemd` init system if the service is running

```
systemctl status nginx
```

2. Request a page from Nginx

Type the IP in the browser URL bar:

```
http://207.154.205.22/
```

You should see the "Welcome to nginx" page

---

## 2.4 Nginx Process management

1. Stop the web server

```
sudo systemctl stop nginx
```

2. Start the web server

```
sudo systemctl start nginx
```

3. Restart the process

```
sudo systemctl restart nginx
```

4. Reload configuration changes without dropping connections

```
sudo systemctl reload nginx
```

5. Disable automatic start at server boot

```
sudo systemctl disable nginx
```

6. Re-enable service to start up at boot

```
sudo systemctl enable nginx
```

---

## 2.5 Modify default server block to proxy API requests

**Only for testing**

1. Backup default server block

```
cd /etc/nginx/sites-available
sudo cp default default.bak
```

2. Add `/api` directive to default server block to proxy to node server later

```
location /api {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header Access-Control-Allow-Origin *;
        # Basic Auth
        #   auth_basic "Restricted Content";
        # auth_basic_user_file /etc/nginx/.htpasswd;
}

```

3. Reload nginx

```
sudo systemctl reload nginx
```

4. Confirm change

Navigate to URL

```
http://207.154.205.22/api
```

expect a 502 Bad Gateway response (instead of 404)

---

## 2.6 Set up Server Blocks

1. Create `/var/www` directory for admin

```
sudo mkdir -p /var/www/admin.mistymap.com/html
```

2. Assign ownership of the directory with the $USER environment variable

```
sudo chown -R $USER:$USER /var/www/admin.mistymap.com/html
```

3. Allow the owner to read, write, and execute the files while granting only read and execute permissions to groups and others

```
sudo chmod -R 755 /var/www/app.mistymap.com
```

4. Create a sample `index.html`:

```
sudo nano /var/www/app.mistymap.com/html/index.html
```

```
<html>
    <head>
        <title>Welcome to admin.mistymap.com!</title>
    </head>
    <body>
        <h1>Success!  The admin.mistymap.com server block is working!</h1>
    </body>
</html>
```

5. Create a server block for the subdomain

```
sudo nano /etc/nginx/sites-available/admin.mistymap.com
```

```
server {
	listen 80;
	listen [::]:80;

	#  Basic auth to block crawlers
	auth_basic "Administrator’s Area";
	auth_basic_user_file /etc/apache2/.htpasswd;

	root /var/www/admin.mistymap.com/html;
	index index.html index.htm index.nginx-debian.html;

	server_name admin.mistymap.com www.admin.mistymap.com;

	location /api {
		proxy_pass http://localhost:5000;
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection 'upgrade';
		proxy_set_header Host $host;
		proxy_cache_bypass $http_upgrade;
		proxy_set_header Access-Control-Allow-Origin *;
	}

	location / {
		# React router index.html response
		try_files $uri /index.html;
	}
}
```

6. Enable the file by creating a link from it to the `sites-enabled` directory,

```
sudo ln -s /etc/nginx/sites-available/admin.mistymap.com /etc/nginx/sites-enabled/
```

7. Avoid hash bucket memory problem

Adjust the value in `/etc/nginx/nginx.conf`

```
sudo nano /etc/nginx/nginx.conf
```

and uncomment

```
server_names_hash_bucket_size 64;
```

8. Test nginx files syntax

```
sudo nginx -t
```

9. Rest nginx

```
sudo systemctl restart nginx
```

---

# 2.7 Nginx configurations

1. Increase file size to upload. 413 – Request Entity Too Large Error and Solution

- Open `nginx.conf`

```
sudo nano /etc/nginx/nginx.conf
```

- Add the following line to http or server or location context to increase the size limit in nginx.conf, enter:

```
# set client body size to 2M #
client_max_body_size 2M;
```

## [Contents](#contents)

# 3. PostgreSQL Database

## 3.1 Install PostgreSQL

1. Install the Postgres package along with a `-contrib` package that adds some additional utilities and functionality

```
sudo apt update
sudo apt install postgresql postgresql-contrib
```

2. Ensure that the service is started

```
sudo systemctl start postgresql.service
```

---

## 3.2 Create a New Role

Postgres is set up to use **ident authentication**, meaning that it associates Postgres roles with a matching Unix/Linux system account.

1. Use sudo to create role without switching from normal account

```
sudo -u postgres createuser --interactive
```

---

## 3.3 Create the Production Database

Postgres authentication expects by default that any role used to log in, will have a database with the same name which it can access.

1. Create a database with the same name as the ident user

```
sudo -u postgres createdb ioannis
```

2. Create the production database with the ident user as owner

```
sudo -u ioannis createdb mistymap_production
```

3. Enter the Postgres prompt as the ident user

```
psql
```

4. List all databases

```
ioannis=# \l
```

| Name                | Owner    |
| ------------------- | -------- |
| ioannis             | postgres |
| mistymap_production | ioannis  |
| postgres            | postgres |

---

## 3.4 Set password for the ident user

1. Connect to the PosteSQL command prompt:

```
psql
```

2. Set password for the user

```
ALTER USER ioannis WITH PASSWORD '********';
```

the password is required for the authentication of the Dbeaver connection

---

## 3.5 Backup and restore Production Database

1. SSH into server and create dump

```
pg_dump -U ioannis -f mistymap_production.dump mistymap_production -Fc
```

2. Download database locally

3. Restore database

```
pg_restore -U user -d mistymap_development -1 mistymap_production.dump
```

---

## 3.6 Shell script to restore production database locally

TODO: change default port

File: `db_restore.sh`

```
echo "---------- Connect to server ----------";
ssh -i ~/.ssh/id_rsa_do_mistymap ioannis@207.154.205.22 << EOF
pwd;
whoami;
rm -f mistymap_production.dump;
echo "---------- Dump production database ----------";
pg_dump -U ioannis -f mistymap_production.dump mistymap_production -Fc;
EOF
pwd;
whoami;
echo "---------- Clone production database locally ----------";
rm -f mistymap_production.dump;
rsync -chavzP -e "ssh -p 22" ioannis@207.154.205.22:/home/ioannis/mistymap_production.dump /home/user/Documents/Scripts;
echo "---------- Drop and restore database ----------";
sudo -u postgres dropdb --if-exists 'mistymap_development';
sudo -u postgres createdb 'mistymap_development';
pg_restore -U user -d mistymap_development -1 mistymap_production.dump
echo "---------- Completed successfully ----------";
```

[Contents](#contents)

---

# 4. Dbeaver DB Client

## 4.1 Remote connection

Dbeaver's SSH tunnel won't work by default on Ubuntu 22.04.

1. Allow connection with rsa SSH public key

On the server's file `/etc/ssh/sshd_config` add:

```
PubkeyAcceptedKeyTypes=+ssh-rsa
```

and restart the ssh daemon

```
sudo systemctl restart ssh
```

---

## 4.2 Configure SSH tunnel

In Dbeaver create a new connection and in the SSH tab click on **Use SSH Tunnel**

1. Set

- `Host/IP: 207.154.205.22`
- `Port: 22` (default - for now)
- `User Name: ioannis` (of Ubuntu server)

2. Set `Authentication Method: Public key`

3. Specify location of the Private key

4. Test tunnel configuration

---

## 4.3 Specify DB details

In the Main tab of the Dbeaver connection:

1. Set

- `Host: localhost`
- `Port 5432` (default - for now)

2. Name of the production database: `mistymap_production`

3. Ident user authentication for the database server

- Username: `ioannis`
- Password: `********` (The use that was set in step 3.4)

[Contents](#contents)

---

# 5. Node.js and npm

## 5.1 Install Node.js

1. Refresh your local package index

```
sudo apt update
```

2. Install Node.js

```
sudo apt install nodejs
```

3. Check installed node version

```
node -v
```

---

## 5.2 Install npm

1. Install Node.js package manager

```
sudo apt install npm
```

## 5.3 Set Node.js NODE_ENV

1. Add the NODE_ENV in `bash_profile`

In `~/.bash_profile` add the line:

```
export NODE_ENV="production"
```

2. Source the file to enable env

```
source ~/.bash_profile
```

3. Test NODE_ENV env is enabled

```
printenv NODE_ENV
```

[Contents](#contents)

---

# 6. Elastic Search

## 6.1 Install OpenJDK 11

1. Execute the following command to install the JRE from OpenJDK 11

```
sudo apt install default-jre
```

2. Verify the installation with

```
java -version
```

---

## 6.2 Install Elasticsearch

1. Import the Elasticsearch public GPG key into APT with cURL

```
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

2. Add the Elastic source list to the `sources.list.d` directory

```
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
```

3. Update your package lists so APT will read the new Elastic source

```
sudo apt update
```

4. Install Elasticsearch

```
sudo apt install elasticsearch
```

---

## 6.3 Configure Elasticsearch

Restrict outside access by:

1. Open config file

```
sudo nano /etc/elasticsearch/elasticsearch.yml
```

2. Update `network.host`

```
network.host: localhost
```

3. Start the Elasticsearch service

```
sudo systemctl start elasticsearch
```

4. Enable Elasticsearch to start up on server boot

```
sudo systemctl enable elasticsearch
```

5. Test Elasticsearch is listening

```
curl http://localhost:9200/
```

---

## 6.4 Create the application index

For the purposes of this app, we need a mapping on `filter_suggest` of type **search_as_you_type**

1. Create the index `exhibits` with `filter_suggest` mappings

```
curl -X PUT -H "Content-Type: application/json" -d '{
    "mappings": {
        "properties": {
            "filter_suggest": {
                "type": "search_as_you_type"
            }
        }
    }
}' "http://localhost:9200/exhibits"
```

2. Check created index

```
curl http://localhost:9200/_cat/indices
```

## 6.5 Elasticsearch API

1. Delete an index

```
curl -X DELETE http://localhost:9200/exhibits -H "Accept: application/json"
```

2. Search for a document with the current index

```
curl -X POST -H "Content-Type: application/json" \
    -d '{
        "query": {
            "multi_match": {
                "query": "a",
                "type": "bool_prefix",
                "fields": [
                    "title",
                    "subtitle",
                    "filter_suggest",
                    "filter_suggest._2gram",
                    "filter_suggest._3gram"
                ],
                "operator": "and"
            }
        }
    }' \
    http://localhost:9200/exhibits/_search
```

3. Get all documents in index

```
curl http://localhost:9200/exhibits/_search
```

[Contents](#contents)

---

# 7. pm2 Node.js process manager

## 7.1 Install pm2

1. Install PM2 by typing the following at the command line

```
sudo npm install pm2 -g
```

2. Run the app with pm2

```
pm2 start app.js -i 0
```

[Contents](#contents)

---
