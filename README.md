# My proxmox related tests

Replacing lab based ESXI by PVE (https://proxmox.com/en/proxmox-virtual-environment/overview)

# Purpose:
- unattended USB key installation (embedded answer file/fetch online)
- unattended PXE installation (embedded answer file/fetch online)

# Answer files options
- keymap/lang: US|FR
- password: `changemenow`
- ssh-key: none| (passwordless generated for testing purpose: you want to put your own)
- disk setup: (single disk: sda|nvme0n1, with ext4|xfs|lvm|zfs), (dual disks:raid1 or zfs mirror) 
- boot loader:  MBR(legacy bootin) or UEFI
- custom partitionning (mdadm mirror for /boot, swap /), zfs (mirror), ceph OS, ...?
- named user? w/wo ssh_keys?
- ssh by keys only?

# unattended USB key with embedded file
- fetch the GA iso image
- create an answer.toml file
- generate the new iso image and transfer to the usb key
` proxmox-auto-install-assistant prepare-iso /path/to/source.iso --fetch-from iso --answer-file /path/to/answer.toml`

```
wget https://enterprise.proxmox.com/iso/proxmox-ve_8.2-2.iso
cat <<EOF1> fr-nvme-zfs.toml
[global]
keyboard = "fr"
country = "fr"
fqdn = "pveauto.testinstall"
mailto = "tru@pasteur.fr"
timezone = "Europe/Paris"
root_password = "changemenow"

[network]
source = "from-dhcp"

[disk-setup]
filesystem = "zfs"
zfs.raid = "raid0"
zfs.hdsize = 150
disk_list = ["nvme0n1"]
EOF1
cat <<EOF2> fr-nvme-xfs.toml
[global]
keyboard = "fr"
country = "fr"
fqdn = "pveauto.testinstall"
mailto = "tru@pasteur.fr"
timezone = "Europe/Paris"
root_password = "changemenow"

[network]
source = "from-dhcp"

[disk-setup]
filesystem = "xfs"
disk_list = ["nvme0n1"]
EOF2
proxmox-auto-install-assistant prepare-iso proxmox-ve_8.2-2.iso --fetch-from iso --answer-file fr-nvme-xfs.toml
sudo dd if= of=/dev/sdxxxx bs=8M status=progress
```
# unattended USB key with url
- fetch the GA iso image
- generate the new iso image and transfer to the usb key
```
[ -f proxmox-ve_8.2-2.iso ] || wget https://enterprise.proxmox.com/iso/proxmox-ve_8.2-2.iso
proxmox-auto-install-assistant prepare-iso proxmox-ve_8.2-2.iso --fetch-from http --url "https://raw.githubusercontent.com/truatpasteurdotfr/pve-setup/refs/heads/main/fr-nvme-zfs.toml" --cert-fingerprint "FD:6E:9B:0E:F3:98:BC:D9:04:C3:B2:EC:16:7A:7B:0F:DA:72:01:C9:03:C5:3A:6A:6A:E5:D0:41:43:63:EF:65"
sudo dd if= of=/dev/sdxxxx bs=8M status=progress
```
# PXE boot
- convert the customised iso with `pve-iso-2-pxe`
- use ipxe/.. to provide the kernel and initrd (MBR or UEFI)

# References:
- https://pve.proxmox.com/pve-docs/
- https://pve.proxmox.com/wiki/Automated_Installation
- https://github.com/morph027/pve-iso-2-pxe
- https://git.proxmox.com/?p=pve-installer.git;a=summary
