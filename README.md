# mdm-bypass-guide
A guide for bypassing MDM locked Apple Silicon laptops. 
# Liberating an MDM-Locked MacBook

A step-by-step guide to bypassing enterprise MDM enrollment on Apple Silicon Macs and installing Linux.

**This is documentation, not a tool.** No exploits. No jailbreaks. No sketchy software. Just Apple's own tools, used correctly.

---

## The Story

Three years ago, a corporation held my final paycheck hostage unless I returned a MacBook using my own money. When I refused, they MDM-locked it with Mosyle - enterprise-grade device management that integrates with Apple Business Manager. The kind Fortune 500 companies pay premium for because it's "unbreakable."

Every forum said the same thing: *"You have a \$2000 paperweight."*

Three years later, I'm typing this on that MacBook. It runs Fedora. It's mine.

Here's how.

---

## The Liberation

To everyone staring at a black screen demanding a code they don't have. To everyone looking at a message that says "This MacBook is managed by [Corporation That Doesn't Own Your Soul]."

There is a will. There is a way.

If you can follow instructions, I will walk you through every step of bypassing that MDM lock, establishing ownership, and liberating that hardware from the clutches of corporate control.

**One caveat:** Your Mac will never run macOS again. As of this writing, you'll also lose Neural Engine functionality until the teams at Asahi Linux and Fedora figure out how to reverse-engineer it.

---

## What You'll Need

- The MDM-locked Mac
- A Linux computer
- A USB-C data cable (not a charge cable)
- ~40GB free space on an internal drive
- Patience

## What You'll Lose

- macOS (forever)
- Neural Engine functionality (for now)

## What You'll Gain

- A fully functional laptop running Linux
- Your dignity

---

## Why This Works

Apple's security isn't one lock - it's a series of doors. Each door checks that you came through the previous one legitimately.

This guide walks you through every door in the right order until the system recognizes you as the owner. No jailbreak. No exploits. No sketchy software. Just Apple's own tools, used correctly.

---

## Why This Is Free

I could script this. Package it. Sell it. There are millions of MDM-locked Macs sitting in drawers and closets and e-waste bins. People would pay for this.

But the moment I sell it, it becomes a "bypass tool" - and that puts a target on my back I neither want nor need. This is documentation. A tutorial. Nothing more.

**A note about Mosyle:** This wasn't some bargain-basement MDM. Mosyle markets themselves as "the only Apple Unified Platform" - enterprise-grade device management that integrates directly with Apple Business Manager. They compete with Jamf for Fortune 500 contracts. They're what companies deploy when they want serious endpoint security. The enrollment is tied to the hardware serial number at Apple's activation servers.

It's supposed to be unbreakable.

It's not.

---

## Step 1: Download the Firmware

Before you touch that locked Mac, you need to download the restore files onto your Linux machine.

