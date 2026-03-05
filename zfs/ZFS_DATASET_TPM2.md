# Unlock encrypted ZFS dataset with TPM2

You can build a mechanism to automatically unlock an encrypted ZFS dataset using the machines TPM2 device with `systemd-creds`. This allows for unattended restarts while still retaining integrity of the machine. It is assumed you already have ZFS datasets with plaintext passphrases for this guide.

The easiest way to build this is by creating a systemd unit template for your ZFS pool. Here is an example for a pool named `hdd`:

```systemd
[Unit]
Description=Load encryption key from systemd-creds for hdd/%I
DefaultDependencies=no

[Service]
Type=oneshot
RemainAfterExit=yes
LoadCredentialEncrypted=hdd-%i:/var/lib/zfs-keys/hdd-%i.enc
ExecStart=zfs load-key -L file://%d/hdd-%i hdd/%i
StandardInput=tty-force
PrivateMounts=true
Restart=on-failure

[Install]
WantedBy=pve-storage.target
```

This is taken from a Proxmox installation. You can replace the `WantedBy=` with a more fitting target for your machine. Keep in mind that reaching the mentioned target requires a successful run of this unit.

## Creating the encrypted credential file

First, check if your machine supports the full functionality of `systemd-creds`:

```sh
systemd-creds has-tpm2
```

Everything should be checkmarked on modern machines. If you don't have a dedicated hardware TPM2 device, many modern CPUs have a builtin software fTPM you can enable in the UEFI. Unlike many statements on the internet, Secure Boot is not a requirement to use TPM2. UEFI is a requirement.

Set up `systemd-creds` for the first time:

```sh
systemd-creds setup
```

Encrypt the passphrase for your dataset. Replace the `DATASETNAME` according to your setup. This guide uses the pool name `hdd` as a prefix for the secret files to make it more clear to which ZFS pool the secrets belong to.

```sh
systemd-ask-password -n | systemd-creds encrypt --name=hdd-DATASETNAME - /var/lib/zfs-keys/hdd-DATASETNAME.enc
```

Create the unit template in `/etc/systemd/system/load-key-hdd@.service`. Then reload systemd:

```sh
systemctl daemon-reload
```

Unload your ZFS dataset key for a test run:

```sh
zfs unload-key hdd/DATASETNAME
```

Start the templated unit with the correct dataset name:

```sh
systemctl start load-key-hdd@DATASETNAME
```

Check if the dataset is now unlocked:

```sh
zfs get keystatus -r hdd
```

If the key is available, you can enable the unit for autostart on boot and test a system reboot:

```sh
systemctl enable load-key-hdd@DATASETNAME
reboot
```
