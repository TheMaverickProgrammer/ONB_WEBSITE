# ARM Rasp Pi 3 On Bazzite

<!-- TOC -->
- [ARM Rasp Pi 3 On Bazzite](#arm-rasp-pi-3-on-bazzite)
	- [Author](#author)
	- [About](#about)
	- [Image](#image)
- [Tutorial](#tutorial)
	- [Preparing `Qemu`](#preparing-qemu)
	- [Setup](#setup)
	- [Credentials](#credentials)
	- [Resizing](#resizing)
- [Tool chains](#tool-chains)
	- [System](#system)
		- [`OpenSSL`](#openssl)
	- [Rust](#rust)
	- [C++](#c)
- [Transferring Files](#transferring-files)
	- [From Host To Container](#from-host-to-container)
	- [Container To Image](#container-to-image)
	- [Image To Container](#image-to-container)
	- [Container To Host](#container-to-host)
<!-- /TOC -->

## Author
James King | <https://github.com/TheMaverickProgrammer>

## About
This tutorial installs a raspberry pi 3 virtual image running inside of `podman` on [Bazzite][BZZ]; configured for modern development. 

These instructions should also work on Fedora or other compatible unix-based operating systems.

## Image

I used this image: `lukechilds/dockerpi pi3`.

>Source: <https://github.com/lukechilds/dockerpi>

# Tutorial
## Preparing `Qemu`

These are the shell commands I ran to prepare qemu on my Bazzite host machine.

```bash
sudo -i
rpm-ostree install qemu-user-static
systemctrl reboot
```
## Setup

We want to mount our file-system image in order to resize it later with `qemu`. Before we can mount the image, we need to be sure rootless podman has the correct permissions to access our host using `unshare`. We will create a subdirectory on our host machine that we will share and mount the file-system named `.dockerpi`.

```bash
mkdir ./dockerpi
podman unshare chown 200:200 -R .dockerpi/
```

In this example, we are choosing to assign `200` as the `userID` given to podman later.

> The `:Z` flag is crucial on SELinux-enabled systems to allow container access to the mounted directory.

We can verify that permissions were given by running `unshare` with `-al`.

```bash
podman unshare ls -al .dockerpi
	total 4024112
	drwxrwxrwx. 1  200  200         28 Nov  7 21:17 .
	drwx------. 1 root root        712 Nov  7 22:49 ..
	-rw-r--r--. 1 root root 8589934592 Nov  7 23:53 filesystem.img
```

We can see the first row has `200` in the `userID` columns.

Now we are ready to pull and install the image to the target volume:

```bash
podman run --user 200 -it -v $HOME/.dockerpi:/sdcard:Z \
	lukechilds/dockerpi pi3
```

This will take some time. Once the setup is finished, enter the default pi credentials.
## Credentials

user: `pi`
pass: `raspberry`

One the system has authorized and given control over to shell, check to see if the environment looks healthy. Then, power off the container. The file-system will persist.

```bash
sudo poweroff
```

> The `pi3` image is experimental and has issues on shutdown. Once you see the shutdown process is reached, it __should__ hang. This is expected. Simply `stop` the container using `podman` in another terminal.
## Resizing

The default pi image is too small for modern development. We are going to use `qemu-img` to resize the image and then update the boot sectors of the image to adapt to the new size. In this example, the new size is `10G`.

```bash
sudo qemu-img resize -f raw $HOME/.dockerpi/filesystem.img 10G
startsector=$(fdisk -u -l $HOME/.dockerpi/filesystem.img | grep filesystem.img2 | awk '{print $2
}')
```

Verify that `startsector` has the new address.

```bash
echo $startsector
	532480
```

Then use `parted` to make a new partition in the image.

```bash
sudo parted $HOME/.dockerpi/filesystem.img --script rm 2
sudo parted $HOME/.dockerpi/filesystem.img --script "mkpart primary ext2 ${startsector}s -1s"
```

Now restart the container. Reminder that this container will automatically launch a shell inside the pi image.

```bash
podman run -it -v $HOME/.dockerpi:/sdcard:Z lukechilds/dockerpi pi3 -u 200
```

When you are back inside the pi's shell, identify the device to resize. This may be similar to the MBR name. In the example below, the boot partition was `mmcblk0p1` and the partition we should resize is `mmcblk0p2`.

```bash
sudo fdisk -l
	Device         Boot  Start      End  Sectors  Size Id Type
	/dev/mmcblk0p1        8192   532479   524288  256M  c W95 FAT32 (LBA)
	/dev/mmcblk0p2      532480 20971519 20439040  9.8G 83 Linux

```

Resize the partition to the new size given to `qemu` earlier. 

> Note that the following example, the author only gave it `6G` of the full `10G`, but this is not recommended.

```bash
sudo resize2fs /dev/mmcblk0p2 6G
```

Confirm that your file-system can see the new memory.
```bash
dh -f
	Filesystem      Size  Used Avail Use% Mounted on
	/dev/root       5.9G  2.7G  3.0G  47% /
```

This concludes the resizing steps.

>Further reading <https://stackoverflow.com/questions/68101936/how-to-increase-the-size-of-dev-root-on-a-docker-image-on-a-raspberry-pi/68328492#68328492>

# Tool chains

In order to build anything, we need to have the right system packages installed.
## System

First we need to update our raspi OS package manager and system binaries before we run the commands to setup any necessary tool chains. While still inside the pi, run:

```bash
sudo apt get update
sudo apt upgrade
```

Reboot the image. It's a good idea to install `git` once you log back into the pi.

```bash
sudo apt get install git
```

### `OpenSSL`

We need AARCH64 libs for OpenSSL.

```bash
sudo apt-get install -y openssl:armhf
sudo apt-get install -y libssl-dev:armhf
```

This concludes the system setup.
## Rust

For ONB servers run this install script for ARM.

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Add `cargo` to user's environment.

```bash
source $HOME/.cargo/env
```

Confirm Rust is installed.

```bash
rustc --version
	rustc 1.91.0 (f8297e351 2025-10-28)
```

With the tool-chains and dependencies installed, the server can be compiled with:

```bash
cargo build --release
```

## C++

TODO

# Transferring Files

There are three layers composing this virtualization of pi3:
1. The user's host machine running podman.
2. Podman running the container.
3. The container running the raspi image via qemu.

## From Host To Container

In order to share programs built within in the image back to the host, we need a way to access the file-system; this time as a local volume.

In terminal of the _host_ machine, identify the container to mount.
```bash
podman ps
	CONTAINER ID  IMAGE                                 COMMAND     CREATED      STATUS      PORTS       NAMES
	2443e4c8876e  docker.io/lukechilds/dockerpi:latest  pi3 -u 200  2 hours ago  Up 2 hours              loving_mccarthy
```

For convenience, define a variable to hold the path to the mounted volume.

```bash
mnt=$(podman mount CONTAINERID)
```

> If you get an error that podman must execute unshare first, run `podman unshare` and try again.

Now we can treat this path as a native path in our host machine's terminal. For example:

```bash
cp -R ${mnt}/etc/foobar /tmp
```

This command will copy `/etc/foobar` in the container's file-system to host's `/tmp` directory.

Whenever you're done, `umount` the volume.

```bash
podman umount CONTAINERID
```

Use podman to restart the container or `sudo reboot` if you have a session within the container. Once more, start up the container.

Next, we want to `wget` our files over the virtual network device and we need to determine what our container IP address is for the pi image.

Run `podman ps` to find the name of the container we want to enter via `sh`.

```bash
podman ps
	CONTAINER ID  IMAGE                                 COMMAND     CREATED       STATUS       PORTS       NAMES
	bd151c72b97a  docker.io/lukechilds/dockerpi:latest  pi3 -u 200  11 hours ago  Up 11 hours              jovial_murdock
```

In this example, the name is `jovial_murdock`.

```bash
podman exec -it jovial_murdock /bin/sh
```

Finally run `ifconfig` to see how the container identifies itself to the qemu image.

```bash
wlp2s0    Link encap:Ethernet  HWaddr A6:F6:04:6F:1E:6A  
          inet addr:192.168.0.110  Bcast:192.168.0.255  Mask:255.255.255.0
```

## Container To Image

Because we equipped `qemu` with a network device earlier, back inside the raspberry image, we can use `wget` on the IP address to download the file in the container.

```bash
wget 192.168.0.110/<FILE>
```

## Image To Container

If we ever need to perform the reverse operation - that is, exporting files from within our qemu image back into the container - we use `curl` to _upload_  files over the network to the container's `ftp` server.

> If the pi image had `ftpd` installed we could have used `curl` for downloading files too, but I didn't know that at the time of writing this tutorial.

In the container, bind a tcp socket and run `ftpd` on it exposing the `/sdcard` path we used earlier.

```bash
tcpsvd -vE 0.0.0.0 21 ftpd -w -A /sdcard
```

Back in the pi shell, `curl -T` to the IP we identified earlier in order to send the file. Note that we explicitly use the `:21` port for both the tcp service and the curl commands.

```bash
curl -T /<FILE> ftp://192.168.0.110:21
```
> Be sure that `<FILE>` is the absolute path to the file you wish to upload.

When the transfer(s) is/are complete, you can terminate the `tcpsvd` process.
## Container To Host

Because we mounted the container volume earlier in this tutorial, our work is now trivial. We can easily fetch that output file from our host machine. This was shown in the very first steps of this section. For convenience, another example is provided.

```bash
 cp .dockerpi/<FILE> ~/Desktop/
```

[BZZ]: https://bazzite.gg/