---
sidebar_position: 2
title: V-Server Setup
description: Secure initial setup of an Ubuntu server with SSH keys, Git, GitHub, and Nginx.
---

import GithubLinkAdmonition from '@site/src/components/GithubLinkAdmonition';

# V-Server Setup

This guide explains how to configure and secure an Ubuntu-based virtual server. The resulting server uses Ed25519 key-based SSH access, can authenticate with GitHub, and serves two websites through Nginx.

<GithubLinkAdmonition
  link="https://github.com/FriggemannMichael/V-Server"
  title="Project repository"
  type="tip"
>
  The repository contains the standalone setup documentation.
</GithubLinkAdmonition>

## Table Of Contents

- [Quickstart](#quickstart)
  - [Prerequisites](#prerequisites)
- [Usage](#usage)
  - [Configure local SSH access](#configure-local-ssh-access)
  - [Disable SSH password authentication](#disable-ssh-password-authentication)
  - [Configure Git on the server](#configure-git-on-the-server)
  - [Connect the server to GitHub](#connect-the-server-to-github)
  - [Install Nginx](#install-nginx)
  - [Configure an alternative website](#configure-an-alternative-website)
  - [Create a local SSH alias](#create-a-local-ssh-alias)
  - [Validation](#validation)
- [Additional Information](#additional-information)
  - [Project checklist](#project-checklist)
  - [Server information](#server-information)
  - [Security considerations](#security-considerations)
  - [Further references](#further-references)

## Quickstart

### Prerequisites

The setup requires:

- an Ubuntu 24.04 V-Server;
- a server user with `sudo` permissions;
- Git and OpenSSH on the local computer;
- a GitHub account; and
- access to ports `22`, `80`, and `8081`.

The examples use the intentionally invalid IP address `123.456.789.10`. Replace it and every value enclosed in angle brackets, such as `<server_user>`, with the real connection details. Never commit real credentials, private SSH keys, or sensitive infrastructure data.

## Usage

### Configure local SSH access

Create a dedicated directory for the local server key:

```bash
mkdir -p ~/.ssh/<folder>
```

Protect the directory:

```bash
chmod 700 ~/.ssh/<folder>
```

Generate an Ed25519 key pair locally:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/<folder>/<key_name> -C "<key_comment>"
```

Restrict the private key permissions:

```bash
chmod 600 ~/.ssh/<folder>/<key_name>
```

Set the public key permissions:

```bash
chmod 644 ~/.ssh/<folder>/<key_name>.pub
```

Copy only the public key to the server:

```bash
ssh-copy-id -i ~/.ssh/<folder>/<key_name>.pub <server_user>@123.456.789.10
```

Test key-based login before changing the SSH server configuration:

```bash
ssh -i ~/.ssh/<folder>/<key_name> -o IdentitiesOnly=yes <server_user>@123.456.789.10
```

### Disable SSH password authentication

Create a dedicated OpenSSH configuration file on the server:

```bash
sudo nano /etc/ssh/sshd_config.d/00-disable-password.conf
```

Add the following settings:

```sshconfig
PasswordAuthentication no
KbdInteractiveAuthentication no
```

Validate the SSH configuration before reloading the service:

```bash
sudo sshd -t
```

Reload OpenSSH:

```bash
sudo systemctl reload ssh
```

Confirm the effective authentication settings:

```bash
sudo sshd -T | grep -E "^(passwordauthentication|kbdinteractiveauthentication|pubkeyauthentication) "
```

The expected settings are:

```text
pubkeyauthentication yes
passwordauthentication no
kbdinteractiveauthentication no
```

Keep the active SSH session open and run the following negative test from a second local terminal:

```bash
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no -o KbdInteractiveAuthentication=no -o NumberOfPasswordPrompts=1 <server_user>@123.456.789.10
```

The expected response is `Permission denied (publickey)`. A normal key-based login must continue to work.

### Configure Git on the server

Check that Git is installed:

```bash
git --version
```

Configure the GitHub username used for commits:

```bash
git config --global user.name "<github_username>"
```

Configure the email address associated with the GitHub account:

```bash
git config --global user.email "<github_email>"
```

Verify the configured username:

```bash
git config --global --get user.name
```

Verify the configured email address:

```bash
git config --global --get user.email
```

### Connect the server to GitHub

Check for existing SSH identities before creating a dedicated GitHub key:

```bash
ls -la ~/.ssh
```

Generate an Ed25519 key pair on the server. Do not overwrite an existing key:

```bash
ssh-keygen -t ed25519 -C "<github_email>" -f ~/.ssh/<github_key_name>
```

Display only the public key:

```bash
cat ~/.ssh/<github_key_name>.pub
```

Add the complete public key to **GitHub → Settings → SSH and GPG keys → New SSH key** as an authentication key. The private file `~/.ssh/<github_key_name>` must never be displayed, copied, or committed.

Open the server's SSH client configuration:

```bash
nano ~/.ssh/config
```

Configure GitHub to use the dedicated key:

```sshconfig
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/<github_key_name>
    IdentitiesOnly yes
```

Protect the SSH client configuration:

```bash
chmod 600 ~/.ssh/config
```

Test authentication from the server to GitHub:

```bash
ssh -T git@github.com
```

GitHub confirms successful authentication with the configured username and explains that it does not provide shell access.

### Install Nginx

Update the package index:

```bash
sudo apt update
```

Install Nginx:

```bash
sudo apt install -y nginx
```

Confirm that the service is active:

```bash
sudo systemctl is-active nginx
```

Confirm that Nginx starts automatically:

```bash
sudo systemctl is-enabled nginx
```

Test the default website on the server:

```bash
curl -I http://127.0.0.1/
```

Replace the example IP address in `http://123.456.789.10/`, open the resulting URL from another computer, and expect an `HTTP/1.1 200 OK` response.

### Configure an alternative website

Create a separate document root:

```bash
sudo mkdir -p /var/www/<site_name>
```

Create the alternative entry page:

```bash
sudo nano /var/www/<site_name>/index.html
```

Add the page content:

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to NEW nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<h2>Es hat funktioniert</h2>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Create the Nginx server block under `sites-available`:

```bash
sudo nano /etc/nginx/sites-available/<site_name>
```

Add the server block:

```nginx
server {
    listen 8081;
    listen [::]:8081;

    root /var/www/<site_name>;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Enable the site with a symbolic link:

```bash
sudo ln -s /etc/nginx/sites-available/<site_name> /etc/nginx/sites-enabled/<site_name>
```

Validate the Nginx configuration:

```bash
sudo nginx -t
```

Reload Nginx:

```bash
sudo systemctl reload nginx
```

Test the alternative page locally on the server:

```bash
curl -I http://127.0.0.1:8081/
```

Verify its identifying content:

```bash
curl -s http://127.0.0.1:8081/ | grep -E "Welcome to NEW nginx|Es hat funktioniert"
```

Replace the example IP address in `http://123.456.789.10:8081/`, open the resulting URL from another computer, and expect an `HTTP/1.1 200 OK` response.

### Create a local SSH alias

Open the local SSH client configuration:

```bash
nano ~/.ssh/config
```

Add a convenient host definition:

```sshconfig
Host <ssh_alias>
    HostName 123.456.789.10
    User <server_user>
    IdentityFile ~/.ssh/<folder>/<key_name>
    IdentitiesOnly yes
```

Protect the configuration file:

```bash
chmod 600 ~/.ssh/config
```

Test the alias:

```bash
ssh <ssh_alias>
```

### Validation

Run each check after replacing the example values and placeholders with the real connection details.

| Check | Command or action | Expected result |
| --- | --- | --- |
| Local Ed25519 key pair | `ls -l ~/.ssh/<folder>/<key_name>*` | Private and public key files are present with restricted permissions. |
| SSH key login | Connect with the dedicated identity. | Login succeeds without the account password. |
| Password-only SSH login | Run the negative authentication test above. | The server responds with `Permission denied (publickey)`. |
| Git identity | Run both `git config --global --get` commands. | The configured GitHub username and email are returned. |
| Server authentication to GitHub | `ssh -T git@github.com` | GitHub confirms authentication and states that shell access is unavailable. |
| Nginx configuration | `sudo nginx -t` | The syntax test reports success. |
| Default Nginx page | Open the default website from another computer. | The page loads and returns HTTP 200. |
| Alternative Nginx page | Open the website on port `8081`. | The customized page loads and returns HTTP 200. |
| Local SSH alias | `ssh <ssh_alias>` | The alias opens the server connection. |

## Additional Information

### Project checklist

The original assignment checklist is included in this repository:

- [Download the Git and V-Server checklist (PDF)](/checklists/git-vserver-checklist.pdf)

### Server information

For security reasons, the public documentation uses example values for the IP address and SSH user.

| Property | Value |
| --- | --- |
| Operating system | Ubuntu 24.04.4 LTS |
| Public IP address | `123.456.789.10` (intentionally invalid example) |
| SSH user | `<server_user>` |
| Git | 2.43.0 |
| Web server | Nginx 1.24.0 |
| Default website | `http://123.456.789.10/` (documentation example) |
| Alternative website | `http://123.456.789.10:8081/` (documentation example) |

### Security considerations

- Private SSH keys remain on the machine where they were generated.
- Passwords, private keys, key passphrases, and real infrastructure details are not stored in the repository.
- Password authentication was disabled only after key-based login worked in a separate session.
- Existing SSH keys were checked before a new identity was generated.
- GitHub's host fingerprint was verified before the host was trusted.
- SSH and Nginx configuration changes were validated before their services were reloaded.
- Separate keys are used for local server access and server-to-GitHub authentication.

### Further references

- [Ubuntu Server documentation](https://documentation.ubuntu.com/server/)
- [OpenSSH server documentation](https://documentation.ubuntu.com/server/how-to/security/openssh-server/)
- [GitHub SSH documentation](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [GitHub SSH key fingerprints](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints)
- [Nginx documentation](https://nginx.org/en/docs/)
