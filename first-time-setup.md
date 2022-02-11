---
label: First Time Setup
icon: file
author:
  name: Antonio Hernandez
  email: contacto@antoniohernandez.mx
order: -1
---

# First Time Setup

The following is the step-by-step procedure that I followed for the inital
setup on a MacBook running *MacOS Catalina*.

## Nix setup

[Nix](https://nixos.org) is a tool for reproducible builds and deployment.

1 - Install Nix

    sh <(curl -L https://nixos.org/nix/install) --darwin-use-unencrypted-nix-store-volume

Follow any prompts during installation.

2 - Close terminal & reopen (to make sure that all environment variables are set)

3 - Edit the /etc/nix/nix.conf file as root

    sudo emacs /etc/nix/nix.conf

adding the following lines

    substituters        = https://hydra.iohk.io https://iohk.cachix.org https://cache.nixos.org/
    trusted-public-keys = hydra.iohk.io:f/Ea+s+dFdN+3Y/G+FDgSq+a5NEWhJGzdjvKNGv0/EQ= iohk.cachix.org-1:DpRUyj7h7V830dp/i6Nti+NEO2/nhblbov/8MW7Rqoo= cache.nixos.org-1:6NCHdD59X431o0gWypbMrAURkbJ16ZPMQFGspcDShjY=
    experimental-features = nix-command

The first two lines tells Nix to use the binary cache maintained by IOHK. Omitting those lines will result in extending the building time by many hours.

The last line tells Nix to enable `nix-command` (an experimental
feature disabled by default).

4 - Restart your computer

## Clone the Plutus repositories

5 - Clone the following

    git clone https://github.com/input-output-hk/plutus.git

    git clone https://github.com/imput-output-hk/plutus-apps.git

    git clone https://github.com/input-output-hk/plutus-pioneer-program.git

6 - Build the Plutus Core (takes some time)

While in the `plutus` directory:

    nix build -f default.nix plutus.haskell.packages.plutus-core.components.library

**Note:** Cloning the first repo in step 5 and building in step 6 might be unnecessary, since the weekly setup starts with building the other two repos.  Nevertheless, this is how I did it and it worked.
