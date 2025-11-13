---
sidebar_label: 'Keys and addresses'
sidebar_position: 6
---

# Keys and addresses

## Keys

Create payment keys, staking keys, and addresses, then submit a simple transaction.

### Payment keys

Create directories to store keys and addresses:

:::danger

We are not following best practices here. If you are creating payment or other key pairs that require security on mainnet or that interact with real funds, you **MUST** store the secret keys offline. 

:::

```
mkdir -p  ~/preview/{wallet1,wallet2}
cd ~/preview/wallet1
```

Create the payment key pair for wallet1. This set of keys allows for the holding and transfer of ada and other native assets:

```
cardano-cli address key-gen \
--verification-key-file ~/preview/wallet1/payment.vkey \
--signing-key-file ~/preview/wallet1/payment.skey
```

### Staking keys

Create a staking key pair for wallet1. An address can be generated using **both** payment and staking keys, which enables staking on the Cardano blockchain. An address derived from **only** payment keys is considered an *enterprise* address, which cannot participate in staking:

```
cardano-cli conway stake-address key-gen \
--verification-key-file ~/preview/wallet1/stake.vkey \
--signing-key-file ~/preview/wallet1/stake.skey
```

### Address

Build an address using the verification (public) keys of the payment and stake key pairs created for wallet1:

```
cardano-cli address build \
--payment-verification-key-file ~/preview/wallet1/payment.vkey \
--stake-verification-key-file stake.vkey \
--testnet-magic 2 \
--out-file ~/preview/wallet1/payment.addr
```

Display the payment address:

```
cat ~/preview/wallet1/payment.addr
```

![paymentaddr](/img/paymentaddrw1.png)


:::info

Once you have generated your address for Wallet1, please use the [faucet](https://docs.cardano.org/cardano-testnets/tools/faucet) to request some test ada to play around with. 

:::

### Wallet 2

Create a second wallet, wallet2:

```
cd ~/preview/wallet2
```

Create a simple enterprise address without the ability to participate in staking.

Create the payment key pair:

```
cardano-cli address key-gen \
--verification-key-file ~/preview/wallet2/payment.vkey \
--signing-key-file ~/preview/wallet2/payment.skey
```

### Enterprise address

Build the address using the verification key file:

```
cardano-cli address build \
--payment-verification-key-file ~/preview/wallet2/payment.vkey \
--out-file ~/preview/wallet2/payment.addr \
--testnet-magic 2
```

After requesting test ada from the faucet (for wallet1), query the UTXOs belonging to the payment address:

```
cardano-cli query utxo --address $(cat ~/preview/wallet1/payment.addr) --testnet-magic 2
```

:::tip

This query may take a moment to complete.

:::

The output will display the TxHash, TxIx value, and the amount of ada contained in the UTXO (in lovelaces): 

![utxo1](/img/utxo1.png)

:::note

1,000,000 lovelaces equals 1 ada. When using `cardano-cli`, always input values in lovelaces.

:::

Confirm the wallet has a valid UTXO with an amount of ada, then craft and submit a simple transaction using `cardano-cli`.

Create a directory for transaction files:

```
mkdir ~/preview/tx
cd ~/preview/tx
```

Send a small amount of ada from wallet1 to wallet2.

Build the transaction:

:::tip

Replace the placeholder UTXO hash and TxIx with the actual UTXO information obtained from the `query utxo` command.

:::

```
cardano-cli conway transaction build \
--socket-path /home/n(x)/preview/socket/node.socket \
--testnet-magic 2 \
--tx-in <the utxo hash you want to consume>#<the txix of the utxo> \
--change-address $(cat /home/n(x)/preview/wallet1/payment.addr) \
--tx-out $(cat /home/n(x)/preview/wallet2/payment.addr)+10000000 \
--out-file ~/preview/tx/tx1.raw
```

The estimated fee in lovelace for the transaction will be displayed:

![fee1](/img/estfee1.png)

Sign the transaction with the wallet1 `payment.skey`:

```
cardano-cli conway transaction sign --tx-body-file ~/preview/tx/tx1.raw \
--signing-key-file ~/preview/wallet1/payment.skey \
--out-file ~/preview/tx/tx1.signed
```

Submit the signed transaction:

```
cardano-cli conway transaction submit --tx-file ~/preview/tx/tx1.signed \
--socket-path /home/n(x)/preview/socket/node.socket \
--testnet-magic 2
```

A success notification should be displayed: 

![success](/img/txsub1.png)


Verify the UTXOs in wallet1 and wallet2:

```
cardano-cli query utxo --address $(cat ~/preview/wallet1/payment.addr) --testnet-magic 2
```

![utx1](/img/w1utxo1.png)

For wallet2:

```
cardano-cli query utxo --address $(cat ~/preview/wallet2/payment.addr) --testnet-magic 2
```

![utx2](/img/w2utxo1.png)

:::info

This transaction demonstrates the simplest use of `cardano-cli`.

Extra credit exercises:
- Create wallet3
- Send 10 ada each to wallet3 and wallet2 from wallet1, consuming a single UTXO
- Send 10 ada each from wallet1 and from wallet2 to wallet3 (20 ada total) in a single transaction
- Create a transaction using `build-raw` instead of `build` (manually calculate the fee, lovelaces, change, TTL, etc).

If you want to try to build a transaction with `build-raw`, follow the instructions [here](https://developers.cardano.org/docs/operate-a-stake-pool/register-stake-address).
