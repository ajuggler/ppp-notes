---
label: Lecture 3
icon: file
author:
  name: Antonio Hernandez
  email: contacto@antoniohernandez.mx
order: -6
---

# Vesting example

[Lecture Videos :icon-link-external:](https://www.youtube.com/playlist?list=PLNEK_Ejlx3x2zxcfoVGARFExzOHwXFCCL)

Topics covered:

1. Configuring playground time out
2. Script contexts
3. Handling time
4. A vesting example
5. Parametrized contracts
6. Deploying to the Cardano testnet
7. Homework
8. Summary


## Installing the Cardano components on **MacOS Catalina**

Before deploying on the Cardano testnet, I had to build the `cardano-node` and
`cardano-cli` (the Cardano Node and Cardano Command Line interface).

The repository of the cardano node components sits at 
https://github.com/input-output-hk/cardano-node .

The following steps worked for me (based and modified from the *Installing
cardano-node* [instructions at
developers.cardano.org](https://developers.cardano.org/docs/get-started/installing-cardano-node/)).

### XCode

1 - [Downloaded](https://developer.apple.com/xcode/) Xcode and the Xcode
Comand Line Tools.  Since my computer runs MacOS Catalina, I downloaded
version 12.4.  [Homebrew](https://brew.sh/) needs to also be installed.

### Installing Homebrew packages

2 - Installed some libraries with `brew`:

```
brew install jq
brew install libtool
brew install autoconf
brew install automake
brew install pkg-config
```

!!!
My Mac runs on an Intel chip.  If yours uses **M1**, refer to the
[instructions at
developers.cardano.org](https://deve\lopers.cardano.org/docs/get-started/installing-cardano-node/)
for further steps.
!!!

### Downloading & Compiling

3- Created a working directory to store the source-code and build the
components.

```
mkdir -p $HOME/cardano-src
cd $HOME/cardano-src
```

4 - Install *libsodium*

Next, I downloaded, compiled and installed `libsodium`:

```
git clone https://github.com/input-output-hk/libsodium
cd libsodium
git checkout 66f017f1
./autogen.sh
./configure
make
sudo make install
```

5 - I added the following lines to my bash shell profile `$Home/.bashrc`:

```
export LD_LIBRARY_PATH="/usr/local/lib:$LD_LIBRARY_PATH"
export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH"
```

Reloaded the shell profile with `source $Home/.bashrc`.

5 - Install the **Cardano** components

Next, I downloaded, compiled and installed `cardano-node` and `cardano-cli`:

```
cd $Home/cardano-src
```

Downloaded the `cardano-node` respository:

```
git clone https://github.com/input-output-hk/cardano-node.git
cd cardano-node
git fetch --all --recurse-submodules --tags
```

Switched the repository to the latest tagged commit:

```
git checkout $(curl -s https://api.github.com/repos/input-output-hk/cardano-node/releases/latest | jq -r .tag_name)
```

Built the node

```
cabal build all
```

and installed the newly built node and CLI to `$Home/.local/bin`

```
mkdir -p $HOME/.local/bin
cp -p "$(./scripts/bin-path.sh cardano-node)" $HOME/.local/bin/
cp -p "$(./scripts/bin-path.sh cardano-cli)" $HOME/.local/bin/
```

Added the following line to the shell profile `$Home/.bashrc`

```
export PATH="$HOME/.local/bin/:$PATH"
```

and reloaded with `source $Home/.bashrc`.

To verfiy all is OK, checked versions installed:

```
cardano-cli --version
cardano-node --version
```


## Running cardano-node (with external hard drive)

Set the current directory to `plutus-pioneer-program/code/week03/testnet`.  It
contains the [configuration
files](https://hydra.iohk.io/build/8111119/download/1/index.html) for the
**testnet**:

- `testnet-config.json`
- `testnet-byron-genesis.json`
- `testnet-shelley-genesis.json`
- `testnet-alonzo-genesis.json`
- `testnet-topology.json`

The script `start-node-testnet.sh` runs `cardano-node` with the following
configuration:

```
cardano-node run \
 --topology testnet-topology.json \
 --database-path db \
 --socket-path node.socket \
 --host-addr 127.0.0.1 \
 --port 3001 \
 --config testnet-config.json
```

The `database-path` sets the name of the database directory.  For me it was
desirable to have the database on an **external hard drive**.  This is
achieved simply by setting a symbolic link.  Assume we want the database to
sit inside directory `cardano` at the root of an external hard drive named
*External-Hard-Drive*.  Remembering that the active directory on our shell is
still `testnet`, we execute

```
mkdir -p /Volume/External-Hard-Drive/cardano/db
ln -s /Volume/External-Hard-Drive/cardano/db .
```

After the symbolic link was created, I started the Cardano node with

```
./start-node-testnet.sh
```

My MacBook took several ours to sync the testnet.