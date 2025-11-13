---
sidebar_label: 'Catching up'
sidebar_position: 3
---

# Mithril

[Mithril](https://mithril.network/doc/) is a stake-based multi-signature protocol for efficiency and scalability. It enables the secure aggregation of cryptographic signatures (in this case, Cardano stake pool operators (SPOs) running Mithril signers with agreement on an aggregator). All SPOs running Mithril signers sign and verify regular snapshots. The DB snapshots themselves are not hosted on-chain (that would be unwise), but rather hosted on a fast cloud provider (Google, in this case). The security and purpose come from the cryptographic signatures that verify the snapshots.

The more stake involved in signing snapshots, the more secure the Mithril protocol is.

Download these signed snapshots using `mithril-client-cli`. Compile the binaries for this client.

This is a relatively quick process (especially compared to compiling the robust `cardano-node` in Haskell, since Rust crates can be compiled in parallel).

Install the Rust toolchain, as Mithril is built with Rust:

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Select the default options by pressing 'Enter'.

Source the cargo environment directory:

```
. "$HOME/.cargo/env"
```

Install dependencies:

```
sudo apt-get install -y libssl-dev make build-essential m4 pkg-config unzip
```

Create a directory, clone the repository, and check out the appropriate version for `mithril-client-cli`: 

```
mkdir /home/n(x)/mithril/
cd /home/n(x)/mithril/

git clone https://github.com/input-output-hk/mithril.git

cd /home/n(x)/mithril/mithril/mithril-client-cli

git fetch --tags --all

git checkout 2524.0
```

Build the client:

```
make build
```

:::note

This build process will take a few minutes.

:::

:::tip

Compiling can be quite system-intensive. To monitor system performance, open another SSH session and run the `htop` or `btop` commands to see system resource usage. ![workwork](/img/workingharthtop.png)

To install `btop`, open a new SSH session and run:

```
sudo apt install -y btop
```

![worktop](/img/btop.png)
:::

Copy the compiled binary to the directory within the path:

```
cp /home/n(x)/mithril/mithril/mithril-client-cli/mithril-client /home/n(x)/preview/bin/
```

Verify the `mithril-client` version:

```
mithril-client --version
```

The output should look similar to this:

![clientv](/img/mithrilclient1.png)

Download a snapshot to sync the node with the chain.

Set variables specific to the aggregator and network. These variables only need to be created for this terminal/SSH session.

Set the network variable:

```
export CARDANO_NETWORK=preview
```

Set the aggregator endpoint:

```
export AGGREGATOR_ENDPOINT=https://aggregator.pre-release-preview.api.mithril.network/aggregator
```

Set the genesis verification key:

```
export GENESIS_VERIFICATION_KEY=$(wget -q -O - https://raw.githubusercontent.com/input-output-hk/mithril/main/mithril-infra/configuration/pre-release-preview/genesis.vkey)
```

Set the snapshot digest:

```
export SNAPSHOT_DIGEST=latest
```

View available snapshots: 

```
mithril-client cardano-db snapshot list
```

A list of available snapshots will be displayed (the list will differ based on when this command is run):

![snapshotlist](/img/snapshotlist2.png)

View the details of a specific snapshot. For the most up-to-date snapshot, use the most recent digest from the list:

```
mithril-client cardano-db snapshot show 2c0894e86576aa702f5949044b0c6ff04e047333191c72ed34ccdea0ef87515a
```

This command displays the digest for the specified snapshot, providing critical information such as the node version:

![digest](/img/digest21133.png)

:::note

Ensure you are in the desired working directory, as the snapshots are considerable in size.

:::

Change to the preview directory:

```
cd /home/n(x)/preview
```

Download the snapshot (use the appropriate digest hash for the selected snapshot):

```
mithril-client cardano-db download e640efdf122c07bd753cac8d2fefdd95be994980494a9553f918e5cba35bfefd
```

The download progress should be displayed:

![snapdl](/img/mithrildownload1.png)

:::note

This download will take a few minutes.

:::

Run the node after the snapshot download completes: 

```
cardano-node run --topology ~/preview/config/topology.json \
--database-path ~/preview/db \
--socket-path ~/preview/socket/node.socket \
--port 1694 \
--config ~/preview/config/config.json
```

The node start-up output should be visible in the current session.

:::note

If 'Replayed Block' messages appear, this is a normal security function of the Haskell node, where it performs a full replay on the database to re-process the block history of the network:

![block-replay](/img/replayed-block.png)

Let this process complete before proceeding. This does not take as long on the testnet since the ledger is much smaller compared to mainnet.

:::

Check the syncing status in another session:

```
watch -n 1 cardano-cli query tip --testnet-magic 2
```

The node may take a couple of minutes to start and sync:

:::info

If a 'socket not found' error appears, it means the `cardano-cli` cannot find the socket to communicate with the node. The socket file has not been created yet as part of the node start-up process.

:::

![syncprog](/img/querytipinsync1.png)

Stop the node with `ctrl + c`. The next time the node starts, it will resume from where it left off.



