---
label: Lecture 5
icon: file
author:
  name: Antonio Hernandez
  email: contacto@antoniohernandez.mx
order: -8
---

# 

[Lecture Videos :icon-link-external:](https://www.youtube.com/playlist?list=PLNEK_Ejlx3x0G8V8CDBnRDZ86POVsrfzw)

Topics covered:

1. Introduction
2. Values
3. A simple minting policy
4. A more realistic minting policy
5. NFT's
6. Homework


## Native tokens and value

The topic of this lecture is to explain how native tokens can be minted and
burned in Cardano.

Recall that each UTxO has an address and a value (in adition of a Datum).  In
Plutus, the `Value` is defined as a map from `CurrencySymbol`s to maps from
`TokenName`s to `Integer`s.

The `Value` module is defined in `Plutus.V1.Ledger.Value`.

Each native token (includying ADA) is identified by a *currency symbol* and a
*token name*.  Both are ByteStrings.

A *value* tells how many units are contained in each asset class.  The latter
is identified by a pair of currency symbol and token name.


## Minting policies

Recall that, in general, the validation of a script requires a *Datum*, a
*Redeemer* and a *Context*.  For a *minting policy*, the Datum is not
required.

The `ScriptContext` contains the `TxInfo` and the `ScriptPurpose`.

The `ScriptContext` is defined in `Plutus.V1.Ledger.Contexts`.

A minting policy is triggered if the `txInfoMint` field of the `TxInfo` has a
non-zero *value* (a "bag" of different asset classes, each identified by a
currency symbol and a token name), specified in the field `txInfoMint`.

For each *currency symbol* in the value specified by `txInfoMint` a
corresponding minting policy script is run.  Each currency symbol is the hash
of one such minting policy script.

A minting policy script only has two inputs: a *redeemer* and a *context* (no
datum).  Recall that the transaction provides the redeemer.


## A parametrized minting policy

In part 4, Lars discusses a minting policy that can only be triggered by a
specified public key.  This is achieved by parametrizing a minting policy so
that it depends on a value of type `PaymentPubKeyHash`.  The corresponding
policy in PlutusCore takes an argument that has to be lifted.


## NFT's

NFT's are discussed in part 5.  An NFT is a token that can only exist once.
The trick to ensure uniqueness of the token is to include the ID of the UTxO
as a parameter in the minting policy.  (There can not be two different UTxO's
with the same ID; Lars gives an inductive argument proving this claim based on
the fact that *fees* are always present.)

Notice that it is not enough to constrain a transaction to only mint once,
since nothing prevents having many transactions with such a constraint.

## Homework (*spoiler alert*)

Two problems:

1. Write a contract with a Shelly-era solution for creating an NFT, based on a
deadline not too far in the future.

2. Write a minting policy for an NFT where the TokenName is an empty
ByteString.

### Solution to homework 1

Probably the main challenge was to create a minting policy with two
parameters.  My solution can be found in file
[mySolution1.hs](https://github.com/ajuggler/ppp-solutions/tree/main/lecture05/mySolution1.hs)
.

### Solution to homework 2

The only difficulty that I found was how to represent the empty ByteString.
It is expressed using the data constructor `TokenName`, so that the empty
ByteString is represented by `TokenName emptyByteString`.

Something that puzzles me is that apparently it is sufficient to put the empty
ByteString constraint only on the off-chain part of the code.  To be precise, I
found that my solution still works if I sustitute the code

```haskell
    checkMintedAmount :: Bool
    checkMintedAmount = case flattenValue (txInfoMint info) of
      [(_, tn, amt)] -> tn == (TokenName emptyByteString) && amt == 1
      _              -> False
```

with the alternative

```haskell
    checkMintedAmount :: Bool
    checkMintedAmount = case flattenValue (txInfoMint info) of
      [(_, _, amt)] -> amt == 1
      _             -> False
```

In any case, my solution can be found in file
[mySolution2.hs](https://github.com/ajuggler/ppp-solutions/tree/main/lecture05/mySolution2.hs)
.