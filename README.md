# Web Server Setup

## Prerequisites

- An Ubuntu 22.10 server with 4GB RAM and 2 CPUs.
- An Ubuntu Desktop to connect to server

## Technology Stack

### Server

- UFW
- Nginx
- PostgreSQL

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

[3. PostgreSQL Database](#3-postgresql-database)

- [3.1 Install PostgreSQL](#31-install-postgresql)
- [3.2 Create a New Role](#32-create-a-new-role)
- [3.3 Create the Production Database](#33-create-the-production-database)
- [3.4 Set password for the ident user](#34-set-password-for-the-ident-user)

[4. Dbeaver DB Client Setup](#4-dbeaver-db-client-setup)

- [4.1 Remote connection](#41-remote-connection)
- [4.2 Configure SSH tunnel](#42-configure-ssh-tunnel)
- [4.3 Specify DB details](#43-specify-db-details)

## References

- [https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04)
- [https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04)
- [https://www.digitalocean.com/community/tutorials/how-to-install-postgresql-on-ubuntu-20-04-quickstart](https://www.digitalocean.com/community/tutorials/how-to-install-postgresql-on-ubuntu-20-04-quickstart)
- [https://github.com/dbeaver/dbeaver/issues/17537](https://github.com/dbeaver/dbeaver/issues/17537)

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

[Contents](#contents)

---

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

[Contents](#contents)

---

# 4. Dbeaver DB Client Setup

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
