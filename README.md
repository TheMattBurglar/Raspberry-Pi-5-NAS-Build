# 🧰 Raspberry Pi 5 NAS Build

A compact, energy-efficient home NAS built on Raspberry Pi 5, running Pi OS Lite (Trixie), with a 1TB NVMe SSD and a full stack of services for media, sync, DNS filtering, and remote access.

---

## ⚙️ Hardware

- **Raspberry Pi 5**  
- **Active Cooler for Raspberry Pi 5 (H505 or similar)**
- **Geekworm X1011 NVMe board**  
- **Geekworm X1011-C1 PCIe Metal Case**
- **CanaKit 45W PD Power Supply (might need more headroom than the standard 27W PSU if I fully populate the X1011 with 4 NVMe drives)**
- **Ediloca EN870 1TB NVMe SSD**  
- **Ethernet connection (via switch)**  
- **Surge-protected power source (UPS preferably)**

---

## 🧱 OS & Boot Setup

- Imaged Pi OS Bookworm to NVMe using Raspberry Pi Imager and a USB to M.2 NVMe Drive Enclosure
  - Note: NVMe SSDs are far more reliable than SD cards.  It may be enticing to use an SD card for the OS, and then NVMe SSDs just for storage, but trust me, it's better to not use SD cards at all.  When the OS drive starts to get corrupted (and SD cards do that a lot), they spread that corruption around to the other drives.  This setup has more than enough slots for NVMe drives.  Use one as an OS drive, if you want modularity.
- Changed default username and hostname  
- Enabled SSH using public-key authentication only
- Assembled hardware, installed NVMe into Pi 5 and booted  
- SSH’d in via local IP  
- Ran `apt update && apt upgrade`  
- Upgraded EEPROM: `sudo rpi-eeprom-update`  
- Changed boot order: NVMe → USB → SD via `raspi-config`  
- Upgraded OS from Bookworm to Trixie
  - [Upgrade guide](https://forums.raspberrypi.com/viewtopic.php?t=389477)
  - Note: the `rpd-wayland-all+ rpd-x-all+` part of the `sudo apt full-upgrade...` command isn't needed in a headless setup

---

## 🛡️ Network Services

### Pi-Hole
- Installed via: `curl -sSL https://install.pi-hole.net | bash`  
- Updated gravity, changed default password  
- Set router DNS to point to Pi-Hole

### PiVPN
- Installed via: `curl -L https://install.pivpn.io | bash`  
- Changed default port to bypass work Wi-Fi restrictions  
- Forwarded port on router  
- Added phone and laptop as VPN clients  
- Resolved IPv6 leak by reinstalling PiVPN and reconfiguring clients

### SSH
- Desktop public-key authentication was setup by Raspberry Pi Imager, but I needed to manually setup public-key authentication for my laptop as well

---

## 📁 File & Media Sharing

### Samba
- Installed: `sudo apt install samba samba-common-bin -y`  
- Replaced default `smb.conf` with more secure config:
  ```ini
  [global]
  workgroup = WORKGROUP
  security = user
  min protocol = SMB2
  usershare allow guests = no
  wide links = no

  [MyNASShare]
  comment = PiNAS
  path = /home/<user>/share
  browseable = yes
  writeable = yes
  guest ok = no
  read only = no
  create mask = 0775
  directory mask = 0775
  force user = <user>
- Avoided insecure chmod 777 suggestions from Gemini
- Eventually changed 'path=' section to different location on drive than is seen in this config due to MiniDLNA permissions issue


### MiniDLNA
- Installed via apt

- Configured /etc/minidlna.conf to point to media folders

- Set user=minidlna to avoid root execution

- Moved share folder out of /home/<user> to resolve permission issues

- Successfully streamed media to Xbox


---
## 🔄 Syncing

### Syncthing
- Installed via apt

- In order to allow LAN access to browser based GUI, updated config file at: ~/.local/state/syncthing/config.xml

- Added Syncthing Tray via Flatpak on desktop and laptop for sync notifications
---

## ✅ Status

NAS is fully operational with:

- DNS filtering (Pi-Hole)

- Remote access (PiVPN)

- File sharing (Samba)

- Media streaming (MiniDLNA)

- File sync (Syncthing)

---
## 🧠 Lessons Learned

- Avoid Phison NVMe controllers for better thermals, firmware stability, and compatibility with Geekworm X1011

- Replacing default config files is sometimes more secure than merely editing them

- Make sure IPv6 doesn't bypass VPN setup

- Permissions in /home/<user\> are restrictive by default, so storing media files in another location is useful to allow minidlna to stream them
- It's also not a bad idea to have your shared folder in a different location than /home for security reasons

- Gemini sometimes suggests insecure defaults — always double-check