Go to [ipsw.me](https://ipsw.me) and use their device picker to find your specific Mac model. You're looking for the **UniversalMac** firmware files, and you want the **latest signed version** - that's the one at the top of the list.

**Why "signed" matters:** Apple's restore process includes checksum verification. If you try to restore with an unsigned or corrupted firmware file, it will fail. Download the latest signed version and save yourself the headache.

**Why download first:** You're going to be running a restore process that needs stable, verified files. Downloading mid-restore over sketchy internet is asking for corruption. Get the file first. Put it somewhere you can find it - your Downloads folder, a dedicated directory on your Desktop, wherever. Just know where it is.

This file is large. Mine took two days to download on rural internet. Plan accordingly.

**Important:** When idevicerestore runs, it will unzip this firmware file into a lock file in the same directory. You need approximately **40GB of free disk space** to accommodate this, and the file **must be on an internal drive**. A USB drive will not work. Plan your download location accordingly.

---

## Step 2: Build idevicerestore from Source

This is the tool that's going to perform the actual firmware restore. You need to compile it from source using the GitHub repository - **do not use the packaged versions from AUR, DNF, Pacman, or APT.** The repo version is more current and more reliable for this specific use case.

**The repository:** [https://github.com/libimobiledevice/idevicerestore](https://github.com/libimobiledevice/idevicerestore)

Follow the README instructions exactly. You'll need to build the dependencies first:

- libimobiledevice
- libirecovery
- libplist
- libusbmuxd
- libimobiledevice-glue

Yes, it's a lot. Yes, it's tedious. No, you cannot skip this part.

**Critical:** Make sure your `usbmuxd` service is installed, updated, and running. This is what allows your Linux machine to communicate with the Mac in DFU mode. If usbmuxd isn't working, nothing else will either.

To check:

```bash
systemctl status usbmuxd
```
If it's not running:
```
bash
sudo systemctl start usbmuxd
sudo systemctl enable usbmuxd
```

Once everything is compiled and usbmuxd is confirmed working, you're ready for the fun part.

## Step 3: Enter DFU Mode (The Finger Dance)

DFU (Device Firmware Update) mode is a low-level recovery state that bypasses the normal boot process. Apple doesn't want you here. That's why it's so hard to figure out how to unlock the door...turns out, there are two keys for this one. 

Connect your machines:
Plug your USB-C data cable (not a charge cable, this will not work with a charge cable) into your Linux machine and your Mac. The Mac can be on or off for this part - we'll get to that.

Open a terminal on your Linux machine and run:

```
bash
sudo dmesg -w
```

This watches for USB device connections in real-time. You should see something indicating the Mac is connected. You might see a "bind failed" message - that's fine. What matters is what comes next.

## The Finger Dance

There are two methods. Try Method 2 first (it's simpler), and if it doesn't work after 2-3 attempts, use Method 1.

### Method 2: Powered Down (Try This First)
Make sure your Mac is completely powered off
Press and hold simultaneously:
Power button
Left Control
Left Option (Alt)
Right Shift
Hold all four for 10 seconds
Release Control, Option, and Shift, but keep holding Power for another 7-10 seconds
Watch your dmesg output

### Method 1: Live Screen
If Method 2 fails repeatedly, try this with the Mac powered on (any screen showing):

Tap the power button once, then immediately tap and hold it again
On that second tap, simultaneously press:
Left Control
Left Option (Alt)
Right Shift
Hold all four for 10 seconds
Release Control, Option, and Shift, but keep holding Power for another 7-10 seconds
Watch your dmesg output

## What you're looking for:
Your dmesg output should show your Mac model name followed by (DFU Mode) in parentheses.

If you see anything else - Recovery Mode, normal boot, nothing at all - start over. It must say DFU Mode. This is non-negotiable.

The screen on the Mac will be black. That's correct. DFU mode has no visual indicator on the device itself - you confirm it through dmesg on your Linux machine.

Don't panic if it takes a few tries. I had to restore this thing three times before I got through the whole process successfully. The finger dance is finicky. The timing matters. Keep your dmesg window visible and keep trying until you see those magic words.

## Step 4: Restore the Firmware
Leave your dmesg terminal open and open a new terminal window or tab.

Navigate to the directory where you downloaded your firmware file and run:

```bash
TMPDIR=$PWD systemd-inhibit sudo idevicerestore -e UniversalMac_26.2_25C56_Restore.ipsw
```

Replace the filename with your actual firmware file if different.

## What this command does:

TMPDIR=$PWD - Sets the temporary directory to your current working directory, ensuring the unzipped lock file ends up in the right place
systemd-inhibit - Prevents your Linux system from going to sleep or hibernating during the restore (which would be catastrophic)
sudo idevicerestore -e - Runs the restore with elevated privileges; the -e flag enables erase/restore mode

## What happens next:
idevicerestore will verify the firmware, communicate with Apple's servers for signing verification, and then wipe and restore your Mac. This takes a while. Let it run. Don't touch anything. Don't breathe on it. Go make a sandwich.

## The nuclear option:
If you have an Apple ID or Find My Device previously configured on this Mac, or if the restore completes but you still hit issues in the next steps, you may need to go scorched earth:

## Complete the restore
Boot into Recovery Mode (power button hold until "Loading startup options")
Open Disk Utility
Select your internal SSD
Erase it completely - APFS format, GUID partition scheme
Close Disk Utility
Go back to Step 3 and restore the firmware again onto the blank drive
This ensures every trace of previous users, profiles, and configurations is gone. Clean slate.

## Step 5: Bypass Setup Assistant
After your firmware restore completes successfully, your Mac will reboot to the Setup Assistant. This is where Apple expects you to connect to WiFi, sign in with an Apple ID, and - if your device is MDM enrolled - get slapped right back into corporate jail.

We're not doing that.

Instead, you're going to reboot into Recovery Mode and use the terminal to convince macOS that setup already happened. Here's how:

### 5a. Boot into Recovery Mode
Power off the Mac completely. Then press and hold the power button until you see "Loading startup options..." appear. Select "Options" and click Continue. This drops you into Recovery Mode.

### 5b. Open Terminal
From the menu bar at the top, click Utilities > Terminal. This is the Recovery terminal - it has root access and doesn't give a fuck about your MDM enrollment.

### 5c. Find your volume name
```bash
ls /Volumes/
```
You're looking for your main macOS volume. It's probably called Macintosh HD but might be different. Note the exact name, spaces and all.

### 5d. Create the setup completion breadcrumb
This tells macOS that Setup Assistant already did its job:

```bash
touch "/Volumes/Macintosh HD/var/db/.AppleSetupDone"
```
(Replace Macintosh HD with your actual volume name if different.)

### 5e. Create your admin user
Now we create a local admin account that bypasses the entire setup flow.

Important: Each of these commands should return silently - no output, just a new command prompt. Silence means success. If you see any error message or response from the system, the command either wasn't typed correctly or something else has gone wrong. You'll need to go back to the nuclear option: wipe the SSD via Disk Utility in Recovery Mode, restore the firmware again from Step 3, and retry these steps. Don't proceed if you see errors here - they will cascade into bigger problems later.

Run each of these commands one at a time, pressing Enter after each:

```bash
dscl -f "/Volumes/Macintosh HD/var/db/dslocal/nodes/Default" localonly -create /Local/Default/Users/admin
```
```bash
dscl -f "/Volumes/Macintosh HD/var/db/dslocal/nodes/Default" localonly -create /Local/Default/Users/admin UserShell /bin/zsh
```
```bash
dscl -f "/Volumes/Macintosh HD/var/db/dslocal/nodes/Default" localonly -create /Local/Default/Users/admin RealName "Admin"
```
```bash
dscl -f "/Volumes/Macintosh HD/var/db/dslocal/nodes/Default" localonly -create /Local/Default/Users/admin UniqueID 501
```
```bash
dscl -f "/Volumes/Macintosh HD/var/db/dslocal/nodes/Default" localonly -create /Local/Default/Users/admin PrimaryGroupID 20
```
```bash
dscl -f "/Volumes/Macintosh HD/var/db/dslocal/nodes/Default" localonly -create /Local/Default/Users/admin NFSHomeDirectory /Users/admin
```
```bash
dscl -f "/Volumes/Macintosh HD/var/db/dslocal/nodes/Default" localonly -passwd /Local/Default/Users/admin "yourpasswordhere"
```
(Replace yourpasswordhere with an actual password you'll remember. You'll need it in about 30 seconds.)

### 5f. Add your user to the admin group
```bash
dscl -f "/Volumes/Macintosh HD/var/db/dslocal/nodes/Default" localonly -append /Local/Default/Groups/admin GroupMembership admin
```
### 5g. Create the home directory
```bash
mkdir "/Volumes/Macintosh HD/Users/admin"
```
### 5h. Reboot
```bash
sudo reboot now
```

## What happens next:
Your Mac will boot to a login screen. Enter admin and the password you set.

You're in.

Note: On first login, you'll be walked through a "Setup Assistant Lite" - a stripped-down version of the usual setup process. Skip everything you can skip. Click through it. Don't connect to WiFi yet, don't sign into an Apple ID, just get through it. This should not trigger an MDM enrollment screen - that comes later.

## Step 6: Establish Ownership
You're logged in, but we're not done. Your dscl-created admin account is a ghost as far as the Secure Enclave is concerned. macOS doesn't fully trust it. You need to create a "real" user through the GUI and get proper secure tokens assigned.

### 6a. Create a second admin user via System Settings
Open System Settings (or System Preferences on older macOS)
Go to Users & Groups
Click the + to add a new user
Create a new Administrator account with whatever username you actually want to use
Set a password
Log out of the admin account and log into your new account

### 6b. Update the Preboot volume
This step is critical. Do not skip it. Do not forget it. This is the step that makes everything else possible.

Open Terminal in your new account and run:

```bash
sudo diskutil apfs updatePreboot /
```
Then:
```bash
sudo reboot now
```

### Why this matters:
The Secure Enclave is Apple's hardware security module. It controls encryption keys, biometric data, and - crucially for us - determines who is a "legitimate" owner of the device. Without a secure token, you can't do things like enable FileVault, approve kernel extensions, or - most importantly for our purposes - tell MDM enrollment to fuck off and then install an alternative OS.

This command updates the Preboot volume with the current user credentials, granting secure tokens to both admin accounts - the dscl-created ghost account and your new GUI-created account. You are now recognized as a real owner. The Mac is yours.

From this point forward, use your new GUI-created admin account for everything - including the Asahi installation in Step 7.

## Step 7: Install Asahi Linux
This is it. The moment of triumph. The middle finger raised high.

### 7a. Install Asahi Linux
Log into your GUI-created admin account, connect to the internet, open Terminal and run:

```bash
curl https://alx.sh | sh
```
Follow the on-screen prompts. 

Asahi will:

- Resize your macOS partition (I recommend min to keep the macOS partition as small as possible)
- Create a new partition for Linux (I recommend Max to give the new OS all available space)
- Download and install Asahi Linux (I recommend Fedora Asahi Remix with KDE Plasma)
- Configure the bootloader
- During the installation process the MDM enrollment prompt may appear. If it does, there should be an option to click "Not Now". Enrollment is no longer a requirement.

The installation process is remarkably straightforward. The Asahi team has done incredible work making this accessible.

### 7b. Reboot into Linux
When the installation completes:

```bash
sudo reboot now
```
You'll see a boot picker. Select your new Linux installation.

**Welcome to your liberated Mac.**

## The Aftermath
You now have a fully functional MacBook running Linux. The MDM enrollment is still technically "there" - if you ever booted back into macOS and connected to the internet, it would try again. But you're not going to do that. That tiny macOS partition exists only as a vestigial organ, a reminder of what you overcame.

### What works:
- Everything that Asahi supports: display, keyboard, trackpad, WiFi, Bluetooth, audio, webcam, USB-C, battery management
- It's fast. Smooth. Responsive. Zero latency.
- It's yours.

### What doesn't work (yet):
- Neural Engine - Apple's ML acceleration hardware remains locked behind proprietary drivers
- Some GPU features are still being reverse-engineered

The Asahi and Fedora teams are actively working on these. Check asahilinux.org for updates.

## Resources
- ipsw.me - Firmware downloads and device identification
- libimobiledevice project - idevicerestore and dependencies
- Asahi Linux - The incredible team reverse-engineering Apple Silicon
- Fedora Asahi Remix - The Linux distribution I recommend

## Contributing
Tried this guide? Please report your results in TESTED-DEVICES.md or open an issue.

Found a problem or edge case? Check TROUBLESHOOTING.md or submit an issue.

## License
This documentation is released under CC BY-SA 4.0. Share it. Adapt it. Just give credit and keep it open.

Carissa Bays | Hawkington Technologies | February 2026
