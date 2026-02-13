The "LVM Expander" Script

Save this code as expand_lvm.sh and give it execute permissions: chmod +x expand_lvm.sh.

```
#!/bin/bash

# --- Configuration ---
DISK="/dev/sda"
# You can customize these or let the script prompt you
VG_NAME=""
LV_PATH=""

# Ensure the script is run as root
if [[ $EUID -ne 0 ]]; then
   echo "This script must be run as root."
   exit 1
fi

echo "--- Starting LVM Expansion on $DISK ---"

# 1. Identify Volume Group and Logical Volume if not set
if [ -z "$VG_NAME" ]; then
    echo "Current Volume Groups:"
    vgs --noheadings -o vg_name
    read -p "Enter the Volume Group (VG) name to expand: " VG_NAME
fi

if [ -z "$LV_PATH" ]; then
    echo "Current Logical Volumes in $VG_NAME:"
    lvs $VG_NAME --noheadings -o lv_path
    read -p "Enter the full LV Path to expand (e.g., /dev/mapper/ubuntu--vg-root): " LV_PATH
fi

# 2. Create the new partition
echo "Creating new partition on $DISK..."
# This sends commands to fdisk: 
# n (new), p (primary), default number, default start, default end, 
# t (type), [last partition number], 30 (LVM), w (write)
NEW_PART_NUM=$(parted $DISK print | grep -c "^ [0-9]")
let NEW_PART_NUM++

sed -e 's/\s*\([\+0-9a-zA-Z]*\).*/\1/' << EOF | fdisk $DISK
  n # new partition
  p # primary
    # partition number (default)
    # first cylinder (default)
    # last cylinder (default)
  t # change type
    # partition number (default)
  30 # Linux LVM type
  w # write and quit
EOF

NEW_PART="${DISK}${NEW_PART_NUM}"
echo "New partition created: $NEW_PART"

# 3. Inform Kernel
echo "Reloading partition table..."
partprobe $DISK
sleep 2

# 4. LVM Operations
echo "Initializing Physical Volume..."
pvcreate $NEW_PART

echo "Extending Volume Group $VG_NAME..."
vgextend $VG_NAME $NEW_PART

echo "Extending Logical Volume $LV_PATH..."
lvextend -l +100%FREE $LV_PATH

# 5. Resize Filesystem
FS_TYPE=$(blkid -o value -s TYPE $LV_PATH)
echo "Detected Filesystem: $FS_TYPE"

if [ "$FS_TYPE" == "xfs" ]; then
    MOUNT_POINT=$(lsblk -no MOUNTPOINT $LV_PATH)
    xfs_growfs $MOUNT_POINT
elif [ "$FS_TYPE" == "ext4" ] || [ "$FS_TYPE" == "ext3" ]; then
    resize2fs $LV_PATH
else
    echo "Unknown filesystem type. Manual resize may be required."
fi

# 6. Final Report
echo "------------------------------------------------"
echo "RESUME REPORT FOR SYSTEM ADMINISTRATOR"
echo "------------------------------------------------"
echo "Date: $(date)"
echo "Disk Modified: $DISK"
echo "New Partition: $NEW_PART"
echo "VG Extended:   $VG_NAME"
echo "LV Extended:   $LV_PATH"
echo "New LV Size:   $(lvs --noheadings -o lv_size $LV_PATH | xargs)"
echo "Filesystem:    $FS_TYPE"
echo "Status:        Success"
echo "------------------------------------------------"
```
How this script is "Smart":

  Dynamic Partition Detection: It calculates the next available partition number so it doesn't overwrite existing data.
  Filesystem Awareness: It checks if you are using XFS or Ext4. This is critical because resize2fs will fail on XFS, and xfs_growfs requires a mount point rather than a device path.
  Automation of fdisk: Using a sed redirected block allows the script to "type" into the fdisk prompt automatically.
  Admin Summary: It provides a clean, copy-pasteable report at the end for your logs.

A Note of Caution

While this script is designed to be safe, disk operations always carry risk. * Always ensure you have a backup of critical data before running.

If your disk uses a GPT partition table (common in UEFI systems) instead of MBR, you might need to use sgdisk instead of fdisk.
