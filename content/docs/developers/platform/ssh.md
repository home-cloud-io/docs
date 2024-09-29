---
weight: 322
title: "Enabling SSH"
description: "A quick guide to enabling SSH on your Home Cloud device"
icon: "article"
draft: false
toc: true
---

By default, SSH is disabled on Home Cloud devices as a security measure since most users will not need SSH access to their device. As a developer, you may want SSH to debug things, sideload files or apps, and manually change system configuration.

Follow the steps below to enable SSH on your device.

## Gain device access

First you'll need to access your device by plugging in a monitor and keyboard. You should see a terminal screen that says "Welcome to NixOS" and asks for a "home-cloud login".

Type in the username `admin` then press `enter/return`.

{{< alert context="info" text="The user is always `admin` even if you chose a different username during device setup." />}}

Next it will ask you for a password, type in the password you set during device setup then press `enter/return` (you can change this password in [device settings](http://home-cloud.local/settings)).

## Updating the configuration

Now you should see a new terminal prompt: `[admin@home-cloud:~]$`

We need to update the NixOS configuration file. To do this, run the below command to edit the file:

```shell
sudo nano /etc/nixos/configuration.nix
```

Scroll down the file until you get to the line:

```nix
  # services.openssh.enable = true;
```

Uncomment this line by removing the `#` symbol so that it now looks like:

```nix
  services.openssh.enable = true;
```

Save the file by pressing `ctrl + x`, then `y`, then `enter/return`.

## Applying the configuration

Finally we need to apply the new system configuration. This can be done with a single command:

```shell
sudo nixos-rebuild switch
```

You can check if it worked by running the below command. You should see the text `Active: active (running)` in the output.

```shell
systemctl status sshd
```

Press `q` to exit the status command.

## Connecting over SSH

You're all set! You can connect to your Home Cloud device by running the below command from a terminal or any other SSH client:

```shell
ssh admin@home-cloud.local
```

Then enter the password you have defined for the device (you can change this password in [device settings](http://home-cloud.local/settings)).

{{< alert context="info" text="You can also add your SSH public key so you don't have to always enter a password by following [this guide](https://linuxhandbook.com/add-ssh-public-key-to-server/)" />}}
