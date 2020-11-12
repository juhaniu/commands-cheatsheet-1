# Useful Linux commands

### Create file what will utlize disk
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
xfs_growfs
```

### Deactive/Active LV
```
lvchange -an /dev/lvm_pool_data1/lvol001
lvchange -ay /dev/lvm_pool_data1/lvol001
```

### Mounting a MS Fileshare 
```
Manually:
mount -t cifs -o username=domain/user //ms-hostname/winfolder$ /mnt/

Fstab entry:
//ms-hostname/winfolder$ /mnt/  cifs  username=domain/user,password=SomePasswords,iocharset=utf8  0  0
```
 
