---
sidebar_label: 'Running the Node'
sidebar_position: 2
---

# Running the Cardano node

### Acquire the node

There are several options for acquiring the `cardano-node` and `cardano-cli` binaries:

- Intersect MBO offers pre-compiled static binaries on their cardano-node [releases page](https://github.com/IntersectMBO/cardano-node/releases).
- Static or dynamic binaries may also be built from [source](https://github.com/IntersectMBO/cardano-node).

The pre-compiled static binaries will not work on Raspberry Pis, as they run on ARM architecture (aarch64).

Compiling `cardano-node` and `cardano-cli` from source on Raspberry Pi 5 single-board computers is time-intensive.

Dynamically compiled binaries require specific libraries (libsodium, secp256k1, and blst, each of which needs to be compiled separately).

For this workshop, use the statically compiled binaries for aarch64 provided by the [Armada Alliance](https://armada-alliance.com/), specifically the efforts of ZW3RK pool. These should run on most distributions of Linux, as the dependent libraries are part of the compiled binary.

Download the statically compiled `cardano-node` and `cardano-cli` binaries and copy them to the directory added to the path (`/home/n(X)/preview/bin/`):

```
cd /tmp
wget -c https://github.com/armada-alliance/cardano-node-binaries/blob/main/static-binaries/cardano-10_4_1-aarch64-static-musl-ghc_9101.tar.zst?raw=true -O cardano-binaries.tar.zst
```

:::note

If you are using amd64 CPU architecture, make sure and grab the right binary from the [releases](https://github.com/intersectmbo/cardano-node/releases) page rather than the Armada Alliance arm binary in the instructions.

:::


Extract `cardano-node` and `cardano-cli` from the download:

```
tar -I zstd -xvf cardano-binaries.tar.zst --wildcards '*cardano-node' '*cardano-cli'
```

Copy `cardano-node` and `cardano-cli` to the bin directory:

```
cd cardano-10_4_1-aarch64-static-musl-ghc_9101
```

```
cp cardano-node /home/n(x)/preview/bin/
cp cardano-cli /home/n(x)/preview/bin/
```

Remove the archive:

```
cd /tmp
rm cardano-binaries.tar.zst
```

Verify the versions:

```
cardano-cli --version
```

The output should be:

![cli](/img/ccli.png)

```
cardano-node --version
```

The output should be:

![node1](/img/cnode.png)


### Running the node

To run, `cardano-node` requires configuration files and start-up flags.

The following files are required to run as a basic node or relay (non-block-producing):
- **Main configuration file**: contains node settings and points to the **Shelley**, **Byron**, **Alonzo**, and **Conway** Genesis files.
- **Byron Genesis**: contains initial protocol parameters and instructs `cardano-node` on how to bootstrap the Byron Era of Cardano.
- **Shelley Genesis**: contains initial protocol parameters and instructs `cardano-node` on how to bootstrap the Shelley Era of Cardano.
- **Alonzo Genesis**: contains initial protocol parameters and instructs `cardano-node` on how to bootstrap the Alonzo Era of Cardano.
- **Conway Genesis**: contains initial protocol parameters and instructs `cardano-node` on how to bootstrap the Conway Era of Cardano.
- **Topology file**: contains a list of bootstrap, local, and public peers (peers are other nodes running Cardano).

Download these files:

```
cd /home/n(x)/preview/config

wget https://book.world.dev.cardano.org/environments/preview/config.json
wget https://book.world.dev.cardano.org/environments/preview/topology.json
wget https://book.world.dev.cardano.org/environments/preview/byron-genesis.json
wget https://book.world.dev.cardano.org/environments/preview/shelley-genesis.json
wget https://book.world.dev.cardano.org/environments/preview/alonzo-genesis.json
wget https://book.world.dev.cardano.org/environments/preview/conway-genesis.json
wget https://book.world.dev.cardano.org/environments/preview/peer-snapshot.json
```

Ensure the topology file contains the full path to the `peer-snapshot.json` file.

Edit the `topology.json` file:

```
nano /home/n(x)/preview/config/topology.json
```

Save with `ctrl + o` and exit with `ctrl + x`.

Add the full path to the `peer-snapshot.json` file (adjust the username from n19 to match the actual username):

![peersnap](/img/peer-snap.png)

Run the node: 

```
cardano-node run --topology ~/preview/config/topology.json \
--database-path ~/preview/test-db \
--socket-path ~/preview/socket/node.socket \
--port 1694 \
--config ~/preview/config/config.json
```

The node start-up output will be visible in the terminal window:

![nodestartup](/img/nodestartuptest1.png)

Open an additional SSH session to the Raspberry Pi server while the node runs.

Query the tip of the chain to observe the syncing progress:

```
cardano-cli query tip --testnet-magic 2
```

:::note

Observe active syncing using the watch command:

```
watch -n 1 cardano-cli query tip --testnet-magic 2
```
:::

:::tip

The `--testnet-magic` flag specifies different testnets. For example, pre-production uses `--testnet-magic 1`, while mainnet uses `--mainnet`.

:::

Syncing from scratch is time-intensive and would take considerable time to complete.

Stop the node by pressing `ctrl + c` in the session running the node.

Remove the test database:

```
rm -r /home/n(x)/preview/test-db
```
