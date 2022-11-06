# Web Server Setup

## Prerequisites

- An Ubuntu 22.10 server with 4GB RAM and 2 CPUs.
- An Ubuntu Desktop to connect to server

## Technology Stack

## Contents

[1. Initial Server Setup](#1-initial-server-setup)

- [1.1 Create an SSH key pair](#11-create-an-ssh-key-pair)
- [1.2 Login as root](#12-login-as-root)
- [1.3 Create a new user](#13-create-a-new-user)
- [1.4 Set Up Basic Firewall](#14-set-up-basic-firewall)
- [1.5 SSH directly into the created account](#15-ssh-directly-into-the-created-account)

---

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
