### 1. Back up your data
Before making any changes to partitions or bootloaders, **back up important files** to an external drive or cloud storage.
### 2. Disable BitLocker (Windows)
Disable encryption in case something goes wrong during partitioning or bootloader setup:  
**Settings → Privacy & Security → Device Encryption**  
Turn off BitLocker/device encryption and wait for decryption to finish.

### 3. Download Ubuntu ISO
Download the correct Ubuntu image:
- For Viper: **[Ubuntu 22.04 LTS](https://releases.ubuntu.com/jammy/)**
    
Save the ISO somewhere easy to find.

### 4. Create a bootable USB & start installer
Follow the **official Ubuntu guide** to create a bootable USB and start installation:
- [Official installation guide](https://documentation.ubuntu.com/desktop/en/latest/tutorial/install-ubuntu-desktop/#create-a-bootable-usb-stick)

Restart your computer and boot from the USB (usually via **F12**, **ESC**, or BIOS boot menu).

When the Ubuntu menu appears, choose:  
**“Try Ubuntu”** (this loads the live environment without installing yet).

### 5. Create space for Ubuntu
Ubuntu needs its own empty partition.
You have two main options:
#### Option A — From Windows (simplest)
1. Press **Win + X → Disk Management**
2. Right-click your main drive (usually C:)
3. Select **Shrink Volume**
4. Allocate at least **40–100 GB** free space for Ubuntu
5. Leave this space **unallocated**

**Limitation:** Windows Disk Management can only shrink partitions if free space is at the _end_ of the drive.  
It **cannot move partitions or rearrange them**, so sometimes you won’t be able to shrink as much as expected.
#### Option B — Using GParted (more flexible)

From the Ubuntu live USB:
1. Click **Try Ubuntu**
2. Open **GParted Partition Editor** (preinstalled in most live environments)
3. Select your main disk (usually `/dev/nvme0n1` or `/dev/sda`)
4. You can:
    - Shrink partitions
    - Move partitions
    - Rearrange space
    - Create new partitions

Unlike Windows tools, **GParted can move partitions around the disk**, not just shrink from the end.  
This lets you free up space even if Windows can’t.

⚠️ Moving partitions takes time and carries some risk if interrupted — do not power off during the process.

After shrinking, leave free space unallocated for Ubuntu.

### 6. Install Ubuntu

From the live desktop, click **Install Ubuntu** and follow prompts.
When asked about installation type:
- Choose **Install Ubuntu alongside Windows Boot Manager** (easiest), OR
- Choose **Something Else** if manually partitioning

Suggested manual partition setup:
- `/` (root): 30–50 GB (ext4)
- `swap`: 2–8 GB (optional if you have lots of RAM)
- `/home`: remaining space (ext4, optional but recommended)

### 7. Finish & reboot
Once installation completes:
1. Restart your computer
2. Remove the USB when prompted
3. You should see the **GRUB boot menu** letting you choose:
    - Ubuntu
    - Windows Boot Manager

### 8. After install (recommended)
Update Ubuntu:
`sudo apt update && sudo apt upgrade`
After confirming everything works, you can re-enable BitLocker in Windows if desired.

---

**Done.**  
You now have a dual-boot Ubuntu + Windows system, selectable each time you start your computer.