# 🧱 4TB Drive Expansion Log
## 🛠️ Hardware & Partition Setup
Added a second EN870 NVMe drive, this time with 4TB capacity.

Verified detection with: `lsblk`

Initialized the drive: `sudo fdisk /dev/nvme1n1` Created a new GPT partition table, and added a partition of type 'Linux filesystem'.

Formatted the partition: `sudo mkfs.ext4 /dev/nvme1n1p1`

## 📁 Mounting & Samba Integration
Checked Samba config: `cat /etc/samba/smb.conf`

Confirmed main share (MyNASShare) is located at /srv/media/share.

Created a new subdirectory: `mkdir /srv/media/share/Media4T`

Mounted the new drive: `sudo mount /dev/nvme1n1p1 /srv/media/share/Media4T`

Made the mount persistent: Verified UUID with `sudo blkid /dev/nvme1n1p1`, then added to /etc/fstab: `UUID=your-uuid-here /srv/media/share/Media4T ext4 defaults,noatime 0 2`

Rebooted and confirmed with lsblk — 3.6TB mounted successfully at /srv/media/share/Media4T. 🎉

## 👤 Permissions & File Migration
Updated ownership: `sudo chown -R user:user /srv/media/share`

Transferred video folder via Samba through my desktop file manager — worked, but slow and limited to copy-paste.

Used CLI for remaining directories: `mv -v <source> <destination>`
Much faster, and I strongly recommend using -v for progress visibility.

### ✅ Streaming test: success!

## 🔄 Migrating from Synology NAS
📦 Initial Transfer

Moved old backup game files via desktop File Manager — functional but slow due to the transfer being funneled through the SMB connection on my desktop.

🔗 Mounting Synology Share

Created mount point: `mkdir /mnt/synology`

Attempted direct mount with: `sudo mount -t cifs //synology-ip/share-name /mnt/synology -o username=your-user,password=your-password,vers=3.0`
* This didn't work, and I didn't like my Synology username and password being exposed on my bash history like that, so I removed that line from the bash history, and tried something else.

Switched to credentials file method:

I created a credentials file in /tmp/smb-credentials' with the contents:  
username=your-user  
password=yourpassword

Then successfully mounted with: `sudo mount -t cifs "//192.168.x.x/share-name" /mnt/synology -o credentials=/tmp/smb-credentials,vers=3.0`

Mounted only the video folder initially.

## 🚀 Rsync Transfer
Used: `rsync -avh --progress /mnt/synology/ /srv/media/share/Media4T/`

Transfer speeds: ~100–111 MB/s — Much faster than using my desktop!

Unmounted video folder (`umount /mnt/synology`), then mounted music folder and repeated the rsync process. Continued this process till all contents were moved over.

## ✅ Status & Next Steps
Everything is working smoothly. Now I just need to decide how long to keep the Synology NAS as a fallback before retiring it.
