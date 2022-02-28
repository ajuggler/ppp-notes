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

[Lecture Videos :icon-link-external:](https://www.youtube.com/playlist?list=PLNEK_Ejlx3x2zxcfoVGARFExzOHwXFCCL)

Topics covered:

1. Introduction
2. The EUTXO Model
3. Building the Example Code
4. Auction Contract in the EUTxO Model
5. Auction Contract on the Playground
6. Homework

I found part 1. quite motivating and part 2. very informative.  Part 3. contains the information discussed in my notes [First Time Setup](first-time-setup.md) and [Weekly Setup](./weekly-setup.md).


## Running the Playground

Part 5 of lecture 1 provided the steps for running the Playground.  These steps apply to every week.

1 - Start playground server

Inside a nix-shell, change to the `plutus-apps` directory, then:

    cd ./plutus-playground-client

Next, start the server:

    plutus-playground-server

2 - Start the playground client

Inside another nix-shell, change back into the `plutus-apps/plutus-playground-client` directory, then:

    npm start

3 - Open the playground

Direct your browser to `https://localhost:8009`


## Homework

This week's homework only asked to install the Plutus environment and run the example on the playground.

### Playground

To run the playground:

1. Copy the code into the playground's editor.
2. Press the *Compile* button and verify compilation is successful.
3. Press the *Simulate* button.
4. Build a scenario initializing wallets and constructing actions.
5. Press the *Evaluate* button.
