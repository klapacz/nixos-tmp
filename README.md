# NixOS reinstall (dual-boot, LUKS + LVM + hibernation)

This repo is a flake-based NixOS configuration. The steps below reinstall NixOS on `/dev/sda5` with LUKS + LVM, keep Windows intact, and use the existing EFI partition at `/dev/sda6`.

## Installation (from NixOS live USB)

### 1) Prepare encrypted LUKS + LVM
```
cryptsetup luksFormat /dev/sda5
cryptsetup open /dev/sda5 cryptroot

pvcreate /dev/mapper/cryptroot
vgcreate vg /dev/mapper/cryptroot
lvcreate -L 32G -n swap vg
lvcreate -l 100%FREE -n root vg
```

### 2) Format and mount
```
mkfs.ext4 /dev/vg/root
mkswap /dev/vg/swap

mount /dev/vg/root /mnt
mkdir -p /mnt/boot
mount /dev/sda6 /mnt/boot
swapon /dev/vg/swap
```

### 3) Clone this flake
```
nix-shell -p git

git clone https://github.com/<you>/<repo>.git /mnt/etc/nixos
```

### 4) Generate hardware config (recommended)
```
nixos-generate-config --root /mnt
```

### 5) Update UUIDs in configuration
Get UUIDs:
```
blkid
```

Edit `/mnt/etc/nixos/configuration.nix` and replace:
- `UUID_OF_LUKS_PARTITION` with the UUID of `/dev/sda5`
- `UUID_OF_EFI_PARTITION` with the UUID of `/dev/sda6`

### 5) Install
```
nixos-install --flake /mnt/etc/nixos#nixos
```

## Notes
- Windows uses `/dev/sda1` (EFI) and other NTFS partitions; do not modify them.
- Hibernation uses the LVM swap at `/dev/vg/swap`.
