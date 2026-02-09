# Troubleshooting

This document covers common issues and edge cases. If you encounter a problem not listed here, please [open an issue](../../issues).

---

## Step 1: Firmware Download

### Problem: Download keeps failing or corrupting

**Solution:** 
- Use a wired connection if possible
- Use a download manager that supports resume (like `wget -c`)
- Verify the checksum after download if ipsw.me provides one

### Problem: Not sure which firmware to download

**Solution:**
- Use the device picker on ipsw.me
- Select your exact Mac model
- Choose the **UniversalMac** firmware
- Download the **top entry** (latest signed version)

---

## Step 2: Building idevicerestore

### Problem: Dependency errors during compilation

**Solution:**
Build dependencies in this order:
1. libplist
2. libimobiledevice-glue
3. libusbmuxd
4. libimobiledevice
5. libirecovery
6. idevicerestore

Each one must complete successfully before moving to the next.

### Problem: usbmuxd won't start

**Solution:**
```bash
sudo systemctl stop usbmuxd
sudo usbmuxd -f -v
```

This runs it in foreground with verbose output. Check for errors.

### Problem: Permission denied errors

**Solution:**
Add udev rules for Apple devices:

```bash
sudo nano /etc/udev/rules.d/39-libirecovery.rules
Add:

text
SUBSYSTEM=="usb", ATTR{idVendor}=="05ac", MODE="0666"
Then:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

---

## Step 3: DFU Mode

### Problem: Can't get into DFU mode, keeps booting normally

**Solution:**

- Make sure you're using the left Control, left Option, and right Shift
- Count the full 10 seconds - most people release too early
- Try the powered-on method (Method 1) if powered-off method fails
- Ensure the USB-C cable is a data cable, not charge-only

### Problem: dmesg shows "Recovery Mode" not "DFU Mode"

**Solution:**

- Recovery Mode is not DFU Mode. Start over. The timing is:

    - 10 seconds all keys
    - Release all except power
    - 7-10 more seconds holding only power

### Problem: dmesg shows nothing at all
**Solution:**

- Try a different USB-C port on your Linux machine
- Try a different USB-C cable (must be data-capable)
- Check that usbmuxd is running
- Try lsusb to see if any Apple device appears

## Step 4: Firmware Restore

### Problem: idevicerestore fails with signing error

**Solution:**

- Ensure you downloaded the latest signed firmware
- Your internet connection must be stable during restore (Apple servers verify signing)
- Try again - sometimes Apple's servers are slow

### Problem: idevicerestore fails with "unable to send" or USB errors

**Solution:**

- Don't touch either machine during restore
- Use systemd-inhibit to prevent sleep (included in the command)
- Try a different USB-C port
- Try a shorter/higher-quality USB-C cable

### Problem: Restore completes but Mac won't boot
**Solution:**

Use the nuclear option:

- Boot to Recovery Mode
- Disk Utility > Erase internal SSD (APFS, GUID)
- Retry from Step 3

## Step 5: Setup Assistant Bypass

### Problem: dscl commands return errors
**Solution:**

- Check your volume name exactly: ls /Volumes/
- Volume name is case-sensitive and space-sensitive
- If volume is Macintosh HD, that's different from macintosh hd
- Retype commands carefully - no extra spaces

### Problem: After reboot, Setup Assistant still appears
**Solution:**

- The .AppleSetupDone file wasn't created correctly. Boot to Recovery, open Terminal, and verify:

```bash
ls -la "/Volumes/Macintosh HD/var/db/.AppleSetupDone"
```
- If it doesn't exist, create it again.

### Problem: Can't log in as admin user
**Solution:**

- Password may not have been set correctly. Boot to Recovery, open Terminal:

```bash
dscl -f "/Volumes/Macintosh HD/var/db/dslocal/nodes/Default" localonly -passwd /Local/Default/Users/admin "newpassword"
```

## Step 6: Ownership and Secure Tokens

### Problem: Can't create second user in System Settings
**Solution:**

- Make sure you're logged in as the dscl-created admin
- System Settings > Users & Groups > Click the lock to make changes
- Enter the admin password

### Problem: updatePreboot fails
**Solution:**

```bash
sudo diskutil apfs updatePreboot / -verbose
```
Check the verbose output for specific errors.

### Problem: Secure token not granted
**Solution:**

- Verify token status:

```bash
sysadminctl -secureTokenStatus yourusername
```
If not enabled, the GUI user creation may not have worked correctly. Try creating another admin user through System Settings.

## Step 7: Asahi Installation

### Problem: MDM prompt appears but there's no "Not Now" button
**Solution:**

This means you don't have proper secure tokens. Go back to Step 6:

- Verify updatePreboot completed successfully
- Make sure you're logged into the GUI-created admin account
- Run updatePreboot again if needed

### Problem: Asahi installer fails to resize partition
**Solution:**

- Make sure you have enough free space on the macOS partition
- Try running Disk Utility First Aid on the volume before installing Asahi

## Problem: After reboot, can't find Linux in boot picker
**Solution:**
Hold the power button until you see startup options. Linux should appear as a boot option. If not, the Asahi installation may not have completed - retry the curl command.

## General

### Problem: Something else went wrong
**Solution:**

- Check Asahi Linux documentation
- Check libimobiledevice issues
- Open an issue in this repository with:
    - Your Mac model
    - Which step failed
    - Exact error message
What you tried