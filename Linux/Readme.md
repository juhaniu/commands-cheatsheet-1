# Useful Linux commands

### Create file what will utlize disk
```
xfs_mkfile 10240m 10Gigfile
```


### Format the filesystem with LUKS and add extra password to slot1 
```
cryptsetup luksFormat --cipher aes-xts-plain64 /dev/mapper/lvm_pool_data1-lvol001
cryptsetup luksOpen /dev/mapper/lvm_pool_data1-lvol001 es_data
cryptsetup luksAddKey --key-slot 1 /dev/mapper/lvm_pool_data1-lvol001 
mkfs.xfs /dev/mapper/es_data
```

### Create part/VG/LV/FS 
```
parted /dev/sdb mklabel msdos
parted /dev/sdb mkpart primary 1M 100% set 1 lvm on
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
