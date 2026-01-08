# Khan_Lubuntu24.04 LTS
---

## A Fully Automated Linux Learning Environment for Students
## 🚀 GET STARTED IN SECONDS

**All you need to do is:**

1. Download `setup.bat` from this repository
2. Put it in any folder you want
3. Double-click to run

**That's it!** The script handles everything else automatically.

---


---

## 🔐 LOGIN CREDENTIALS

| Field | Value |
|-------|-------|
| **Username** | student |
| **Password** | **123** |

> ⚠️ **Save this!** You'll need it to login to the virtual machine.

---

## 📁 IMPORTANT: FILE LOCATIONS

### VirtualBox Installation
💡*Important*: Virtual box 7.2.4 will be installed automatically if it is not present at the default download location `C:\Program Files\Oracle\VirtualBox\`

| Item | Location |
|------|----------|
| **VirtualBox Installed At** | `C:\Program Files\Oracle\VirtualBox\` |
| **Installed Automatically?** | ✅ Yes, if not already present |

### VDI File (Virtual Disk)
| Item | Location |
|------|----------|
| **VDI Downloaded To** | Same folder where you put `setup.bat` |
| **Example** | If `setup.bat` is in `D:\MyFolder\`, VDI will be at `D:\MyFolder\KhanLubuntu24.04.vdi` |

### Virtual Machine Files
| Item | Location |
|------|----------|
| **VM Created At** | `C:\Users\YourUsername\VirtualBox VMs\KhanLubuntu\` |
| **Contains** | Cloned VDI + VM configuration files |

> 💡 **Tip:** The script clones the VDI to the VM folder, so you can delete the original VDI from the setup.bat folder after successful installation to save space.

---
## 🎯 Motivation behind this project?


Every class, our teacher had to carry a hard drive, plug it into each computer, and manually install Linux one by one. It took forever. Half the class time was wasted just on setup.

And the worst part? We couldn't practice at home. The installation process was way too complicated for most of us to figure out on our own.

So I built this.

Now, instead of waiting for the hard drive, you just download one file, double-click it, and go grab a cup of coffee or tea or whatever you prefer. By the time you're back, Linux is ready and running.

No hard drives. No complicated steps. No wasted time. Just download, click, and learn.

Whether you're in the classroom or at home, you'll have the exact same Linux environment ready in less than an hour.

---

## 🎓 Perfect For

- 🧑‍🎓 **Students** learning Linux for the first time
- 👨‍🏫 **Teachers** setting up computer labs
- 💻 **Beginners** who want to explore Linux safely
- 📚 **Classrooms** needing identical Linux environments
- 🏠 **Home practice** without complex setup

---

## ✨ What Does It Do?

This script **automatically**:

| Step | Action | Time |
|------|--------|------|
| 1 | Downloads Lubuntu virtual disk (8GB) | ~30 min (depends on internet speed) |
| 2 | Downloads & installs VirtualBox (if not present) | ~3-5 min |
| 3 | Creates virtual machine | ~1 min |
| 4 | Configures all settings | ~30 sec |
| 5 | Starts your Linux environment | Instant |

**Total time: ~35-40 minutes** (mostly just waiting for download)

Then you have a fully working Linux system!

> 💡 **Note:** The VDI download is 8GB. Download time depends on your internet speed. With a good connection it can be faster, with a slower connection it may take longer. Just let it run!

---

## 💡 Why Khan Lubuntu?

| Old Way (Hard Drive) | New Way (This Script) |
|----------------------|----------------------|
| Teacher brings hard drive | Just download and run |
| Hours of manual setup | ~35 minutes automatic |
| Only works in classroom | Works anywhere |
| Can't practice at home if you dont have harddrive | Practice anytime! |
| Different setups each time | Identical environment always |

---

## ⚡ User-Friendly Features

### 🚀 Fast Download
Uses **aria2c** download manager with **16 parallel connections**. This makes the download much faster than a normal browser download!


### 🤖 Fully Automated
- No user input required
- Silent VirtualBox installation
- Auto VM configuration
- Auto VM startup

### 😴 Sleep Prevention
Automatically disables system sleep during download. Restores settings when done. Your download won't fail because PC went to sleep!

### 🛡️ Error Recovery
If something goes wrong, the script:
- Cleans up incomplete files automatically
- Shows clear instructions on what to do
- Allows you to simply run again after fixing the issue

---

## 🛡️ Safe & Reliable

- ✅ Runs inside VirtualBox (won't affect your Windows)
- ✅ Resume support (download interruption? just run again)
- ✅ Tested and verified
- ✅ Open source (you can see exactly what it does)

---

## 📋 Requirements

| Requirement | Minimum |
|-------------|---------|
| Operating System | Windows 10 / 11 |
| RAM | 4GB (2GB for VM) |
| Free Storage | 15GB |
| Internet | Required for first setup |

---

## 🖥️ What You Get

A fully configured Lubuntu 24.04 LTS virtual machine with:

| Setting | Value |
|---------|-------|
| RAM | 2048 MB |
| CPU | 2 cores |
| Video Memory | 50 MB |
| Network | NAT (internet access) |
| Clipboard | Bidirectional (copy-paste works!) |
| Drag & Drop | Bidirectional |

---
## 🤝 Credits

- [aria2c](https://aria2.github.io) - Fast download manager with resume support
- [VirtualBox](https://www.virtualbox.org) - Virtualization software
- [Archive.org](https://archive.org) - Reliable file hosting
- [Lubuntu](https://lubuntu.me) - Lightweight Linux distribution
- [Claude AI](https://claude.ai) - AI assistant by Anthropic that helped build this automation script

---
## 📄 License

MIT License - Free to use, modify, and share.

---

## ⭐ Support

If this helped you, please give it a star! ⭐

Having issues? [Open an issue](https://github.com/YOUR_USERNAME/khan-lubuntu/issues)

---

Made with ❤️ for students who want to learn Linux without the hassle.
