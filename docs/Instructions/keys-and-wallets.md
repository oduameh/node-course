---
sidebar_label: 'Keys and Addresses'
sidebar_position: 6
---

# Keys and Addresses

## Keys

We will now create payment keys, staking keys, and addresses, and submit a simple transaction. 

### Payment keys

Start by creating a couple of directories to house your keys and addresses:

:::danger

We are not following best practices here. If you are creating payment or other key pairs that require security on mainnet or that interact with real funds, you **MUST** store the secret keys in an offline manner. 

:::

```
mkdir -p  ~/preview/{wallet1,wallet2}
cd ~/preview/wallet1
```

The first key pair we will create is the payment key pair for our 'wallet1'. This set of keys allows for the holding and transfer of ada and other native assets: 

```
cardano-cli address key-gen \
--verification-key-file ~/preview/wallet1/payment.vkey \
--signing-key-file ~/preview/wallet1/payment.skey
```

### Staking keys

The next pair we will create is a staking key pair for wallet1. This allows us to generate an address using **both** payment and staking keys, which enables us to stake on the Cardano blockchain. We can derive an address from **just** the payment keys, and that would be considered an *enterprise* address, which cannot participate in staking.

```
cardano-cli conway stake-address key-gen \
--verification-key-file ~/preview/wallet1/stake.vkey \
--signing-key-file ~/preview/wallet1/stake.skey
```

### Address

Next we are going to build an address using the verification (public) keys of the payment and stake key pairs we just created for wallet1. 

```
cardano-cli address build \
--payment-verification-key-file ~/preview/wallet1/payment.vkey \
--stake-verification-key-file stake.vkey \
--testnet-magic 2 \
--out-file ~/preview/wallet1/payment.addr
```

Once this is done, concatenate the payment.addr file. 

```
cat ~/preview/wallet1/payment.addr
```

![paymentaddr](/img/paymentaddrw1.png)


:::info

Once you have generated your address for Wallet1, please use the [Faucet](https://docs.cardano.org/cardano-testnets/tools/faucet) to request some test ADA to play around with. 

:::

### Wallet 2

Move on to the next step and create your second 'wallet', wallet2:

```
cd ~/preview/wallet2
```

This time, we are going to create a simple enterprise address without the ability to participate in staking. 

First, create the payment key pair again:

```
cardano-cli address key-gen \
--verification-key-file ~/preview/wallet2/payment.vkey \
--signing-key-file ~/preview/wallet2/payment.skey
```

### Enterprise address

From here, simply build the address using the verification key file:

```
cardano-cli address build \
--payment-verification-key-file ~/preview/wallet2/payment.vkey \
--out-file ~/preview/wallet2/payment.addr \
--testnet-magic 2
```

Once you have requested test ada from the faucet (in wallet1), query the UTXOs belonging to your payment address: 

```
cardano-cli query utxo --address $(cat ~/preview/wallet1/payment.addr) --testnet-magic 2
```

:::tip

This may take a second, don't panic.

:::

You should see an output containing the TxHash, TxIx value, and the amount of ada contained on the UTxO in lovelaces. 

![utxo1](/img/utxo1.png)

:::note

1000000 lovelaces equals 1 ada. When using `cardano-cli` we will always input values in lovelaces

:::


Once you confirm that your wallet has a valid UTxO with an amount of ada associated with it, we will craft and submit a simple transaction using `cardano-cli`.

First, create a directory to house your transaction files: 

```
mkdir ~/preview/tx
cd ~/preview/tx
```
For this first transaction, we are going to send a small amount of ada from 'wallet1' to 'wallet2'.

Build the transaction

:::tip

Please ensure you replace the carroted utxo hash text and the txix text with your actual utxo information obtained form the `query utxo` command we did earlier.

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

You should see at the bottom of the command an Estimated fee in lovelace for the transaction. 

![fee1](/img/estfee1.png)

Once we have the transaction file built (`tx1.raw`), sign the transaction with the wallet1 `payment.skey`:

```
cardano-cli conway transaction sign --tx-body-file ~/preview/tx/tx1.raw \
--signing-key-file ~/preview/wallet1/payment.skey \
--out-file ~/preview/tx/tx1.signed
```

Once we have signed the transaction, it is time to submit it:

```
cardano-cli conway transaction submit --tx-file ~/preview/tx/tx1.signed \
--socket-path /home/n(x)/preview/socket/node.socket \
--testnet-magic 2
```

If all was done correctly, you will be greeted with a success notification. 

![success](/img/txsub1.png)


Now check the addresses of wallet1 and wallet2 to see what UTXOs they now contain: 

```
cardano-cli query utxo --address $(cat ~/preview/wallet1/payment.addr) --testnet-magic 2
```
![utx1](/img/w1utxo1.png)

and for wallet2

```
cardano-cli query utxo --address $(cat ~/preview/wallet2/payment.addr) --testnet-magic 2
```

![utx2](/img/w2utxo1.png)

What do you notice? 

:::info

At this point, we have created a transaction with `cardano-cli` in the simplest possible way. However, we can add to what we have just done. 

Extra credit exercises: 
- Create a wallet3
- Send 10 ada each to wallet3 and wallet2 from wallet1, consuming a single UTXO
- Send 10 ada each from wallet1 and from wallet2 to wallet3 (20 ada total) in a single transaction
- Create a transaction using `build-raw` instead of `build` (you will need to manually calculate the fee, lovelaces, change, ttl, etc).

If you want to try to build a tx with `build-raw` follow the instructions [here](https://developers.cardano.org/docs/operate-a-stake-pool/register-stake-address).
