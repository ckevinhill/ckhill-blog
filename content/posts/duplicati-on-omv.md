---
title: "Setting up Duplicati on OpenMediaVault5"
date: 2021-08-07T08:44:55+08:00
tags: ["tutorial", "other"]
---

## Backup Approach

In order to provide robust backup of family photos and videos I have the following setup:

* NAS-attached 1 Tb drive - primary location for storing and sharing files on home network
* NAS-attached 1 Tb drive - local backup of primary drive
* Backblaze Cloud storage - provides non-local backup of file

The NAS-attached drives are connected to a [RaspberryPi](https://www.raspberrypi.org/) running [OpenMediaVault](https://www.openmediavault.org/).  In order to schedule and execute local and cloud _secure_ backups [Duplicati](https://www.duplicati.com/) is additionally installed.

## OMV4 to OMV5

Previous versions of OpenMediaVault have provided the ability to install "extras" (like duplicati) that are run directly on the raspberrypi (in host debian OS).  As of OMV5 a docker-based strategy has been adopted to better seperate "extras" and associated dependencies.  This is a significantly better approach given challenges in managing different aspects of [RaspberryPi OS](https://www.raspberrypi.org/software/) vs. specific applications like duplicati.

As an example - recent versions of duplicati have required TLSv1.2 which is not supported via installed libraries of RaspberryPi OS.  This has led to issues where duplicati has stopped working for remote backups due to issues connecting to SSL-secured APIs.

## Installing RaspberryPi OS

You will need to install the latest [RaspberryPi OS](https://www.raspberrypi.org/software/) onto your SD card.  As we will be using our Raspberry Pi primarily as a network-attached storage device we should install _Raspberry Pi OS Lite_ version.

The latest IMG files can be downloaded and then written to your SD card via a utility like [Etcher](https://www.balena.io/etcher/) (recommended over the RaspberryPi Image utility).
![Etcher](https://www.balena.io/blog/content/images/2020/07/etcher-1.png)

## Initial Pi Setup

For initial setup and network attachment we will need to connect the Pi to a keyboard and monitor.  Once the below steps have been completed you can disconnect monitor and keyboard and connect to the headless Pi via SSH (default username: pi, password: raspberry).

### Connecting to Wifi

We will need to configure the wpa client on the Pi to enable network connectivity:

```bash
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```

And add the following lines:

```bash
network={
   ssid="network_name"
   psk="network_password"
}
```

More complete instructions can be found [here](https://raspberrypihq.com/how-to-connect-your-raspberry-pi-to-wifi/).

### Enabling SSH

We will want to access our Pi via SSH.  We can use the `raspi-config` utility:

![raspi-config](https://phoenixnap.com/kb/wp-content/uploads/2021/04/raspi-config-interfacing-options.png)

More complete instructions can be found [here](https://phoenixnap.com/kb/enable-ssh-raspberry-pi).

## Installing OpenMediaVault

Installation of latest OMV can be execute via:

```bash
wget -O - https://raw.githubusercontent.com/OpenMediaVault-Plugin-Developers/installScript/master/install | sudo bash
```

![OMV](https://pimylifeup.com/wp-content/uploads/2020/04/Raspberry-Pi-OpenMediaVault-Login-Screen.png)

Really good instructions on OMV installation, disk attachment and user-setup can be found [here](https://pimylifeup.com/raspberry-pi-openmediavault/).

## Enabling Docker/Portainer (OMV-Extras)

Now that OMV is installed we can enable the `OMV-Extras` functionality which includes [Docker](https://www.docker.com/) and [Portainer](https://www.portainer.io/) a UI for Docker management).

![OMV-extras](/images/omv-docker-extra.png)

For both Docker and Portainer choose the `Install` options and wait for installation to complete.

### Duplicati container setup

#### Add Container

Login to portainer interface (e.g. `https://raspberrypi:9000`), select local Docker and then _Add Container_:

![Step 1](/images/portainer-container-setup-1.png)

#### Set Image

Specify docker image as `linuxserver/duplicati:latest` and map host port 8200 to container port 8200 (or choose an alternative host port if you want to access duplicati on another port).

![Step 2](/images/portainer-duplicati-1.png)

#### Map Bindings

In the _Advanced Container_ settings specify bindings between local (host) system and container to indicate the data you want to backup and the locations for backup.

* /source - location of data that you will backup
* /backups - location where backups will be stored

If you do not specify volume bindings they will be automatically created to default locations (e.g. in picture above /backup was used so /backups was automatically created).

![Step 3](/images/portainer-duplicati-2.png)

#### Env Variables

Specify the following environment variables:

* PGID = 1000
* PUID = 1000

For more information on PG/PUID see [here](https://docs.linuxserver.io/general/understanding-puid-and-pgid).

![Step 4](/images/portainer-duplicati-3.png)

#### Restart Policy

Change _Restart policy_ to `Unless stopped`.

#### Deploy Container

You should now be able to _Deploy Container_.  Once container is deployed you will be able to see it with status _running_ in the container dashboard view:

![Container Running](/images/portainer-duplicati-4.png)

## Backblaze Account creation

[Backblaze](https://www.backblaze.com/) is a low-cost backup storage provider.  Other provides like Azure, etc. are [also supposed by Duplicati](https://duplicati.readthedocs.io/en/latest/01-introduction/#supported-backends) but I have generally found BackBlaze ([B2](https://www.backblaze.com/b2/cloud-storage.html)) to be most cost effective.

Account Creation is fairly straight-forward:

* [Sign-up for B2 storage account](https://www.backblaze.com/b2/sign-up.html?referrer=nopref)

>When creating account make sure to record Backup codes as these will be required to access account if your MFA stops working!

* Create a new bucket (e.g. `duplicati-backup-[name]`) - bucket names are globally unique so you may need to play around awhile before you find one that works.

![B2 Bucket](/images/b2-bucket.png)

* Generate an _App Key_ - this will be used by duplicati to read/write to your bucket.

>Store the keyId and applicationKey values somewhere as you will need them for duplicati configuration.

For more details on account creation and duplicati setup see [here](https://www.backblaze.com/blog/duplicati-backups-cloud-storage/).

## Duplicati configuration

Configuration in duplicati involves:

* Add backup
* Select B2 Storage type
* Specify Source files and backup frequency

![Duplicati B2](https://www.backblaze.com/blog/wp-content/uploads/2020/11/image11-1-1024x776.png)

More details can be found [here](https://www.backblaze.com/blog/duplicati-backups-cloud-storage/).

> Depending on the size of your files it may take several days for the initial backup task to run, after that backups should only be incremental files that are added.

For my setup I have 2 tasks that run:

* Local backups (from 1 external drive to another) - every 2 days
* Remote backups (to B2 cloud storage) - every week
