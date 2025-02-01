---
layout: post
title:  "My journey to Linux with Ansible 🐧"
date:   2024-11-24 10:32:20 +0100
categories: linux ansible zsh
---

> This is the story of my adventure of transitioning to linux

## Why?

I have been hacking and working with computers for almost 15 years. Along these years there were multiple time times when I have tried to switch to linux from Windows. Mostly because of the
curiosity of the new, or the promise of the more stable and faster OS. Usually these attempts are failed somewhere around 3rd or 5th days of usage with some cryptic error message after I have tried to install some package or configure a cool looking feature. 🤣

Since I have been working on my current workplace the need for an unix based os for work machine is getting bigger and bigger. Windows is simply not enough friendly for developers and the surrounding ecosystem. I know WSL is getting better and better, but still all the cumbersome problems will stay since they are originating in the root of window.

## 🔥 Why windows made my life harder

- env variable handling is painful especially PATH
- PowerShell -> please don't 🥲
- white space in folder names -> Program Files has to be used with quotes or %20 instead of whitespace
- case insensitive file system
- path separator is \
- random windows updates
- WSL and Virtualbox can run parallel but it is easily breakable
- File changes are not properly propagating when mounted from host to WSL
- All cool commandline utility can only run in WSL
- inproper cpu fan drives -> runs hotter then it is neccessary with shorter battery life

## This time I want to do the transition SMART so I have made a checklist

- every configuration has to be version controller -> no more random config file edition
- file system level option for roll back when anything goes down
- all important files either version controller OR synced to my NAS
- I want to go with the distribution which is used by the most people -> probably less surprise and bigger support
- I want to have device specific configuration

## 💀 One of the dead ends 

After quick googling [NixOS](/assets/linux-ansible/https://nixos.org/) seemed the perfect solution with its declarative nature, but after a couple of days of struggling the initial euphoria has faded away. I reached a point where configuring anything differently from the already available examples were a huge challenge. [This is the memento of my attempt.](/assets/linux-ansible/https://github.com/dtap001/nix-config)

## ✅ The working solution

> **Ubuntu 24** installed on a **LUKS** protected **ZFS** filesystem and configured by **Ansible** from **GIT** repository.
{: .prompt-info }

### Ubuntu

I have been using debian as server os so it seemed a straight forward choice since Ubuntu is debian based. So package manager and the internals of the OS will be similar for me. Ubuntu has LTS version and has one of the biggest user base in the Linux world.

### LUKS

LUKS (Linux Unified Key Setup) is a standard for disk encryption in Linux, ensuring data confidentiality by encrypting the entire disk or partitions. It supports multiple encryption keys, making it flexible and secure for protecting sensitive information. Ubuntu is supporting it so the data on my laptop will be safe if it is stollen.

### ZFS

ZFS is a modern filesystem designed for data integrity, scalability, and performance. It features built-in support for snapshots, compression, and data checksums, making it ideal for managing large amounts of data while maintaining reliability and efficiency. It seems the perfect solution for my need to have a plan B when anything goes bad.

### 🤖 Ansible

Ansible is an open-source automation tool used for configuration management, application deployment, and task automation. It operates agentless, relying on SSH to connect to systems, and uses simple, human-readable YAML files (playbooks) to define tasks. Its ease of use, scalability, and support for diverse environments make it a popular choice for automating IT workflows. There will solve my need to handle all the configuration on OS level and store my configuration in the repo it self.

## 🚀 How to have a similar fantastic setup like me? 

- download ubuntu 24 LTS from [here](/assets/linux-ansible/https://ubuntu.com/download/desktop)
- create an installer USB disk with [balenaEtcher](/assets/linux-ansible/https://etcher.balena.io/)
- boot from the USB disk
- in the installer:
  - choose interactive installation
  - choose extended selection for app install
    - select third party apps and wifi
    - select download install support for additional media formats
  - How do you want to install Ubuntu? -> select advanced features
    - erase disk and use ZFS with encryption
- after booting in you new  install create a PAT in github [LINK](/assets/linux-ansible/https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/)
managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token)
- Fork this repository [LINK](https://github.com/dtap001/epic-ubuntu-with-ansible)
- use this PAT to checkout your forked repository

```bash
    git clone https://{{ your_github_pat }}@{{ the_repository_url }} #the_repository_url ending with .git e.g.: github.com/github_username/your-forked-repo-name.git
```

## 🔨 Let's get hands dirty!

> **⚡ Before going any further it is the time to understand the basic terms of ansible and the structure of this repository**

{: .prompt-warning }

### Ansible - Playbook

 A YAML file that defines a set of plays (which include tasks) to be executed on target machines.

### Ansible - Tasks

Individual operations like installing packages, managing services, copying files, or executing commands.

### Ansible - Inventory

A file that lists the remote machines (hosts) Ansible will manage, often grouped under categories like web_servers or db_servers.

### Okay, showtime!

- in your checkout run the following script to generate your host configuration based on the host name of the current machine you are running this script

> It is really important to understand how this script works. It will configure the current host that is it running based on the hostname of it. So a hostname will identify a specific ansible inventory host and will be using the configuration according to that.
The user in the ansible playbooks will be the one that is running this script. So make sure you run it with the appropriate user.
{: .prompt-warning }

```bash
./configure.sh
```

- first it will ask for your new machine hostname. This hostname is important because configurations are separated based on it. This way you can have all of your machines configuration in one repository
![input your hostname](/assets/linux-ansible/input-your-hostname.png)

- after this it will ask every configuration detail. only respond with Y if the default value is OK for you. Make sure you set the username to the same as the currently logged in user!
![config](/assets/linux-ansible/config.png)

- roles can be turned on and off. They contain apps and configurations for a specific purpose.
![role](/assets/linux-ansible/role-select.png)

- finally it will ask for your sudo password for tasks which requires elevated permissions
![become-ask](/assets/linux-ansible/become-ask.png)


### Short tutorial for the setup

The following list contains the utilities that are installed with this setup. 

- first of all zsh!
  it is the go to shell when interacting with linux. It makes it more fluent and hassle free.
