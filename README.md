# callingcard

An initcpio module enabling relatively secure full-disk encryption using LUKS and a CCID smart card. Conveniently, this also works quite well with a Yubikey, as that's how I use it.

*NOTE: This package is designed for an Arch Linux installation, as both I and my laptop hate myself.*

## installation

- Set up LUKS for your system, using a password that you'll remember. The password you use to set this up will be kept as a backup. Note that your LUKS disk is only as secure as your least-secure password.
- Generate a 256-byte key for LUKS to use: `dd if=/dev/urandom of=/tmp/<key_name> bs=32 count=1`.
  - Yes, I'm aware that LUKS uses 256-BIT aes-xts and not 256-BYTE aes-xts, but pkcs15-crypt wants 256 BYTES for some reason so it's getting the whole goddamned 256 bytes.
- Encrypt that key with your smartcard: `pkcs15-crypt -w -s -i /tmp/<key_name> -o /tmp/<key_name_enc>`.
- Add your **UNENCRYPTED** key to your LUKS device: `sudo cryptsetup luksAddKey /dev/<disk_name> /tmp/<key_name>`. Then, delete the file.
- Move `callingcard.hook` and `callingcard.install` to `/usr/lib/initcpio/hooks/callingcard` and `/usr/lib/initcpio/install/callingcard`, respectively.
- Add to your bootloader's Linux command line args, defining `root=/dev/mapper/cryptroot` and `callingcard=<properly formatted args> `. In grub this can be done by appending to the `GRUB_CMDLINE_LINUX="<some>"` line in `/etc/default/grub`. Make sure that you re-generate your grub config if you dork with it. For reference, my grub config includes these:
  - `root=/dev/mapper/cryptroot callingcard=/dev/<disk_name>:vfat:rootkey_enc.pkcs1:/dev/<disk_name>`
- Edit `/etc/mkinitcpio.conf`  and edit the hooks line to include `callingcard`. It *must* come before `fsck` and after `filesystems` . For reference, the hooks line in my config looks like this:
  - `HOOKS=(base udev autodetect keyboard consolefont modconf block filesystems callingcard fsck)`
- Rebuild initcpio and your grub config:
  - `mkinitcpio -p linux`
  - `grub-mkconfig -o /boot/grub/grub.cfg`