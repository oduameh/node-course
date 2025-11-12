---
sidebar_label: 'Automating Things'
sidebar_position: 4
---

# Automating the Node

:::info

This section covers RTS options for `cardano-node` before automating the node.

:::

**RTS** stands for runtime system—a Haskell software layer that allows customization of the following: 
- Memory management
- Garbage collection
- Concurrency and parallelism
- Exception handling.

RTS options are a Haskell-specific feature that `cardano-node` takes specific advantage of.

These options can be implemented:
- During the compilation of the node
- As an override flag while running the node.

Check the RTS options baked into the static binary `cardano-node`:

```
cardano-node +RTS --info
```

The script below overrides an RTS option to increase the threads `cardano-node` consumes from 2 to 4.

:::warning

Warning! Only set the RTS flag value equal to the number of CPU cores on the server you are configuring. If you are unsure, just leave it at 2.

:::


## Building the script

Create a start-up script to automate the running of the node:

```
nano /home/n(x)/preview/scripts/node.sh
```

Add the following to the `node.sh` shell script: 

```
#!/bin/bash
#
# Set a few variables to shorten the launch command
# Database path
DB=/home/n(x)/preview/db
# Socket path
SOCKET=/home/n(x)/preview/socket/node.socket
# Configuration file
CONFIG=/home/n(x)/preview/config/config.json
# Topology file
TOPOLOGY=/home/n(x)/preview/config/topology.json
# Host address - set to listen for incoming connections
HOST=0.0.0.0
# Change <your port> to the port for either the block-producing node or relay
PORT=1694
#
# Command to run the node
#
/home/n(x)/preview/bin/cardano-node run --topology $TOPOLOGY --database-path $DB --socket-path $SOCKET --port $PORT --config $CONFIG --host-addr $HOST +RTS -N4
```

Save with `ctrl + o` and exit with `ctrl + x`.

The variables set within this bash script match what is input when starting the node on the command line, except for the RTS option added at the end of the command to run the node. 

Make the script executable:

```
chmod +x /home/n(x)/preview/scripts/node.sh
```

Test the script:

```
cd /home/n(x)/preview/scripts

./node.sh
```

The output should look similar to the following:

![scrptop](/img/testnodescript.png)

Kill the process with `ctrl + c`.

## Creating the systemd service

Create the service file in the scripts directory:

```
cd /home/n(x)/preview/scripts

nano node.service
```

Add the following to the service file (this is a basic configuration that can be modified based on specific needs): 

```
[Unit]
Description       = Cardano Node
Wants             = network-online.target
After             = network-online.target  
  
[Service]
User              = n(x)
Type              = simple
WorkingDirectory  = /home/n(x)/preview
ExecStart         = /bin/bash -c '/home/n(x)/preview/scripts/node.sh'
KillSignal        = SIGINT
RestartKillSignal = SIGINT
LimitNOFILE       = 24000
Restart           = always
RestartSec        = 7
SyslogIdentifier  = cardano-node
  
 [Install]
WantedBy          = multi-user.target
```

Save the file with `ctrl + o` and exit with `ctrl + x`.

Copy the service file to the systemd directory:

```
sudo cp /home/n(x)/preview/scripts/node.service /etc/systemd/system/
```

Ensure the service file has the appropriate permissions:

```
sudo chmod 0644 /etc/systemd/system/node.service
```

Reload the daemon so the system recognizes the new service file:

```
sudo systemctl daemon-reload
```

:::tip

Reload the systemd daemon whenever a service file is changed.

:::

Start the node service:

```
sudo systemctl start node.service
```

Verify the service started successfully:

```
sudo systemctl status node.service
```

The output should show the service as `active`:

![active](/img/nodeserviceactive.png)

If the service is active, enable it to run automatically on system start-up:

```
sudo systemctl enable node.service
```

A symlink will be created for the service:

![symlink](/img/enabledsymlink.png)

The systemd service will now ensure that `cardano-node` runs in the background as specified.

Query the tip of the chain to verify synchronization:

```
cardano-cli query tip --testnet-magic 2
```

Confirm the node is in sync:

![sync](/img/querytipinsync1.png)


