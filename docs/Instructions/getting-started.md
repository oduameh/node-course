---
sidebar_label: 'Getting Started'
sidebar_position: 1
---

# Getting started

### Welcome to the Cardano operations course
![lab1](/img/lab1.jpeg)
## Connecting to your server via SSH

Connect to the server via Secure Shell (SSH).

Mac OS X and Linux (most distributions) have the SSH client already installed. Windows users need to install [PuTTY](https://www.putty.org/).

:::warning

We are not going to use a best practice for SSH today as we are relaxing security for ease of use for this lab exercise. In a production environment you will need to harden your SSH server connection methods. 

Here is a short list of ways to harden SSH: 
- Change default SSH port from 22 to a much higher number. Port 22 is commonly scanned and attacked via brute-force attacks, and scanners typically start and ascend when looking for opened ports.
- Disable password authentication and use generated SSH keys only.
- Configure firewall to only allow SSH connections from specific IPs that you own.
- Install `fail2ban` or similar to limit authentication attempts to your server.
- Configure a tunnel between devices using something like `wireguard` or `openvpn` between client and server.

:::

Connect to the device with SSH.

For Mac OS X and Linux, use the following command: 

:::note

Please note that the user 'n' and IP addresses used here are for a preconfigured lab environment, and your IP address(es) will be different.

:::

```
ssh n(X)@10.42.0.X
```

For PuTTY users, ensure the port is `22`, protocol or connection type is `SSH`, and the IP address is correct. Then open the connection: 

![putty1](/img/putty1.png)

:::tip

Replace the `X`, or whatever is in the last octet in `10.42.0.X` with the correct information on your credential card, as well as n`X`

::: 

## Time to get started

Check for updates and upgrades to the server:

```
sudo apt update -y && sudo apt upgrade -y
```

Create working directories:

```
cd
mkdir -p preview/{node,config,socket,test-db,scripts,bin,logs}
```

Add path items to the `.bashrc` file.

Open the `.bashrc` file with a text editor:

```
nano /home/n(x)/.bashrc
```
Add the following export lines to the end of the `.bashrc` file (modify `n(X)` to match the username):

```
export PATH="/home/n(X)/preview/bin:$PATH"
export CARDANO_NODE_SOCKET_PATH="/home/n(X)/preview/socket/node.socket"
```

Save with `ctrl + o` and exit the editor with `ctrl + x`.

Source the bash file and verify the paths:

```
cd
. .bashrc
echo $PATH $CARDANO_NODE_SOCKET_PATH
```

The output should include `/home/n(X)/preview/bin:/home/n(X)/` and `/home/n(X)/preview/socket/node.socket` as part of the paths.




