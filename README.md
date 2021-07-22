# Things to do to install Linux on the Asus ROG Flow X13

This shall be the guide to install Linux or in this particular case Pop!_OS on the Asus ROG Flow X13. Partly forked from [this repo](https://github.com/CO-1/asus-flow-x13-linux)

## Preparation

## Deactivate Secureboot
Go into your BIOS by spamming DEL during boot. Under advanced (F7), go into Security and deactivate Secureboot.

## Deactivate Bitlocker Drive Encyrption
Boot into Windows and deactivate Bitlocker Drive Encryption.

## Resize Windows Partition
Using Windows standard tools resize your main partition and make room for your linux install.

## Installation
Download the image from [here](https://pop.system76.com/) Make a bootable Pop!_OS drive. (Remark: I only tested with 21.04)
The installation process should work without any problems. If you want to dual boot, you have to make sure that during install you are not using the entire disk. You might need some extra partitions as the installation wizard will guide you.

Since 18.04 Pop!_OS is not utilizing GRUB but rather Systemd-boot, you can just choose the OS to boot by pressing ESC after boot to use the UEFI Menu.

## Post Instalaltion

### Update and Upgrade
Start by installing all updates:
```sh
sudo apt update
sudo apt upgrade
```
After all packages have been upgraded reboot your system.

### Installing the latest kernel
The easiest way for ubuntu based distros is the Ubuntu Mainline Kernel Software. Start by adding the PPA:
```sh
sudo add-apt-repository ppa:cappelikan/ppa
```

Followed by updating the package list and installing the mainline tool

```sh
sudo apt update
sudo apt install mainline
```
Start the mainline tool. For me with kernel 5.13.4 Audio seems to be fixed. So I recommend installing 5.13.4 or newer. Reboot your system, and test if most things are working.

## Driver and other Fixes

Forked from [this repo](https://github.com/CO-1/asus-flow-x13-linux) Might be obsolete with newer kernels.

With small tweaks almost everything (surprisingly touch, pen input and camera) works fine .
Notable exceptions is fingerprint.

While writing on this, power draw from battery (with terminal, vscode and chromium open) is between 6 and 10W.
CPU temperature is ~50C and fans are off.

To quickly install the fixes presented here one could
```sh
sudo sh install.sh
```

And to uninstall them
```sh
sudo sh uninstall.sh
```


### Keyboard
Out of the box keyboard now works.

### GPU

#### AMD iGPU
Works.

#### Nvidia
Nouveau drivers hangs laptop at boot. It can be blacklisted by appending
`nouveau.blacklist=1` to kernel command line or by following commands as root
```sh
echo "blacklist nouveau" >  /etc/modprobe.d/asus-flow-x13-nouveau.conf
echo "alias nouveau off" >> /etc/modprobe.d/asus-flow-x13-nouveau.conf
```

Even if nouveau or nvidia is not loaded nvidia gpu will still consume ~10W of power.
We need to set power/control to auto to reduce power.

```sh
echo '#nvidia dGPU' > /etc/udev/rules.d/99-asus-flow-power.rules
echo 'ACTION=="add", SUBSYSTEM=="pci", TEST=="power/control", ATTR{vendor}=="0x10de", ATTR{power/control}="auto"' >> /etc/udev/rules.d/99-asus-flow-power.rules
udevadm control --reload
```
