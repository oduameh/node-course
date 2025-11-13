---
sidebar_label: 'Logging, firewall, and monitoring'
sidebar_position: 5
---

# Logging, firewall, and monitoring

## A couple of daemons

Install two essential services on the server.

The first is **fail2ban**.

**Fail2ban** is a daemon that monitors and bans clients that are repeatedly failing authentication checks, such as brute-force SSH attempts.

Run this daemon on any publicly accessible server:

```
sudo apt install -y fail2ban
```

The second service is **chrony**.

**Chrony** is an implementation of the Network Time Protocol (NTP). **Cardano** is very time-dependent and requires that block-producing nodes, relays, and other nodes keep accurate time. This is especially critical for block-producing nodes, as the network expects blocks to be minted and propagated within a second (slot). With default settings, **chrony** regularly checks with publicly available NTP servers (in this case, `pool.ntp.org`). While many options can be configured in `/etc/chrony.conf`, such as specifying a lower stratum NTP server, the default settings are adequate for most purposes:

```
sudo apt install -y chrony
```

Verify **chrony** is keeping the server in sync:

```
chronyc tracking
```

![chrony](/img/chronyct.png)

## Firewall

Running an internet-accessible infrastructure requires security measures, and the firewall is one of the most important. Use **ufw** (uncomplicated firewall), which comes packaged with the Ubuntu Server Linux distribution. **Ufw** is a simple and user-friendly frontend for **iptables** that allows configuration of IP packet filtering rules for the Linux kernel firewall.

Set the default firewall rules.

Deny incoming traffic:

```
sudo ufw default deny incoming
```

Allow outgoing traffic:

```
sudo ufw default allow outgoing
```

Allow the **SSH** service to be reached on the server:

```
sudo ufw allow ssh
```

:::tip

In production, it is a best practice to change the port your SSH server uses to something higher and less frequently used (the default is port 22), as there are many scanners out there specifically targeting port 22, and many scanners start scanning at lower numbers. We are not going to do this today for a couple of reasons: time, these are not publicly accessible, and these servers are not critical infrastructure.

:::

:::danger

If you skip this step and enable the firewall, you will no longer be able to access your server!

:::


Make your `cardano-node` port available:

```
sudo ufw allow 1694/tcp
```

:::info

The `cardano-node` node-to-node (NtN) protocol was designed to facilitate two-way communication over a single outgoing `TCP/IP` connection. This means that we can connect with many other nodes even if we did not allow incoming connections on our specified port.

:::

Next, we need to open the port for the Prometheus node exporter (more on that in a minute):

```
sudo ufw allow 9100/tcp
```

Then, open the port to our `cardano-node` Prometheus exporter:

```
sudo ufw allow 12798/tcp
```

Finally, enable the firewall: 

```
sudo ufw enable
```

Reload the firewall:

```
sudo ufw reload
```

The SSH session should still be active.

View the firewall rules:

```
sudo ufw status
```

![ufwstat](/img/ufwstatus.png)

## Logging and monitoring

Enable logging and monitoring for the node.

`cardano-node` can produce **Prometheus** metrics by default.

:::info

**Prometheus** is an open-source monitoring system.

:::

Adjust the node configuration file to allow a **Prometheus** server to fetch data from the node:

```
nano /home/n(x)/preview/config/config.json
```

Find the `"hasPrometheus"` line and change the IP from localhost `127.0.0.1` to listening `0.0.0.0`:

```
  "hasPrometheus": [
    "0.0.0.0",
    12798
  ],
```

Before saving and exiting the file, make adjustments to enable logging.

Change the default logging location. This specifies where logs are written if no setup scribe is configured.

Find `"defaultScribes"` and change it from Stdout to FileSK with a path to the logs directory: 

```
  "defaultScribes": [
    [
      "FileSK",
      "/home/n(x)/preview/logs/cardano.json"
    ]
  ],
```

Specify the `"setupScribes"` output. Find the `"setupScribes"` line and modify it to match the following:

```
    "setupScribes": [
    {
      "scFormat": "ScJson",
      "scKind": "FileSK",
      "scName": "/home/n(x)/preview/logs/cardano.json",
      "scRotation": null
    }
  ]
```

:::tip

Be careful to preserve the `.json` formatting when adjusting these. It is easy to break the configuration file and prevent your node from starting by accidentally deleting a comma or causing some other syntax error.

:::

Save the `config.json` file with `ctrl + o` and exit with `ctrl + x`.

Restart the node to activate these changes:

```
sudo systemctl restart node.service
```

Verify the service is running:

```
sudo systemctl status node.service
```

Query the tip to confirm the node is active: 

```
cardano-cli query tip --testnet-magic 2
```

:::tip

If you are getting a 'socket not found' error when doing this, but your `node.service` is active, give the node a minute to start up until the socket file is created and try again.

:::

View the logs in real time with the tail command:

```
cd /home/n(x)/preview/logs/
tail -f cardano.json
```

This displays the live log output of `cardano-node` being written to the `cardano.json` file.

End the command with `ctrl + c`.

Install the **Prometheus** node exporter:

```
sudo apt install -y prometheus-node-exporter
```

This automatically starts the service, allowing the configured **Prometheus** server to collect the server as an endpoint. Node stats should now appear on the **Grafana** dashboard.


