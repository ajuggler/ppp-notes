---
label: Weekly Setup
icon: file
author:
  name: Antonio Hernandez
  email: contacto@antoniohernandez.mx
order: -2
---

# Weekly Setup

Each week while the pioneer program is running, the [Plutus Pioneer Program
repository](https://github.com/input-output-hk/plutus-pioneer-program) will be
updated.  As a consequence, these steps should be executed weekly.

!!!
Make sure to have done the [first time setup](./first-time-setup.md).
!!!

## Pull updates

1 - Change into the `plutus-apps` directory, then:

    git pull

Most likely step 1 is unnecessary, but just to make sure.  What is definitely necessary is the next step.

2 - Change into the `plutus-pioneer-program` directory, then:

    git pull

## Get tag of this week's package

3 - While in the `plutus-pioneer-program` directory:

    cd ./code/week01

Substitute `week01` with the corresponding week you are in.

4 - View the Cabal project file.

    emacs cabal.project

Look for the following lines

    source-repository-package
      type: git
      location: https://github.com/input-output-hk/plutus-apps.git
      tag: [long string of letters and numbers]

Copy the `tag` value [long string of letters and numbers]

5 - Change back into the `plutus-apps` directory, then:

    git checkout [tag]

where [tag] is the tag value on your clipboard.

## Start Nix Shell

6 - While still in the `plutus-apps` directory:

    nix-shell

The first time it takes a while (20-30 minutes).

**Note:** Along the way it will be necessary to start more Nix shells; all of them should be started while in the `plutus-apps` directory.

## Build this week's project

7 - Inside the nix-shell,

    cd ../plutus-pioneer-program/code/weeknn

where 'nn' is this week's number.  Then

    cabal build

The first time it may take a while.

## Haddock documentation

8 - Inside a nix-shell, change back into the `plutus-apps` directory, then:

    build-and-serve-docs

This builds the haddock documentation, while will be available at `http://localhost:8002/haddock/`

The first time it may take a while.

**Note:**  Inside the haddock documentation, press command-S to search.

## End of week

At the end of the week I found it useful to recup hard drive space with

    nix-collect-garbage

which I ran while in the `plutus-apps` directory.
