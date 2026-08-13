---
sidebar_position: 2
title: V-Server Setup
description: Secure initial setup of an Ubuntu server with SSH keys and Nginx.
---

import GithubLinkAdmonition from '@site/src/components/GithubLinkAdmonition';

# V-Server Setup

This project documents how I configured and secured my first Ubuntu-based virtual server during the Developer Akademie DevSecOps course. The server uses Ed25519 key-based SSH access and serves two websites through Nginx.

<GithubLinkAdmonition
  link="https://github.com/FriggemannMichael/V-Server"
  title="Project repository"
  type="tip"
>
  The repository contains the complete setup documentation.
</GithubLinkAdmonition>

## Contents

- [Project overview](#project-overview)
- [Server information](#server-information)
- [Create a local SSH key](#create-a-local-ssh-key)
- [Install the public key](#install-the-public-key)
- [Disable SSH password authentication](#disable-ssh-password-authentication)
- [Install Nginx](#install-nginx)
- [Configure an alternative website](#configure-an-alternative-website)
- [Create an SSH alias](#create-an-ssh-alias)
- [Verification](#verification)
- [Security considerations](#security-considerations)

## Project overview

The goal was to configure secure remote access, disable password-based SSH authentication, install Nginx, and publish an additional HTML page on a separate port. I ran all local commands with Git Bash.

## Server information

For security reasons, the public documentation uses example values for the IP address and SSH user.

| Property | Value |
| --- | --- |
| Operating system | Ubuntu 24.04.4 LTS |
| Public IP address | `203.0.113.10` (example) |
| SSH user | `server-user` (example) |
| Web server | Nginx 1.24.0 |
| Default website | `http://203.0.113.10` (example) |
| Alternative website | `http://203.0.113.10:8081` (example) |

## Create a local SSH key

I generated the SSH key pair on my local computer. A dedicated directory keeps the course key separate from other SSH identities.

```bash
mkdir -p ~/.ssh/vserver-kurs
chmod 700 ~/.ssh/vserver-kurs
ssh-keygen -t ed25519 -f ~/.ssh/vserver-kurs/vserver_ed25519 -C "vserver-kurs"
```

The command creates two files:

- `vserver_ed25519` is the private key and must remain secret and local.
- `vserver_ed25519.pub` is the public key that can be installed on the server.

I restricted the file permissions:

```bash
chmod 600 ~/.ssh/vserver-kurs/vserver_ed25519
chmod 644 ~/.ssh/vserver-kurs/vserver_ed25519.pub
```

## Install the public key

I added the public key to the server user's `~/.ssh/authorized_keys` file:

```bash
ssh-copy-id -i ~/.ssh/vserver-kurs/vserver_ed25519.pub \
  server-user@203.0.113.10
```

I then tested key-based login explicitly:

```bash
ssh -i ~/.ssh/vserver-kurs/vserver_ed25519 \
  -o IdentitiesOnly=yes \
  server-user@203.0.113.10
```

On the server, I verified the active account:

```bash
whoami
```

Expected result:

```text
server-user
```

## Disable SSH password authentication

I disabled password authentication only after key-based login had been tested successfully. I created the following server configuration file:

```bash
sudo nano /etc/ssh/sshd_config.d/00-disable-password.conf
```

The file contains:

```sshconfig
PasswordAuthentication no
KbdInteractiveAuthentication no
```

Before loading the configuration, I validated it and checked the effective settings:

```bash
sudo sshd -t
sudo sshd -T | grep -E \
  '^(passwordauthentication|kbdinteractiveauthentication|pubkeyauthentication) '
```

Expected settings:

```text
passwordauthentication no
kbdinteractiveauthentication no
pubkeyauthentication yes
```

I reloaded and checked the SSH service:

```bash
sudo systemctl reload ssh
sudo systemctl is-active ssh
```

Finally, I verified from my local computer that a password-only login is rejected:

```bash
ssh \
  -o PubkeyAuthentication=no \
  -o KbdInteractiveAuthentication=no \
  -o PreferredAuthentications=password \
  server-user@203.0.113.10
```

The server responded with `Permission denied (publickey)`, confirming that password authentication was disabled.

## Install Nginx

I updated the package index and installed Nginx:

```bash
sudo apt update
sudo apt install -y nginx
```

I verified the service state and automatic startup:

```bash
sudo systemctl is-active nginx
sudo systemctl is-enabled nginx
```

I also tested the default website locally on the server:

```bash
curl -I http://127.0.0.1
```

An `HTTP/1.1 200 OK` response confirmed that Nginx was serving the default page. I also opened the page in a browser using the server's public IP address.

## Configure an alternative website

I created a separate document root and HTML entry file:

```bash
sudo mkdir -p /var/www/alternativs
sudo nano /var/www/alternativs/alternate-index.html
```

I added the following Nginx server block to `/etc/nginx/sites-enabled/alternativs`:

```nginx
server {
    listen 8081;
    listen [::]:8081;

    root /var/www/alternativs;
    index alternate-index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

I validated and loaded the configuration:

```bash
sudo nginx -t
sudo systemctl reload nginx
sudo systemctl is-active nginx
```

I tested the alternative website on the server and in a browser:

```bash
curl http://127.0.0.1:8081
```

Example browser address: `http://203.0.113.10:8081`

## Create an SSH alias

I opened my local SSH client configuration:

```bash
nano ~/.ssh/config
```

I added the following host definition:

```sshconfig
Host vserver-kurs
    HostName 203.0.113.10
    User server-user
    IdentityFile ~/.ssh/vserver-kurs/vserver_ed25519
    IdentitiesOnly yes
```

I protected the configuration file and tested the alias:

```bash
chmod 600 ~/.ssh/config
ssh vserver-kurs
```

## Verification

| Check | Result |
| --- | --- |
| Local Ed25519 key pair created | Passed |
| Public key installed on the server | Passed |
| SSH key login works | Passed |
| Password-only SSH login is rejected | Passed |
| Nginx service is active | Passed |
| Default Nginx page is available | Passed |
| Alternative page on port 8081 is available | Passed |
| SSH alias works | Passed |

## Security considerations

- The private SSH key remains on the local computer.
- Passwords, private keys, and key passphrases are not stored in the repository.
- Password authentication was disabled only after key-based login worked in a separate session.
- SSH and Nginx configuration changes were validated before the services were reloaded.
- The dedicated SSH key directory prevents accidental use of the wrong identity.

## Further references

- [Ubuntu Server documentation](https://documentation.ubuntu.com/server/)
- [OpenSSH server documentation](https://documentation.ubuntu.com/server/how-to/security/openssh-server/)
- [Nginx documentation](https://nginx.org/en/docs/)
