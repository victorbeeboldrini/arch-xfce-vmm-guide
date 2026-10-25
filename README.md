# A Guide on How To Install Arch Linux (XFCE4) with VMM

A pedestrian guide on how to install Arch Linux with XFCE4 in a BIOS/MBR VMM virtual machine. This guide is not helpful for a UEFI install nor any bare metal setup, the following steps have been meticulously curated for a VM environment, which comes with many conveniences bundled that eliminate the need of certain packages that would be otherwise crucial at another environment. This guide also assumes you have VMM already installed at your desktop machine.

As for why BIOS/MBR specifically, it is because VMM uses it by default, and I have found myself trampling myself over trying to get the ISO to run as UEFI. 

Essentially, the purpose of this guide is so you can tell yourself that you have installed Arch Linux once (assuming you haven't)

 > [!NOTE] 
 > **Sources Used**
 > 
 > **[EF - Linux Made Simple — Arch Linux Full install on BIOS/MBR with Xfce4 and visual guide](https://www.youtube.com/watch?v=FudOL0-B9Hs)**
 > 
 > **[Arch Linux Wiki — Installation Guide](https://wiki.archlinux.org/title/Installation_guide)**
 


## 1. Pre-installation
---
### 1.1 Acquiring the installation image 

You'll need to install both the ISO (the installation image) and the SIG (the signature) to check if the installation image is safe to use and free from adulteration.

To download both, visit **[https://archlinux.org/download](https://archlinux.org/download)** and scroll down until you get to the HTTP section and mirrors from different countries (they'll be accompanied by their correspondent flag), choose any mirror of your desire and download both the ISO and the SIG (they often look something like archlinux-2026.10.01-x86_64).

Once you've downloaded your installation image, run the following command to check if the ISO corresponds with the signature:

```
$ gpg --verify archlinux-*-x86_64.iso.sig archlinux-*-x86_64.iso
```

If you get a *Good signature*, then your ISO is safe to go.
### 1.2 Setting your virtual machine up

(In case you have yet not installed VMM, visit **https://virt-manager.org/** or browse for Virtual Machine Manger in the software manager of your Linux distribution if available). With VMM open, click on File and then New virtual machine, choose Local install media (though it is selected by default in most cases), at *Choose ISO or CDROM* install media, click on Browse, *Browse Local* and then look after the ISO you just downloaded (e.g.: at the downloads folder), then click on *Choose Volume* and Forward.

Afterwards, it is recommended that you allocate at least 2048 MiB of **Memory**, 2 **CPUs**. In the next page, allocate 35 GiB of storage and click **Forward**. This is the last section of your VM setup, double check if you properly chose your desired specifications and then click on **Finish**.
## 2. Installation

Once your VM is on, you'll see a bootloader, select *Arch install medium* and press ``Enter``.

#### ``root@archiso # ``
---
- timedatectl list-timezones
	- This command will show a list of the world's timezones.
	- It is important that you locate your timezone and note it down, as it'll be necessary later on for syncing your installation with your clock.
- ``(optional)`` localectl list-keymaps
	- I'd recommend you to locate your keymap type and note it down if yours deviate from the default keymap (US), as it'll be necessary later on, otherwise, ignore this step.
- ``(optional)`` loadkeys your_keymap
	- Part of the optional step above, here you'll adjust your current keymap of the live install to the one of your desire (e.g.: br-abnt2, a Brazilian keyboard layout).
- cfdisk
	- With this command, you will create the partition for the installation.
	- Select label type: dos
	- New; ``Enter`` (35g); ``Enter`` (Primary); type ``yes``.
	- Write, type ``yes``.
	- Quit, ``Enter``.
- lsblk
	- Check your newly created partition, it should look something like ``/dev/vda1``.
- mkfs.ext4 /dev/vda1
	- With this command, you will be formating your system partition to ext4, which is the file system for Linux.
- mount /dev/vda1 /mnt
	- With this command, you will mount the partition so you can perform the installation.
- pacstrap /mnt base linux nano
	- Those will be necessary packages we need to install for there to be Linux, though nano is simply a text editor (it will be very handy later on).
- genfstab /mnt >> /mnt/etc/fstab
	- This will generate the file systems table (fstab) for your installation so it can boot up automatically.
- arch-chroot /mnt

#### ``[root@archiso /]#``
---
- ln -sf /usr/share/zoneinfo/Your/Timezone /etc/localtime
	- You should replace ``Your/Timezone`` with your correspondent timezone (e.g.: America/Sao_Paulo).
- hwclock --systohc
	- This command will insert your current time into your (virtual) hardware clock.
- nano /etc/locale.gen
	- ``Ctrl + F``, and then search for **en_US.UTF-8**, upon finding the string most similar to the search, erase the # in it, do not leave any space behind the aforementioned locale aswell.
	- ``Ctrl + O`` to save, ``Enter`` to confirm, and then ``Ctrl + X`` to exit.
- locale-gen
	- This command will generate the locales of your VM.
- nano /etc/locale.conf
	- Type in the string ``LANG=en_US.UTF-8``, write changes with ``Ctrl + O`` and then exit with ``Ctrl + X``.
- ``*optional`` nano /etc/vconsole.conf
	- In case you have a desired keyboard layout, you must type ``KEYMAP=`` followed directly by your desired keymap (e.g.: br-abnt2; de_latin1; etc). Save the changes with ``Ctrl + O`` and exit with ``Ctrl + X``.
- nano /etc/hostname
	- Type in the desired hostname of your machine, e.g.: arch, then save the changes with ``Ctrl + O`` and exit with ``Ctrl + X``.
- nano /etc/hosts
	- There should be some pre generated strings already, however, in case all the strings for the IPs are missing (the third one is often always missing), add in the following strings: ``127.0.0.1 localhost``; ``::1 localhost``; ``127.0.1.1 your_hostname.localdomain your_hostname``. Afterwards, save the changes with ``Ctrl + O`` and exit with ``Ctrl + X``.
- passwd
	- Type in your desired password, make sure you remember it.
- pacman -S grub networkmanager network-manager-applet sudo qemu-guest-agent
	- Proceed each prompt with ``Enter``, and then confirm with Y.
	- For transparency, grub is your bootloader; the packages beginning with network will guarantee your VM internet connection; sudo will be important for managing root privileges for non-root users; qemu-guest-agent is going to make the VM more convenient to use. This is a gross oversimplification, do not take it at face value.
- grub-install --target=i386-pc /dev/vda
- grub-mkconfig -o /boot/grub/grub.cfg
- exit

#### ``root@archiso # ``
---
- umount -a
	- Do not mind the error messages, they are expected.
- reboot

## 3. Post-installation

#### ``root@arch # ``
---
- Log in as root and type in your password
- systemctl start NetworkManager
- systemctl enable NetworkManager.service
- ``(optional)`` ping -c 4 8.8.8.8
	- Use this command to send 4 packets of data to Google Public DNS, if you receive data back, it means you have internet connection.
- ``(optional)`` nmtui
	- In case there isn't internet, you can type in this command, open Activate connection, then hover on Wired connection 1, if it is written ``<Activate>``, press ``Enter`` on the aforementioned connection so it shows up as ``<Deactivate>``, go Back, then Quit.
- useradd -m -G wheel your_username
	- You are free to choose any username you desire, it does not need to be the same one as the hostname.
- passwd your_username
	- Type in your desired password.
- EDITOR=nano visudo
	- Scroll down until you find ``## Uncomment to allow all members of group wheel to execute any command``, erase the ``#`` of the string below, do not leave empty space behind it. Press ``Ctrl + O``, confirm with ``Enter`` to write changes, then ``Ctrl + X`` to exit.
- pacman -S xorg
	- Proceed all prompts with ``Enter`` and confirm installation by typing Y.
	- This will install the display server of your OS, it is crucial for you to actually see and interact with your desktop.
- pacman -S lightdm lightdm-gtk-greeter
	- This will install the graphical login manager (LightDM) of your OS.
- systemctl enable lightdm
	- This will enable said login manager.
- pacman -S xfce4 xfce4-goodies pipewire pipewire-pulse wireplumber xdg-user-dirs
	- Proceed all prompts with ``Enter`` and confirm installation by typing Y.
	- This will install your lightweight Desktop Environment (XFCE), the servers responsible for handling audio and the user folders (Downloads, Pictures, etc).
- reboot

#### ``your_username@arch``
---
- Once you log in, there is one crucial step (it is optional). 
	- Open the terminal by either pressing ``Ctrl + Alt + T`` or by clicking on the dock icon, then, type in ``pacman -S fastfetch``, proceed any prompts that may appear with ``Enter`` and confirm with Y.
	- Type in clear on the terminal to get rid of the clutter, then, type in fastfetch.
	- You may now screenshot, or ponder, the very thing **you** have just created, following this oversimplified guide does not take any merit what you've done.
- In case you want an internet browser, you can install FireFox with ``sudo pacman -S firefox``.
- In case you want to keep using this VM, you can keep it up to date with ``pacman -Syu``.
