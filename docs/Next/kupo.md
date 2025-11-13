---
sidebar_label: 'Kupo'
sudebar_position: 8
---

# Kupo

![kupo1](/img/kupopic1.png)

[Kupo](https://github.com/CardanoSolutions/kupo) is a fast and lightweight chain index for the Cardano blockchain. It is like a lightweight version of [db-sync](https://github.com/IntersectMBO/cardano-db-sync). Chain indexers are beneficial to developers and service providers because they process and organize raw blockchain data to meet developer needs. 

Similar to `ogmios`, download a precompiled static `kupo` binary from the server:

```
cd /tmp

wget https://github.com/CardanoSolutions/kupo/releases/download/v2.11/kupo-v2.11.0-aarch64-linux.zip
```
Unzip it:

```
unzip kupo-v2.11.0-aarch64-linux.zip
```

Copy it to your bin directory:

```
cp /tmp/bin/kupo /home/n(x)/preview/bin/
```
Ensure you have it in the right place:

```
which kupo
```
```
kupo --version
```
![kupover](/img/kupover1.png)

Create a startup script:

```
nano /home/n(x)/preview/scripts/kupo_start.sh
```

Copy the following into the `kupo_start.sh` file:

```
#!/bin/bash

# Kupo configuration:
# - Using local Ogmios instance on port 1337
# - Matching all patterns with "*"
# - Starting from a specific block
# - Deferring DB indexes for faster startup
# - Working directory in the user's home
# - Listening on all interfaces on port 1442

/home/n(x)/preview/bin/kupo \
--ogmios-port 1337 \
--match "*" \
--since "54412148.656e903c510441032ad566d26fecf7be98882bdefde6a1ed961cd8e7bfa9d18a" \
--defer-db-indexes \
--workdir /home/n(x)/preview/kupo-db \
--ogmios-host 127.0.0.1 \
--host 0.0.0.0 \
--port 1442
```

Save and exit using `ctrl + o` and `ctrl + x`.

Make it executable and start it:

```
chmod +x /home/n(x)/preview/scripts/kupo_start.sh

cd /home/n(x)/preview/scripts/

./kupo_start.sh
```

You should see the output of `kupo` syncing:

![kuposy](/img/indexing1.png)

Exit the process using `ctrl + c`.

Now, create another service file in the `scripts` directory:

```
nano /home/n(x)/preview/scripts/kupo.service
```

Copy the following into your service file: 

```
[Unit]
Description       = Kupo
Wants             = network-online.target
After             = network-online.target  
  
[Service]
User              = n(x)
Type              = simple
WorkingDirectory  = /home/n(x)/preview
ExecStart         = /bin/bash -c '/home/n(x)/preview/scripts/kupo_start.sh'
KillSignal        = SIGINT
RestartKillSignal = SIGINT
LimitNOFILE       = 24000
Restart           = always
RestartSec        = 7
SyslogIdentifier  = kupo
  
[Install]
WantedBy          = multi-user.target
```

Copy the service file to your system directory: 

```
sudo cp /home/n(x)preview/scripts/kupo.service /etc/systemd/system/
```

Provide it with the correct permissions: 

```
sudo chmod 0644 /etc/systemd/system/kupo.service
```

Reload daemons:

```
sudo systemctl daemon-reload
```

Start the service: 

```
sudo systemctl start kupo.service
```

Check the status: 

```
sudo systemctl status kupo.service
```

It should say 'active'.

If active, enable the service:

```
sudo systemctl enable kupo.service
```

Now allow connections to port `1442` for queries: 

```
sudo ufw allow 1442/tcp
```

Reload: 

```
sudo ufw reload
```

Now check if everything is working. 

From a local terminal session on your machine (not logged into the server), use `curl` to request health metrics from your `kupo` endpoint:

```
curl "http://yourserverip:1442/health"
```

If everything is correctly configured, you will see output similar to this:

![kupohealth](/img/kupohealth.png)

You can find Kupo documentation [here](https://cardanosolutions.github.io/kupo/#section/Overview). Explore what other endpoints and data you can retrieve from your chain indexer. 
