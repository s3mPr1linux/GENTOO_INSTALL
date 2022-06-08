![](contrib/screenshot_configure.png)

## About gentoo-install

This project aspires to be your favourite way to install gentoo.
It aims to provide a convenient way of installing gentoo, both for beginners and experts.
You may configure it by using a menuconfig-inspired interface or simply via a config file.

It supports the most common disk layouts, different file systems like ext4, ZFS and btrfs as well
as additional layers such as LUKS or mdraid. It also supports both EFI (recommended) and BIOS boot,
and can be used with systemd or OpenRC as the init system. SSH can also be configured to allow using an automation framework
like [Ansible](https://github.com/ansible/ansible) or [fora](https://oddlama.gitbook.io/fora/) to automate beyond system installation.

- [#Usage](Usage)
- [#Modern recommendations](Modern recommendations)
- [#Features](Features)
- [#FAQ](FAQ)

## Why?

This project might appeal to you if

- you want to try gentoo without initially investing a lot of time, or fully committing to it yet.
- you already are a gentoo expert but want an automatic and repeatable best-practices installation.

Of course we do encourage everyone to install gentoo manually. You will learn a lot if you
haven't done it already.

## Usage

First, boot into a live environment of your choice. I recommend using an [Arch Linux](https://www.archlinux.org/download/) live ISO,
as the installer will then be able to automatically download required programs or setup ZFS support on the fly.
Afterwards, proceed with the following steps:

1. Either clone this repo or download and extract a copy
2. Run `./configure` and save your desired configuration
3. Begin installation using `./install`

Every option is explained in detail in `gentoo.conf.example` and in the help menus of the TUI configurator.
When installing, you will be asked to review the partitioning before anything critical is done.

The installer should be able to run without any user supervision after partitioning, but depending
on the current state of the gentoo repository you might need to intervene in case a package fails
to emerge. The critical commands will ask you what to do in case of a failure. If you encounter a
problem you cannot solve, you might want to consider getting in contact with some experienced people
on [IRC](https://www.gentoo.org/get-involved/irc-channels/) or [Discord](https://discord.com/invite/gentoolinux).

## Overview

The installer performs the following main steps (in roughly this order),
with some parts depending on the chosen configuration:

1. Partition disks (highly dependent on configuration)
2. Download and extract stage3 tarball (with cryptographic verification)
   (Continue in chroot from here)
3. Setup portage (initial rsync/git sync, run mirrorselect, create zz-autounmask files)
4. Base system configuration (hostname, timezone, keymap, locales)
5. Install required packages (git, kernel, ...)
6. Make system bootable (generate fstab, build initramfs, create efibootmgr/syslinux boot entry)
7. Ensure minimal working system (automatic wired networking, install eix, set root password)

   - (Optional) Install sshd with secure config (no password logins)
   - (Optional) Install additional packages provided in config

The goal of the installer is just to setup a minimal gentoo system following best-practices.
Anything beyond that is considered out-of-scope (with the exception of configuring sshd).
Here are some things that you might want to consider doing after the system installation is finished:

1. Read the news with `eselect news read`.
2. Compile a custom kernel and remove `gentoo-kernel-bin`
3. Adjust `/etc/portage/make.conf`
   - Set `CFLAGS` to `-O2 -pipe <march_native_flags>` for native builds by useing the `resolve-march-native` tool
   - Set `CPU_FLAGS_X86` using the `cpuid2cpuflags` tool
4. Use a safe umask like `umask 077`

### (Optional) sshd

The script can provide a fully configured ssh daemon with reasonably good security settings.
It will by default only allow ed25519 keys, restrict key exchange
algorithms to a reasonable subset, disable any password based authentication,
and only allow root to login.

You can provide keys that will be written to root's `.ssh/authorized_keys` file. This will allow
you to directly continue your setup with your favourite infrastructure management software.

### (Optional) Additional packages

You can add any amount of additional packages to be installed on the target system.
These will simply be passed to a final `emerge` call before the script is done,
where autounmasking will also be done automatically. It is recommended to keep
this to a minimum, because of the quite "interactive" nature of gentoo package management ;)

## Updating the kernel

By default, the installed system uses gentoo's binary kernel distribution (`sys-kernel/gentoo-kernel-bin`)
together with an initramfs generated by dracut. This ensures that the installed system works on all common hardware configurations.
Feel free to replace this with a custom built kernel (and possibly remove/adjust the initramfs) when the system is booted.

The installer will provide the convenience script `generate_initramfs.sh` in `/boot/efi/`
or `/boot/bios` which may be used to generate a new initramfs for the given kernel version.
Depending on whether your system uses EFI or BIOS boot, you will also find your kernel and initramfs in different locations:

```bash
# EFI
kernel="/boot/efi/vmlinuz.efi"
initrd="/boot/efi/initramfs.img"

# BIOS
kernel="/boot/efi/vmlinuz.efi"
initrd="/boot/efi/initramfs.img"
```

In both cases, the update procedure is as follows:

1. Emerge new kernel
2. `eselect kernel set <kver>`
3. Backup old kernel and initramfs (`mv "$kernel"{,.bak}`, `mv "$initrd"{,.bak}`)
4. Generate new initramfs for this kernel `generate_initramfs.sh <kver> "$initrd"`
5. Copy new kernel `cp /boot/vmlinuz-<kver> "$kernel"`

## Modern recommendations

TODOOOOoo 

Below are some recommendations for setting up a modern system.
Please keep in mind that these are based on my (@oddlama's) personal opinions, but I've tried
my best to explain the rationale behind those decisions. Still, your mileage may vary.
I'll keep this project updated to This project will be updated to reflect my c
After all, these are just recommendations.

- kernel (bin vs own)
- UUIDs
- EFI
- ZFS
- systemd
- encrypted system root
- efistub
- swap
- zstd compression

## Troubleshooting and FAQ

TODO the installer can chroot.

After the initial sanity check, the script should be able to finish unattendedly.
But given the unpredictability of future gentoo versions, you might still run into issues

The script checks every command for success, so if anything fails during installation,
you will be given a proper message of what went wrong. Inside the chroot,
most commands will be executed in a checked loop, and allow you to interactively
fix problems with a shell, to retry, or to skip the command.

#### Q: I get errors after partitioning about blkid not being able to find a UUID

**A:** Use `wipefs -a <DEVICE>` on your partitions or fully wipe the disk before use.
The new partitions probably align with previously existing partitions that had
filesystems on them. Some filesystems signatures like those of ZFS can coexist with
other signatures and may cause blkid to find ambiguous information.

## References

* [Sakaki's EFI Install Guide](https://wiki.gentoo.org/wiki/Sakaki%27s_EFI_Install_Guide)
* [Gentoo AMD64 Handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64)
