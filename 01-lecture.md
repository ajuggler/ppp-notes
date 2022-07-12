---
label: Lecture 1
icon: file
author:
  name: Antonio Hernandez
  email: contacto@antoniohernandez.mx
order: -4
---

# EUTxO and English Auction

## Lecture 1

[Lecture Videos :icon-link-external:](https://www.youtube.com/playlist?list=PLNEK_Ejlx3x2nLM4fAck2JS6KhFQlXq2N)

Topics covered:

1. Introduction
2. The EUTXO Model
3. Building the Example Code
4. Auction Contract in the EUTxO Model
5. Auction Contract on the Playground
6. Homework

I found part 1. quite motivating and part 2. very informative.  Part
3. contains the information discussed in my notes [First Time
Setup](first-time-setup.md) and [Weekly Setup](./weekly-setup.md).


## Running the Playground

Part 5 of lecture 1 provided the steps for running the Playground.
These steps apply to every week.

1 - Start playground server

Inside a nix-shell, change to the `plutus-apps` directory, then:

    cd ./plutus-playground-client

Next, start the server:

    plutus-playground-server

2 - Start the playground client

Inside another nix-shell, change back into the
`plutus-apps/plutus-playground-client` directory, then:

    npm start

3 - Open the playground

Direct your browser to `https://localhost:8009`


## The UTxO & EUTxO protocols


### The UTxO model

In the UTxO model, *transaction outputs* exist on the blockchain in
two possible modes: spent or unspent.  A (new) *transaction* takes as
inputs *unspent transaction outputs* (UTxO's) and produces as outputs
new UTxO's.  Ignoring for the time being the presence of fees, or
minting and burning mechanisms, there is a constraing on the balances
of the *input* and *output* UTxO's:

> sum of balances of input UTxO's = sum of balances of output UTxO's

We say that the input UTxO's are consumed by the transaction.  In this
process, the transaction output passes from being *unspent* to being
*spent*.  (Afterwards the transaction output will forever remain on
the blockchain in a passive state of having been spent; it will no
longer participate in future transactions and from then onwards its
only role will be to serve as a historic record.)

In order for a transaction (`Tx`) to be able to consume an unspent
transaction output (`UTxO`), it has to be signed by the owner of such
`UTxO`.

If a particular `UTxO` is spent, it has to be spent in full; there are
no *partial spends* in the model.  If only part of the funds stored in
a `UTxO` need to be spent, the way it works is that the `Tx` that
consumes the `UTxO` produces as one of its outputs a new `UTxO` with
"the change" corresponding to the excess funds.


### The EUTxO model

**Cardano** extends the UTxO model by allowing that arbitrary logic to
  take place instead of just simple transfer of funds.  This is called
  the *Extended Unspent Transaction Output* (EUTxO) model.

In the EUTxO model, instead of an address, a `UTxO` provides an
arbitrary *Script*; and instead of a signature, an input of `Tx`
provides a *Redeemer* consisting of arbitrary data which the script
uses to validate the transaction.  Since the script validates the
transaction, we also refer to it as the *validator*.

In order to validate the spending of the `UTxO`, the script uses the
information in the Redeemer and also the Context provided by all the
inputs and outputs in that particular spending `Tx`.  In adition, the
`UTxO` also can carry a piece of data, called *Datum*, that can be
used by the Plutus script.

So, in adition to value in Ada or other tokens, a script address can
carry arbitrary data called Datum.  With this, it can be
mathematically proved that the EUTxO model is at least as expressive
as Ethereum's model.


#### Validation

Who is responsible for providing the script (validator), datum and
redeemer?

The `UTxO` that sits on a particular script address only needs to
provide the hashes of the script and the hash of the datum.
Optionally it can provide the datum and/or the script (not only their
hashes).

The spending `Tx` has to include the full Script, Datum and Redeemer.
So, in order to spend a UTxO, the spending `Tx` needs to know the
Datum.

## Homework

This week's homework only asked to install the Plutus environment and
run the example on the playground.

### Playground

To run the playground:

1. Copy the code into the playground's editor.
2. Press the *Compile* button and verify compilation is successful.
3. Press the *Simulate* button.
4. Build a scenario initializing wallets and constructing actions.
5. Press the *Evaluate* button.

