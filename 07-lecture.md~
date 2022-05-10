---
label: Lecture 6
icon: file
author:
  name: Antonio Hernandez
  email: contacto@antoniohernandez.mx
order: -9
---

# 

[Lecture Videos :icon-link-external:](https://youtube.com/playlist?list=PLNEK_Ejlx3x2sBWXHdFBRgkzPF6N-1LVi)

Topics covered:

1. Introduction
2. The Minting Policy
3. Minting with the CLI
4. Deployment Scenarios
5. The Contracts
6. Minting with the PAB
7. Summary


## Setup

Started by copying the wallets constructed in *week03*.  That is to say,
copied `01.*` and `02.*` from directory `week03/testnet` to `week06/testnet`.

Executed `env.sh` to export the *node socket path* and other variables to the
shell.  Standing on directory `week06`, executed:

```console
. env.sh
```

Verified that `CARDANO_NODE-SOCKET_PATH` is set:

```console
echo $CARDANO_NODE_SOCKET_PATH
```

> `testnet/node.sock`

### Cardano Node

Since I am keeping the node database on an external hard drive, I updated the
symbolic link pointing to `testnet/db`.

On a separate `nix-shell`, standing on directory `week06`, started the Cardano
Node:

```console
./start-testnet-node.sh
```

To check the node is running properly, queried the UTxO's on wallet 1:

```console
./query-key1.sh
```

### Token policy

Copied the `TxHash` and took note of the `TxIx` of a UTxO with enough funds
from the previous query.  Say that the `TxHash` is `a8b588de9129` (it is
actually about five times longer) and the `TxIx` is `0`.  Then the following
code generates, as an example, the file `policy.plutus`:

```console
cabal exec token-policy -- policy.plutus a8b588de9129#0 123456 PPP
```

The file `policy.plututs` contains the **serialized version of the plutus
script** with the policy that generates 123456 tokens named 'PPP'.


## Minting with the CLI

The haskell executable `token-name` converts the token name to hexadecima.
In our case,

```console
cabal exec token-name -- PPP
```

outputs `505050`.  The script `mint-token-cli.sh` uses this.

To mint the tokens with the CLI, executed:

```console
./mint-token-cli.sh a8b588de9129#0 123456 PPP testnet/01.addr testnet/01.skey
```

where, again, you should substitute `a8b588de9129#0` with the actual reference
to the UTxO that is going to be consumed to run the minting policy.

Verified that the minting has occurred quering again wallet 1:

```console
./query-key1.sh
```

### Simulated minting

On the REPL, executed

```haskell
testToken
```

to trace the simulated minting of 100,000 "USDT"s.  This is achieved simply by
the plutus command `activateContractWallet`

> `activateContractWallet :: (...) => Wallet -> contract w s e () -> Eff effs (ContractHandle w s e)`

See the source file `Trace.hs`.


## Minting with the PAB

We now describe several steps involved in minting with the PAB.  For this to be possible, the following should be running (besides the Cardano Node):

- Wallet backend server
- Chain index
- PAB server

### Start the testnet wallet

To start the wallet backend server, executed on a separate `nix-shell`:

```console
./start-testnet-wallet.sh
```

We create a wallet with

```console
./create-wallet.sh MyWallet mysecretpassphrase wallet.json
```

This produces the file `wallet.json` with the recovery phrase.

Loaded the wallet with:

```console
cp wallet.json testnet/restore-wallet.json
./load-wallet.sh
```

From the output, copy the value in the field with key "id".  This is
the WALLETID, which should be pasted into `env.sh`.

Next, start the chain-index:

```console
./start-testnet-chain-index.sh
```

The first time I tried to run it, I got an error.  Asked for help in *Discord* and followed @kindofdev's suggestion.  Standing on directory `plutus-apps`, executed:

```console
cabal install plutus-pab-examples plutus-chain-index
```

which took several minutes and resulted in a very long output.  Went back to directory `week06` and tried to start the chain index again (from a separate nix-shell).  This time it worked.

However, building the chain index **takes several days**.

Next, migrated the PAB:

```console
./migrate-pab.sh
```

### Starting the PAB

Modified the file `pab-config.yml` as follows.  Near the end, find the
`developmentOptions`.  Below the commented "tag" field, one finds the fields
with key "pointBlockId" and "pointSlot".  The corresponding values should be
the latest BlockId and Slot in the shell running the Cardano-Node.  This
speeds the loading of the PAB significantly.

From a separate nix-shell, started the the PAB:

```console
./start-testnet-pab.sh
```

Next, obtained a list of wallet addresses with

```console
./get-address.sh
```

Copied the address ID of one address in the list and pasted into the file
`env.sh` in the corresponding ADDRESS value.  Then executed

```console
. env.sh
```

### Fund the wallet

To fund the wallet, created the file `sendNewWallet.sh` with the following
content:

```
#!/bin/bash
cardano-cli transaction build \
    --alonzo-era \
    $MAGIC \
    --change-address $(cat testnet/01.addr) \
    --tx-in [TxHash]#[TxIx] \
    --tx-out "$ADDRESS 20000000 lovelace" \
    --out-file testnet/txNW.body

cardano-cli transaction sign \
    --tx-body-file testnet/txNW.body \
    --signing-key-file testnet/01.skey \
    --testnet-magic 1097911063 \
    --out-file testnet/txNW.signed

cardano-cli transaction submit \
    --testnet-magic 1097911063 \
    --tx-file testnet/txNW.signed
```

where [TxHash] and [TxIx] are obtained from querying the UTxO in the funding wallet with

```console
cardano-cli query utxo --address $(cat testnet/01.addr) $MAGIC
```

Once this file is made executable (usiing `chmod`), we executed

```console
./sendNewWallet.sh
```

### Minting with cURL

Once the new wallet (the one with id $WALLETID) is funded, we can proceed with
the minting with:

```console
./mint-token-curl.sh 123456 PPP
```

### Minting with haskell

An alternative to using cURL, is to use haskell.  The following script
executes the haskell code:

```
./mint-token-haskell.sh 1000000 Gold
```

### End result

In both cases (cURL and Haskell) we did not get error messages.  However,
checking with Yoroi Nightly, we observed that we did not get any tokens.

Most likely this is due to the fact that the chain index never finished syncronizing.  It got to 86%, but then stalled.

It seems that running the chain-index is a very resource heavy process and my
laptop could not hold it.


## Some haskell

This week's lecture included some relatively rare haskell commands:

### Span

Example:

> `Prelude> span (<5) [1,3,5,7,9,11]`
>
> `([1,3],[5,7,9,11])`

Used in `Utils.hs` to define `unsafeReadTxOutRef`.

### Last

Defined in standard module `Data.Monoid`, `Last` is both a data constructor
and a command.  Its usage can be illustrated with the `[Char]` monoid.

Example:

> `Prelude> import Monoid`

> `Prelude> :t Last`
>
> Last :: Maybe a -> Last a

> `Prelude> Data.Monoid> mempty :: Last Char`
>
> `Last {getLast = Nothing}`

> `Prelude> Data.Monoid> Last (Just 'a')`
>
> `Last {getLast = Just 'a'}`

> `Prelude> Data.Monoid> Last (Just 'a') <> Last (Just 'b')`
>
> `Last {getLast = Just 'b'}`

> `Prelude> Data.Monoid> Last (Just 'a') <> Last (Just 'b') <> Last Nothing`
>
> `Last {getLast = Just 'b'}`

`Last` was used in the definition of command `monitor` in `Monitor.hs`.
