---
label: Lecture 3
icon: file
author:
  name: Antonio Hernandez
  email: contacto@antoniohernandez.mx
order: -6
---


<a id="orgc445ba9"></a>

# Vesting example

[Lecture Videos :icon-link-external:](https://www.youtube.com/playlist?list=PLNEK_Ejlx3x2zxcfoVGARFExzOHwXFCCL)

Topics covered:

1.  Configuring playground time out
2.  Script contexts
3.  Handling time
4.  A vesting example
5.  Parametrized contracts
6.  Deploying to the Cardano testnet
7.  Homework
8.  Summary


<a id="org278cfb6"></a>

## Installing the Cardano components on **MacOS Catalina**

Before deploying on the Cardano testnet, I had to build the `cardano-node` and
`cardano-cli` (the Cardano Node and Cardano Command Line interface).

The repository of the cardano node components sits at 
<https://github.com/input-output-hk/cardano-node> .

The following steps worked for me (based and modified from the *Installing
cardano-node* [instructions at developers.cardano.org](https://developers.cardano.org/docs/get-started/installing-cardano-node/)).


<a id="org4df0f41"></a>

### XCode

1 - [Downloaded](https://developer.apple.com/xcode/) Xcode and the Xcode
Command Line Tools.  Since my computer runs MacOS Catalina, I downloaded
version 12.4.  [Homebrew](https://brew.sh/) needs to also be installed.


<a id="org097645b"></a>

### Installing Homebrew packages

2 - Installed some libraries with `brew`:

    brew install jq
    brew install libtool
    brew install autoconf
    brew install automake
    brew install pkg-config

!!!
My Mac runs on an Intel chip.  If yours uses **M1**, refer to the
[instructions at developers.cardano.org](https://developers.cardano.org/docs/get-started/installing-cardano-node/)
for further steps.
!!!


<a id="org4d06460"></a>

### Downloading & Compiling

3 - Created a working directory to store the source-code and build the
components.

    mkdir -p $HOME/cardano-src
    cd $HOME/cardano-src

4 - Install `libsodium`

Next, I downloaded, compiled and installed `libsodium`:

    git clone https://github.com/input-output-hk/libsodium
    cd libsodium
    git checkout 66f017f1
    ./autogen.sh
    ./configure
    make
    sudo make install

5 - I added the following lines to my bash shell profile `$Home/.bashrc`:

    export LD_LIBRARY_PATH="/usr/local/lib:$LD_LIBRARY_PATH"
    export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH"

Reloaded the shell profile with `source $Home/.bashrc`.

6 - Install the **Cardano** components

Next, I downloaded, compiled and installed `cardano-node` and `cardano-cli`:

    cd $Home/cardano-src

Downloaded the `cardano-node` respository:

    git clone https://github.com/input-output-hk/cardano-node.git
    cd cardano-node
    git fetch --all --recurse-submodules --tagsgit clone https://github.com/input-output-hk/cardano-node.git
    cd cardano-node
    git fetch --all --recurse-submodules --tags

Switched the repository to the latest tagged commit:

    git checkout $(curl -s https://api.github.com/repos/input-output-hk/cardano-node/releases/latest | jq -r .tag_name)

Built the node

    cabal build all

and installed the newly built node and CLI to `$Home/.local/bin`

    mkdir -p $HOME/.local/bin
    cp -p "$(./scripts/bin-path.sh cardano-node)" $HOME/.local/bin/
    cp -p "$(./scripts/bin-path.sh cardano-cli)" $HOME/.local/bin/

Added the following line to the shell profile `$Home/.bashrc`

    export PATH="$HOME/.local/bin/:$PATH"

and reloaded with `source $Home/.bashrc`.

To verfiy all is OK, checked versions installed:

    cardano-cli --version
    cardano-node --version


<a id="org8825f91"></a>

## Running cardano-node (with external hard drive)

Set the current directory to `plutus-pioneer-program/code/week03/testnet`.  It
contains the [configuration files](https://hydra.iohk.io/build/8111119/download/1/index.html) for the
**testnet**:

-   `testnet-config.json`
-   `testnet-byron-genesis.json`
-   `testnet-shelley-genesis.json`
-   `testnet-alonzo-genesis.json`
-   `testnet-topology.json`

The script `start-node-testnet.sh` runs `cardano-node` with the following
configuration:

    cardano-node run \
     --topology testnet-topology.json \
     --database-path db \
     --socket-path node.socket \
     --host-addr 127.0.0.1 \
     --port 3001 \
     --config testnet-config.json

The `database-path` sets the name of the database directory.  For me it was
desirable to have the database on an **external hard drive**.  This is
achieved simply by setting a symbolic link.  Assume we want the database to
sit inside directory `cardano` at the root of an external hard drive named
*External-Hard-Drive*.  Remembering that the active directory on our shell is
still `testnet`, we execute

    mkdir -p /Volume/External-Hard-Drive/cardano/db
    ln -s /Volume/External-Hard-Drive/cardano/db .

After the symbolic link was created, I started the Cardano node with

    ./start-node-testnet.sh

My MacBook took several hours to sync the testnet.


<a id="org2cc2f1f"></a>

## Wallets initialization

Created two key pairs

    [nix-shell]$ cardano-cli address key-gen --verification-key-file 01.vkey --signing-key-file 01.skey
    [nix-shell]$ cardano-cli address key-gen --verification-key-file 02.vkey --signing-key-file 02.skey

and their corresponding wallet addresses:

    [nix-shell]$ cardano-cli address build --payment-verification-key-file 01.vkey --testnet-magic 1097911063 --out-file 01.addr
    [nix-shell]$ cardano-cli address build --payment-verification-key-file 02.vkey --testnet-magic 1097911063 --out-file 02.addr

    [nix-shell]$ ls 01.*
    01.addr 01.skey 01.vkey
    [nix-shell]$ ls 02.*
    02.addr 02.skey 02.vkey

[Funded](https://testnets.cardano.org/en/testnets/cardano/tools/faucet/) wallet 1 (1000 Ada) and then transferred 10 Ada to wallet 2:

    cardano-cli transaction build \
        --alonzo-era \
        --testnet-magic 1097911063 \
        --change-address $(cat 01.addr) \
        --tx-in $(cat utxo1.tmp) \
        --tx-out "$(cat 02.addr) 10000000 lovelace" \
        --out-file tx.body
    
    cardano-cli transaction sign \
        --tx-body-file tx.body \
        --signing-key-file 01.skey \
        --testnet-magic 1097911063 \
        --out-file tx.signed
    
    cardano-cli transaction submit \
        --testnet-magic 1097911063 \
        --tx-file tx.signed

We will also need the public key hash of wallet 2:

    [nix-shell]$ cardano-cli address key-hash --payment-verification-key-file 02.vkey --out-file 02.pkh 
    [nix-shell]$ 
    [nix-shell]$ echo $(cat 02.pkh)
    a70db004b632525adeb7e30c64285da8bf81ee9a2946291502c0cae5


<a id="orgd5d7318"></a>

## Script Context

Besides the *Datum* and *Redeemer*, a plutus script receives a *Context*.  The datatype `ScriptContext` has two fields:

-   **~scriptContextTxnfo:** TxInfo~
-   **~scriptContextPurpose:** ScriptPurpose~

The `ScriptPurpose` describes for which purpose the script is run, and can be of four types:

-   Minting
-   Spending
-   Rewarding
-   Certifying

The datatype `TxInfo` describes the actual context.  In particular, it has fields for the list of all inputs and the list of all outputs of the Tx.  It also has a field for the datum of the UTxOs being spent by the Tx, namely  `txInfoData :: [(DatumHash, Datum)]` ,  which can be viewed as a dictionary from hashes of datum to actual datum.

Recall that a spending Tx script must include the datum of the UTxO that is being spent, whereas scripts that send money to such UTxO only had to include the datum hash.  Spending Tx's have to include the datum of the inputs they spend, oand producing Tx's opttionally can include the datum..


<a id="org37be854"></a>

### Vesting contract (`Vesting.hs`)

In the vesting example, the validator function

    mkValidator :: VestingDatum -> () -> ScriptContext -> Bool

checks that

-   the signature stored in the `TxInfo` field of the context is compared with the `PaymentPubKeyHash` of the beneficiary stored in the datum;
-   ithe `txInfoValidRange` stored in `TxInfo` occurs after the deadline stored in the datum.

The datatype `Vesting` is a *dummy* type that encodes the datum (in this case `VestingDatum`) and redeemer (in this case *unit*).

The following variables are declared:

-   `vesting`  = the untyped version of the plutus core typed validator
-   `valHash`  = the hash of the plutus core typed validator
-   `scrAddress`  = the script address, obtained from the untyped validator

The off-chain part begins with the declaration of  `GiveParams` ,  whch defines the parameters provided by one of the endpoints, namely "`give`".  The type alias  `VestingSchema`  defines the endpoints "`give`" and "`grab`" that we want to expose to the user.  Since endpoint "`grab`" needs no parameters, we write *unit* as a placeholder in place of the parameter.

The endpoints are implemented by the functions

-   **~give:** GiveParams -> Contract w s e ()~
-   **~grab:** Contract w s e ()~

(with the constraint that  `e`  belongs to class  `AsContractError` ).

**Optional exercise**:  remove the checks in the off-chain code to allow for the validator to reject transactions that do not satisfy the requirements.


<a id="orgce2f732"></a>

### Parameterized version (`Parameterized.hs`)

Allows the plutus core validator to have parameters, instead of being a (compile time) constant.

It turns out that there is a way of compiling to Plutus Core the parameter at run-time using `liftCode`.


<a id="org32a1bf8"></a>

### Serialization (`Deploy.hs`)

The `Cardano.Api` library has its own Data type, which is very similar to the Plutus Data type, but is a different type called `ScriptData`.

• The function

    dataToScriptData :: Data -> ScriptData

converts from Plutus Data to the `Cardano.Api` Script Data.

• The helper function

    writeJSON :: PlutusTx.ToData a => FilePath -> a -> IO ()

serializes a value  `a` ,  which is assumed to be in class `ToData` (a typeclass for types that can be converted to and from BuiltinData), to a file in JSON format.

For example, unit is encoded in JSON as

    [nix-shell]$ echo $(cat unit.json)
    {"constructor":0,"fields":[]}

• The helper function, which uses several functionalities from the `Cardano.Api`,

    writeValidator :: FilePath -> Ledger.Validator -> IO (Either (FileError ()) ())

converts a Plutus validator into a script and writes it into a file.

• The function

    writeVestingValidator :: IO (Either (FileError ()) ())

applies  `writeValidator <file>`  to the  `validator`  defined in `Parameterized.hs`, which in turn takes the parameters in  `VestingParams` .

In the parameters, the beneficiaty is just the `PaymentPubKeyHash` of address of wallet 2.  Address of wallet 2 was recovered with

    [nix-shell]$ cardano-cli address key-hash --payment-verification-key-file 02.vkey
    a70db004b632525adeb7e30c64285da8bf81ee9a2946291502c0cae5

The other field in the parameters that we need is the deadline, and for that we used the [Epoch Converter](https://epochconverter.com).

Function  `writeVestingValidator`  wrote the script into the file `vesting.plutus`.  If we peek into this file we get

    [nix-shell]$ cat vesting.plutus
    {
        "type": "PlutusScriptV1",
        "description": "",
        "cborHex": "<long string of serialized data>"
    }


<a id="org45d3008"></a>

### Implementing the endpoints

We now want to implement the "`give`" and "`grab`" endpoints using the cardano-cli.  

1.  Script address

    We frst need to build the script address of script `vesting.plutus`.
    
        [nix-shell]$ cardano-cli address build-script --script-file vesting.plutus --testnet-magic 1097911063 --out-file vesting.addr
    
    which produced the script-address
    
        [nix-shell]$ echo $(cat vesting.addr)
        addr_test1wzpsa88v8trjasq4n92ntzg3g04qmcjqu6fhjvr62t5qshqpvhjvh

2.  Implementing `give`

    Wrote a script that builds signs and submits tthe transaction implementing "give":
    
        [nix-shell]$ cat give.sh
        cardano-cli transaction build \
            --alonzo-era \
            --testnet-magic 1097911063 \
            --change-address $(cat 01.addr) \
            --tx-in $(cat utxo1.tmp) \
            --tx-out "$(cat vesting.addr) 200000000 lovelace" \
            --tx-out-datum-hash-file unit.json \
            --out-file tx.body
        
        cardano-cli transaction sign \
            --tx-body-file tx.body \
            --signing-key-file 01.skey \
            --testnet-magic 1097911063 \
            --out-file tx.signed
        
        cardano-cli transaction submit \
            --testnet-magic 1097911063 \
            --tx-file tx.signed
    
    Notes:
    
    -   File `utxo1.tmp` contains the Tx ID of the UTxO that was sitting at address 1 before submission.
    -   `TxOut` is a concatenation of the script address and the funds we want to "give".
    -   The hash of the serialization-as-JSON of the datum has been included.
    
```
[nix-shell]$ ./give.sh
Estimated transaction fee: Lovelace 167041
Transaction successfully submitted.
[nix-shell]$ 
[nix-shell]$ cardano-cli query utxo --address $(cat vesting.addr) --testnet-magic 1097911063         
			   TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
195fd5d43283956b194d678685fe000916ad9d66bd3f59362bcfbccf1b9b95b1     1        200000000 lovelace + TxOutDatumHash ScriptDataInBabbageEra "923918e403bf43c34b4ef6b48eb2ee04babed17320d8d1b9ff9ad086e86f44ec"
```

3.  Implementing `grab`

        [nix-shell]$ cat grab.sh
        cardano-cli transaction build \
            --alonzo-era \
            --testnet-magic 1097911063 \
            --change-address $(cat 02.addr) \
            --tx-in $(cat utxoSc.tmp) \
            --tx-in-script-file vesting.plutus \
            --tx-in-datum-file unit.json \
            --tx-in-redeemer-file unit.json \
            --tx-in-collateral $(cat utxo2.tmp) \
            --required-signer-hash $(cat 02.pkh) \
            --invalid-before 66206266 \
            --protocol-params-file protocol.json \
            --out-file tx.body
        
        cardano-cli transaction sign \
            --tx-body-file tx.body \
            --signing-key-file 02.skey \
            --testnet-magic 1097911063 \
            --out-file tx.signed
        
        cardano-cli transaction submit \
            --testnet-magic 1097911063 \
            --tx-file tx.signed

