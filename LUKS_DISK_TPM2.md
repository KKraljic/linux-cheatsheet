# Encrypting a disk with LUKS2 and unlock with TPM2

This guide assumes you want to setup LVM on your LUKS2 disk. If you want to build something different on top of LUKS2, just skip the LVM steps.

Create your LUKS2 volume and set a mapping for it:

```sh
cryptsetup luksFormat --type luks2 /dev/DEVICE
cryptsetup luksOpen /dev/DEVICE MAPPING_NAME
```

Set up LVM2:

```sh
pvcreate /dev/mapper/MAPPING_NAME
vgcreate VG_NAME /dev/mapper/MAPPING_NAME
lvcreate -l 100%FREE VG_NAME -n LV_NAME
lvconvert --type thin-pool VG_NAME/LV_NAME
```

Enable automatic decryption with TPM2 on boot:

```sh
# Add --tpm2-with-pin if you want some more security for bootable volumes
# --tpm2-pcrs "0+7" for basic security is good
systemd-cryptenroll --tpm2-device auto /dev/DEVICE
```

Add this line to `/etc/crypttab`:

```sh
MAPPING_NAME    /dev/DEVICE    none    tpm2-device=auto
```

Try to reboot your system to check if automatic decryption works.

## Troubleshooting

Check if your system has a compatible TPM2:

```sh
systemd-analyze has-tpm2
# if verb not found
systemd-creds has-tpm2
```

You might need to install `tpm2-tools`:

```sh
apt install tpm2-tools
dnf install tpm2-tools
```
