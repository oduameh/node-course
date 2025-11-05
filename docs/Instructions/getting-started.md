---
sidebar_label: 'Getting Started'
sidebar_position: 1
---

# Getting started

### Welcome to the Cardano operations course
![lab1](/img/lab1.jpeg)
## Connecting to your server via SSH

The first thing we need to do is connect to your server via secure shell (SSH). 

Mac OSX and Linux (most distros) have the SSH client already installed. If you have a Windows machine, you'll need to install [putty](https://www.putty.org/)

:::warning

We are not going to use a best practice for SSH today as we are relaxing security for ease of use for this lab exercise. In a production environment you will need to harden your SSH server connection methods. 

Here is a short list of ways to harden SSH: 
- Change default SSH port from 22 to a much higher number. Port 22 is commonly scanned and attacked via brute-force attacks, and scanners typically start and ascend when looking for opened ports.
- Disable password authentication and use generated SSH keys only.
- Configure firewall to only allow SSH connections from specific IPs that you own.
- Install `fail2ban` or similar to limit authentication attempts to your server.
- Configure a tunnel between devices using something like `wireguard` or `openvpn` between client and server.

:::

First, we need to connect to your device with SSH. 

For Mac OSX and Linux, use the following command. 

:::note

Please note that the user 'n' and IP addresses used here are for a preconfigured lab environment, and your IP address(es) will be different.

:::

```
ssh n(X)@10.42.0.X
```

For putty users, make sure port is `22`, protocol or connection type is `ssh` and that your IP is correct. Then `open` the connection. 

![putty1](/img/putty1.png)

:::tip

Replace the `X`, or whatever is in the last octet in `10.42.0.X` with the correct information on your credential card, as well as n`X`

::: 

## Time to get started

Once you have connected it is time to get started.

Before we get rolling, check for updates/upgrades to our server. (These should be relatively up-to-date, so this hopefully will not take too long.)

```
sudo apt update -y && sudo apt upgrade -y
```

Next, let's make a few working directories

```
cd
mkdir -p preview/{node,config,socket,test-db,scripts,bin,logs}
```

Now, let's add a few path items to our `.bashrc` file

Open your `.bashrc` file with a text editor

```
nano /home/n(x)/.bashrc
```
Then, note the following export lines and copy them at the end of the `.bashrc` file. Make sure to modify `n(X)` to your username when you do this.

```
export PATH="/home/n(X)/preview/bin:$PATH"
export CARDANO_NODE_SOCKET_PATH="/home/n(X)/preview/socket/node.socket"
```
Save with `ctrl + o` and exit the editor with `ctrl + x`


Source your bash file and then check your paths:

```
cd
. .bashrc
echo $PATH $CARDANO_NODE_SOCKET_PATH
```

You should see the `/home/n(X)/preview/bin:/home/n(X)/` and `/home/n(X)/preview/socket/node.socket` paths as part of the output




