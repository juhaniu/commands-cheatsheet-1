# Useful Linux commands

## Partitions, LVM, LUKS, FS etc

### Create file that will utilize space from storage
```
xfs_mkfile 10240m 10Gigfile
```


### LUKS: Format the filesystem with LUKS and add extra password to slot1 
```
cryptsetup luksFormat --cipher aes-xts-plain64 /dev/mapper/lvm_pool_data1-lvol001
cryptsetup luksOpen /dev/mapper/lvm_pool_data1-lvol001 es_data
cryptsetup luksAddKey --key-slot 1 /dev/mapper/lvm_pool_data1-lvol001 
mkfs.xfs /dev/mapper/es_data
```

### LUKS: Resize VG/LV with LUKS
```
parted /dev/sdc mklabel msdos mkpart primary 1M 100% set 1 lvm on
pvcreate /dev/sdc1
vgextend vg_crypt /dev/sdc1
lvextend -L +10G  /dev/mapper/vg_crypt-lv_crypt
  Size of logical volume vg_crypt/lv_crypt changed from 11.00 GiB (2816 extents) to 21.00 GiB (5376 extents).
  Logical volume lv_crypt successfully resized.
 
cryptsetup resize /dev/mapper/vg_crypt-lv_crypt
xfs_growfs /dev/mapper/crypted_lvm
```

### Create part/VG/LV/FS 
```
parted /dev/sdb mklabel msdos mkpart primary 1M 100% set 1 lvm on
pvcreate /dev/sdb1
vgcreate vg_pgsql /dev/sdb1
lvcreate -L 20G -n lv_pgsql vg_pgsql
mkfs.xfs /dev/mapper/vg_pgsql-lv_pgsql
```
### Growpart if disk was increased 
```
growpart /dev/sda 1
pvscan
lvscan
lvextend -l 100%FREE /dev/mapper/lvm_pool_data1-lvol001
xfs_growfs /dev/mapper/lvm_pool_data1-lvol001
```

### Deactive/Active LV
```
lvchange -an /dev/lvm_pool_data1/lvol001
lvchange -ay /dev/lvm_pool_data1/lvol001
```

### Mounting a Windows fileshare (tested with RHEL6)
```
Manually:
mount -t cifs -o username=domain/user //ms-hostname/winfolder$ /mnt/

Fstab entry:
//ms-hostname/winfolder$ /mnt/  cifs  username=domain/user,password=SomePasswords,iocharset=utf8  0  0
```

### Find dublicates in fstab
```
grep -vE "^#"  /etc/fstab  | awk  '{print $2}' |  uniq -d
```

## Text manipulation with SED, AWK

### SED: Insert line at very end

```
# Add tmp as in-memory entry to fstab
sed -i -e '$atmpfs /tmp tmpfs strictatime,noexec,nodev,nosuid 0 0' /etc/fstab```
```

### SED: Insert line before match found
```
sed -i 's/.*plugins=ifcfg-rh,ibft.*/dns=none\n&/' /etc/NetworkManager/NetworkManager.conf
```

### Find + SED: Replacing values in multiple files inside directory (in MAC)
```
find foldername -type f -exec grep -H 'hostname' {} \;
find foldername -type f -name "*.yaml" -exec sed -i '' 's/hostname.local/new-hostname.local/g' {} \;
```

### Find + SED: Replacing values in multiple files inside directory (in Linux)
```
find foldername/*/some-other/ -type f -name "*.yaml" -exec grep -H 'hostname' {} \;
find foldername/*/some-other/ -type f -name "*.yaml" -exec sed -i 's/hostname.local/new-hostname.local/g' {} \;
```


### AWK: List of all locked accounts (accounts with passwords) :
```
awk -F: '{ system("passwd -S " $1)}' /etc/passwd | grep " LK "
```
### AWK: List of all unlocked accounts (accounts with passwords) :
```
awk -F: '{ system("passwd -S " $1)}' /etc/passwd | grep " PS "
```
