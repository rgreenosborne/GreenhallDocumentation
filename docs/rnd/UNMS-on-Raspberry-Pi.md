## How to setup UNMS & Docker on a Raspberry Pi

This guide will show you how to run the [oznu/unms](https://hub.docker.com/r/oznu/unms/) docker image on a Raspberry Pi.

- [Requirements](#requirements)
- [Initial Raspberry Pi Setup](#initial-raspberry-pi-setup)
- [Step 1: Install Docker](#1-install-docker)
- [Step 2: Install Docker Compose](#2-install-docker-compose)
- [Step 3: Create Docker Compose Manifest](#3-create-docker-compose-manifest)
- [Step 4: Start UNMS](#4-start-unms)
- [Step 5: Managing UNMS](#5-managing-unms)
- [Updating UNMS](#updating-unms)
- [Shell Access](#shell-access)

## Requirements 

* A Raspberry Pi 3
* An SD Card

## Initial Raspberry Pi Setup

### Download and Install Raspbian Stretch

Get the latest copy of **Raspbian Stretch** from the official [Raspberry Pi website](https://www.raspberrypi.org/downloads/raspbian/) and image this to your SD card.

* **Raspbian Stretch Lite** is prefered as there is no need to run a GUI desktop.
* [How to install Raspberry Pi Operating System Images](https://www.raspberrypi.org/documentation/installation/installing-images/README.md)

### Configure Raspbian for Headless Boot

By default SSH access and WiFi are disabled in Raspbian. You can enable both these services before we boot from the SD card for the first time - this will avoid the need to ever connect a screen to the Pi.

*The following changes to the freshly imaged SD card should be made on your computer before you plug the card into the Raspberry Pi for the first time.*

#### Enable SSH

To enable remote SSH access on first boot create an empty file called `ssh` in the root of the SD card.

#### Enable WiFi (Optional)

If you have a Raspberry Pi with built in WiFi (Pi 3 or Zero W), you can configure WiFi on first boot. To do this create a file named `wpa_supplicant.conf` in the root of the SD card that contains the following (replace the [YOUR_COUNTRY_CODE](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2), YOUR_SSID and YOUR_PASSWORD values):

```
country=YOUR_COUNTRY_CODE
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="YOUR_SSID"
    scan_ssid=1
    psk="YOUR_PASSWORD"
    key_mgmt=WPA-PSK
}
```

### Login

Power on your Raspberry Pi and connect to the console using SSH. 

If you're running macOS or have Bonjour for Windows (part of iTunes) installed, you should be able to connect using `raspberrypi.local` as the hostname. If not then you'll need to find out what IP address the Raspberry Pi was assigned and connect using that instead.

```shell
ssh pi@raspberrypi.local
```

The default username is ```pi``` and password ```raspberry```. You should change the default password now using the `passwd` command.

## 1. Install Docker

Install Docker from the official repository by running these commands:

```shell
# Add Docker’s official GPG key:
curl -fsSL https://download.docker.com/linux/raspbian/gpg | sudo apt-key add -

# Use the following command to set up the stable repository:
echo "deb [arch=armhf] https://download.docker.com/linux/raspbian stretch stable" | sudo tee /etc/apt/sources.list.d/docker.list

# Update sources and install docker
sudo apt-get update
sudo apt-get install docker-ce
```

Add the ```pi``` user to the docker group:

```shell
sudo usermod -aG docker pi && logout
```
_You will need to logout of the Raspberry Pi and login again in order for your user to pick up the docker group membership._

## 2. Install Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/) allows you to easily create a manifest for your Docker containers. Since Docker Compose does not provide an official binary for ARM, you need to install it using Python:

```
sudo apt-get -y install python-setuptools && sudo easy_install pip  && sudo pip install docker-compose~=1.23.0
```

## 3. Create Docker Compose Manifest

Create a new directory to store your UNMS docker-compose manifest and config data in. In this example we will install UNMS in the ```pi``` user's home directory.

Create a new directory and change into it:

```
mkdir /home/pi/unms
cd /home/pi/unms
```

Create a new file called ```docker-compose.yml``` using ```nano```:

```
nano docker-compose.yml
```

The contents of this file should be:

```yml
version: '2'
services:
  unms:
    image: oznu/unms:armhf
    restart: always
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./config:/config
    environment:
      - TZ=Europe/London
      - PUBLIC_HTTPS_PORT=443
      - PUBLIC_WS_PORT=443
```

* The `restart: always` line instructs docker to setup the container so that it that will automatically start again if the Raspberry Pi is rebooted, or if the container unexpectedly quits or crashes.
* The `./config:/config` instructs docker to share the local folder `config` with the container. This will allow you to recreate or update the docker container without losing any settings or config.
* The `TZ=Europe/London` line sets the timezone for the container. Pick your [closest timezone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) and use that instead.
* `PUBLIC_HTTPS_PORT` and `PUBLIC_WS_PORT` must match the port your are exposing `443` to on the docker host.

Save and close the file by pressing ```CTRL+X```.

## 4. Start UNMS

Start the UNMS Docker container by running:

```
docker-compose up -d
```

* Docker will now download the latest [oznu/unms](https://hub.docker.com/r/oznu/unms/) docker image.
* It might take some time to download the initial image which is about 250 MB compressed. 
* The ```-d``` flag tells ```docker-compose``` to run the container as a background process.
* UNMS can take several minutes to start the first time you run the container. Please be patient.

You'll probably want to view the UNMS logs to check everything is working:

```
docker-compose logs -f
```

Your UNMS database, certs and config will be stored in the newly created ```/home/pi/unms/config``` directory.

## 5. Managing UNMS

To access UNMS go to `https://<ip of raspberry pi>` in your browser. For example, `https://192.168.1.20`.

You can restart UNMS by running:

```
docker-compose restart unms
```

To stop UNMS:

```
docker-compose stop unms
```

To delete the UNMS container:

```
docker-compose down
```

To bring the UNMS container back up again:

```
docker-compose up -d
```

To reset all UNMS settings:

```
docker-compose down
sudo rm -rf /home/pi/unms/config
docker-compose up -d
```


## Updating UNMS

To update UNMS you just need to pull the latest version of [oznu/unms](https://hub.docker.com/r/oznu/unms/).

Download the latest [oznu/unms](https://hub.docker.com/r/oznu/unms/) image:

```
docker-compose pull unms
```

If a newer version of the image was downloaded, recreate the container using the new image by running the ```up``` command again:

```
docker-compose up -d
```

## Shell Access

If you require shell access to the container you can run:

```
docker-compose exec unms bash
```