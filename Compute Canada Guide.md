
[![N|Solid](https://metacreation.net/wp-content/themes/mama-word-v4.0.1/images/Brand-Regular-White-right-p-500.png)](https://metacreation.net/category/projects/)

# Compute Canada Guide

**Table of contents**
- Introduction
- Connecting to CC
- Navigating the remote machine
- Python environments
- Loading modules

## Introduction

It took me a while to figure out all the different bit and pieces to use Compute Canada (CC) correctly, so I created this .md to help out anyone trying to use it. Think of it as an introductory cheatsheet.

## Connecting to CC

Make sure you have a Compute Canada account and that is has been approved.

This might be obvious to some, but the first thing you need is to register your computer (by generating a key and sharing it with compute Canada) so you can ssh through the terminal.

First, generate an ssh key file. I'm using windows in my case so I generated it like [this](https://phoenixnap.com/kb/generate-ssh-key-windows-10).

Once you have the file, locate it, open it, copy the public key and paste it [here](https://ccdb.computecanada.ca/ssh_authorized_keys) (you'll need to log in to CC first).

By default we should have access to Cedar, Beluga or Graham, so once the ssh key is registered you can connect with the following command on your terminal.

```sh
ssh -Y USERNAME@cedar.computecanada.ca
```

And that's it!

## Navigating the remote machine

### Local vs Global paths

In order to *cp* or *mv* files, you must provide the global paths of both source and destination. So you should put things like:
```sh
cp /home/USERNAME/projects/def-pasquier/USERNAME/file.md /home/USERNAME/projects/scratch
```

For the rest of things, standard Linux commands apply when navigating. 

### Folder structure
In the Metacreation lab we have a shared folder and personal folders. Your */home* folder can be used for you to store anything, and it has 50GB of storage. The */scratch* folder has 1TB of storage that can be used freely but files should be stored there only temporarily, since this folder is purged. Finally, */project* is a shared folder between lab members. It has 1TB of space (although most of it us used by other lab members) and it's a good place to store large files or to put files you want to share with other members.

## Python environments
**Unfortunately Conda is not recommended for use in CC. So we cannot create Conda Envs.**

In order to run python code with dependencies, you will need to create an environment. There are several ways to do this, but I'll put here the one officially recommended by CC. To create and activate an environment you should run the following commands:
```sh
virtualenv --no-download ./ENV                # ENV is the name of the environment
source ./ENV/bin/activate
pip install --no-index --upgrade pip          # You should also upgrade pip
pip install --no-index --upgrade setuptools
```
To deactivate the environment you should run:
```sh
deactivate
```
In order to install dependencies and packages, you should always try the flag *--no-index*, to not install from PyPI, but instead to install only from locally-available packages, i.e. the Compute Canada wheels. If the package is not available, you can pip install it with a remote URL, but maybe this will cause conflicts.

## Loading modules
In the simplest case, the software you need will already be installed on one of the compute servers. It will then be accessible in the form of a "module". If this is not the case, see more info in [this](https://docs.alliancecan.ca/wiki/Utiliser_des_modules/en) link.

To use modules, if is useful to first use the command spider to see if there are any dependencies you need to load first. For example,
```sh
module spider protobuf
```
lists all protobuf versions available for loading. If we spider one of the versions,
```sh
module spider protobuf/3.12.3
```
the system gives us a dependency that must be loaded before loading protobuf. In this case, the dependency **StdEnv/2020**.

To load any module, we use the load command. In this case, we would need to load both modules in order:
```sh
module load StdEnv/2020
module load protobuf/3.12.3
```
> **WARNING**: Affter contacting compute canada, I was recommended not to use any module from nixpkgs/16.09, because it is a old environment and they do not support it anymore. This would mean that we shouldn't try any more developments in Python 3.6. Instead we should try to develop in the CC officially supported versions (as of today 14/05/21) python/3.10.2 or python/3.9.6.


