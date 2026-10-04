# OnePlus 15 / 15R: EFI files and root guide

This repo has the EFI (ABL) bootloader files for the OnePlus 15 and OnePlus 15R, and a step-by-step guide to rooting both phones with Magisk.

The basic method is the normal modern Android root route:

1. Unlock the bootloader.
2. Install Magisk.
3. Patch the stock `init_boot` image.
4. Flash the patched `init_boot`.
5. Flash the matching EFI file from this repo.
6. Format data from recovery.
7. Boot and verify root.

**Warning:** unlocking wipes your data, and flashing the wrong file or partition can brick the phone. Use the correct files for your exact model, keep a backup, and take your time.

---

## What you need

- OnePlus 15 or OnePlus 15R
- A PC running Windows, macOS, or Linux
- A good USB cable
- Android SDK Platform Tools installed on the PC
- The official firmware or ROM package for your exact device and firmware version
- Magisk APK installed on the phone
- A backup of anything important

---

## 1. Install adb and fastboot on your PC

Download Android SDK Platform Tools from the official Android developer site.

Make sure `adb` and `fastboot` work from your terminal or command prompt.

Check with:

```bash
adb version
fastboot version
```

If the commands are not found, add Platform Tools to your system PATH.

---

## 2. Prepare the phone

On the phone:

1. Open Settings.
2. Go to About Phone.
3. Tap Build Number several times until Developer Options are enabled.
4. Open Developer Options.
5. Enable USB debugging.
6. Enable OEM unlocking.

Connect the phone to the PC.

On the PC, run:

```bash
adb devices
```

If this is your first time connecting, accept the USB debugging prompt on the phone.

If the device shows as unauthorized, reconnect and accept the prompt again.

---

## 3. Unlock the bootloader

Unlocking the bootloader will factory reset the phone.

Reboot into Fastboot:

```bash
adb reboot bootloader
```

Confirm the device is detected:

```bash
fastboot devices
```

Unlock the bootloader:

```bash
fastboot flashing unlock
```

On some older devices, the command may be:

```bash
fastboot oem unlock
```

Follow the confirmation prompt on the phone.

After unlocking, the phone will wipe data and reboot.

Once Android starts:

1. Complete the initial setup.
2. Re-enable Developer Options.
3. Re-enable USB debugging.
4. Make sure OEM unlocking is still enabled.
5. Reconnect the phone to the PC.
6. Accept the USB debugging prompt again.

Check ADB again:

```bash
adb devices
```

---

## 4. Install Magisk on the phone

Download Magisk from the official Magisk GitHub releases page:

```text
https://github.com/topjohnwu/Magisk/releases
```

Download the APK to the phone.

Install the APK.

If Android blocks the install, allow your browser or file manager to install unknown apps.

Open Magisk once installed.

You do not need to do anything inside Magisk yet except make sure it opens normally.

---

## 5. Get the stock init_boot image

You need the stock `init_boot.img` from the exact firmware currently installed on the phone.

Do not use a random `init_boot.img` from a different firmware version.

If you already have the correct stock `init_boot.img`, skip to the next section.

If you have an official firmware package, it may contain a `payload.bin` file.

To extract only `init_boot.img`, use payload-dumper-go.

Download payload-dumper-go from its official GitHub releases page.

Extract it somewhere easy to access.

Place the `payload.bin` file in the same folder or note its full path.

On macOS/Linux:

```bash
./payload-dumper-go -partitions init_boot payload.bin
```

On Windows:

```bash
payload-dumper-go.exe -partitions init_boot payload.bin
```

The extracted image will usually be inside an output folder created by payload-dumper-go.

Look for:

```text
init_boot.img
```

If your firmware package already contains `init_boot.img` directly, you can use that instead.

---

## 6. Copy init_boot.img to the phone

Copy the stock `init_boot.img` to the phone so Magisk can patch it.

Example:

```bash
adb push init_boot.img /sdcard/Download/
```

Confirm the file is on the phone.

You should be able to see it in:

```text
Download/init_boot.img
```

---

## 7. Patch init_boot with Magisk

On the phone:

1. Open Magisk.
2. Tap Install.
3. Choose Select and Patch a File.
4. Select the stock `init_boot.img` you copied to the phone.
5. Let Magisk patch it.

When finished, Magisk will create a patched image, usually in:

```text
/sdcard/Download/
```

The filename will look something like:

```text
magisk_patched-xxxxx.img
```

Write down the exact filename.

---

## 8. Pull the patched image to your PC

From the PC, pull the patched image back to your computer.

Example:

```bash
adb pull /sdcard/Download/magisk_patched-xxxxx.img .
```

Replace `magisk_patched-xxxxx.img` with the actual filename Magisk created.

Optionally rename it to something simpler:

```bash
mv magisk_patched-xxxxx.img magisk_patched_init_boot.img
```

On Windows Command Prompt:

```bat
ren magisk_patched-xxxxx.img magisk_patched_init_boot.img
```

On Windows PowerShell:

```powershell
Rename-Item magisk_patched-xxxxx.img magisk_patched_init_boot.img
```

---

## 9. Flash the patched init_boot

Reboot into Fastboot:

```bash
adb reboot bootloader
```

Confirm the device is detected:

```bash
fastboot devices
```

Flash the patched `init_boot`:

```bash
fastboot flash init_boot magisk_patched_init_boot.img
```

If you kept the original Magisk filename, use that instead:

```bash
fastboot flash init_boot magisk_patched-xxxxx.img
```

Only flash the patched `init_boot` here.

Do not flash other partitions unless you know exactly what you are doing.

---

## 10. Flash the matching EFI file from this repo

Use the EFI file that matches your exact phone model.

First, check that the file downloaded correctly. Its SHA-256 hash must match the one in `source/SHA256SUMS.txt`.

On Windows PowerShell:

```powershell
Get-FileHash -Algorithm SHA256 -LiteralPath "source/oneplus15/ABL_with_superfastboot(oneplus15).efi"
```

On macOS/Linux:

```bash
sha256sum -c source/SHA256SUMS.txt
```

On macOS, use `shasum -a 256 -c source/SHA256SUMS.txt` if `sha256sum` is not installed.

If the hash does not match, download the file again. Do not flash it.
For OnePlus 15:

```bash
fastboot flash efisp "source/oneplus15/ABL_with_superfastboot(oneplus15).efi"
```

For OnePlus 15R:

```bash
fastboot flash efisp "source/oneplus15r/(oneplus 15R)ABL_with_superfastboot.efi"
```

The filenames contain spaces and parentheses, so keep the quotes around the path.

If you renamed the files, adjust the command to match the new filename.

Example:

```bash
fastboot flash efisp source/oneplus15/oneplus15_superfastboot.efi
```

---

## 11. Do not reboot into the system yet

After flashing the patched `init_boot` and the EFI file:

Do not reboot straight into Android.

Instead, boot into Recovery:

```bash
fastboot reboot recovery
```

If that command does not work, use the phone hardware keys to enter Recovery manually.

---

## 12. Format data from Recovery

In Recovery:

1. Choose factory reset, format data, or wipe data.
2. Confirm.
3. Wait for the process to finish.

This step is important.

Skipping it can cause boot issues after changing boot components.

After formatting, reboot the system from Recovery.

---

## 13. Verify root

Once Android boots:

1. Open Magisk.
2. It should show that Magisk is installed.
3. If Magisk asks for additional setup, allow it and reboot if requested.

To check root from the PC:

```bash
adb shell
su
id
```

If root is working, `id` should show something like:

```text
uid=0(root)
```

You can also install a simple root checker app from the Play Store if you prefer.

---

## 14. Optional: install modules

After root is working, you can install Magisk modules.

Common examples:

- Zygisk-related modules
- LSPosed
- systemless hosts
- font modules
- audio modules

General advice:

- Prefer modules that work systemless/overlay style.
- Avoid modules that directly modify partitions.
- If a module causes a bootloop, remove it from Magisk or restore the stock image.

If you use apps that are sensitive to root, configure Magisk hiding or DenyList as needed.

---

## 15. Optional: relock the bootloader

For most rooted devices, leaving the bootloader unlocked is normal.

Relocking is optional and can be risky if the device is modified.

Only relock if:

- The phone boots normally.
- Root works.
- You are comfortable with the device state.
- You have stock firmware and recovery tools available.

If you still want to relock:

```bash
adb reboot bootloader
fastboot devices
fastboot flashing lock
```

On some older devices:

```bash
fastboot oem lock
```

Then reboot:

```bash
fastboot reboot
```

If you are unsure, do not relock.

---

## Notes

### Keep OEM unlocking enabled

Do not disable OEM unlocking in Developer Options.

If you disable it, you may lose access to Fastboot later.

Developer Options can be hidden or disabled if you want, but OEM unlocking should stay enabled.

---

### Use the correct files

Always use:

- The correct firmware for your exact device.
- The correct stock `init_boot.img`.
- The correct Magisk-patched image.
- The correct EFI file for your model.

Do not mix OnePlus 15 and OnePlus 15R files.

---

### Do not flash random partitions

For this guide, you only need to flash:

- `init_boot`
- `efisp`

Avoid flashing things like:

- `dtbo`
- recovery
- modem
- persist
- modified ROM partitions
- custom recovery

unless you fully understand what you are doing.

---

### Verified boot

Do not try to disable verified boot features.

This guide works with the normal unlock and patch flow.

---

### TEE, banking apps, and attestation

After unlocking and rooting, some things may change:

- TEE / key attestation state
- Play Integrity
- SafetyNet-related results
- banking apps
- DRM behavior

This is normal on modified devices.

You may need to configure hiding, DenyList, or module settings depending on your apps.

---

### Recovery partition

Do not modify the recovery partition.

Modifying recovery can cause bootloops, random reboots, or failure to boot Android.

---

### Root managers

Any root manager that works by patching `boot` or `init_boot` can generally be used, as long as:

- The bootloader is unlocked.
- The patched image matches the exact device and firmware.
- The correct partition is flashed.

Magisk is used in this guide because it is the most common option.

---

## Files in this repo

| Device | EFI file | Location |
|---|---|---|
| OnePlus 15 | `ABL_with_superfastboot(oneplus15).efi` | `source/oneplus15/` |
| OnePlus 15R | `(oneplus 15R)ABL_with_superfastboot.efi` | `source/oneplus15r/` |

Use the file that matches your device.

The filename may differ if you renamed it. Adjust the fastboot command accordingly.

---

## What the EFI files are

These EFI files are ARM64 UEFI/ABL bootloader-stage files.

They are not root tools.

They are part of the boot chain and handle things like:

- Fastboot
- bootloader unlock confirmation
- A/B slot selection
- verified boot behavior
- boot screens and OEM boot commands

Root comes from the patched `init_boot` image, not from the EFI file.

The name `superfastboot` is a nickname. It does not appear as a literal string inside the binaries.

The OnePlus 15 and OnePlus 15R files are model-specific and are not the same file renamed.

---

## Basic troubleshooting

### adb does not see the phone

Check:

- USB debugging is enabled
- the phone screen is unlocked
- you accepted the USB debugging prompt
- the USB cable is working
- the correct drivers are installed on Windows

Restart adb:

```bash
adb kill-server
adb start-server
adb devices
```

---

### fastboot does not see the phone

Check:

- the phone is actually in Fastboot mode
- the USB cable is working
- the correct drivers are installed
- try another USB port

Confirm with:

```bash
fastboot devices
```

---

### Magisk patch fails

Make sure:

- you are using the correct stock `init_boot.img`
- the file is not corrupted
- the phone has enough storage
- Magisk is updated

If the stock image is wrong, extract it again from the correct firmware.

---

### Phone bootloops after flashing

Try:

- boot into Recovery
- format data again
- reflash the correct patched `init_boot`
- reflash the stock `init_boot` if you want to go back
- restore stock firmware if needed

---

### Root does not work

Check:

- you flashed the patched image to `init_boot`
- the patched image came from the correct stock `init_boot`
- the firmware version matches
- Magisk app is installed
- Magisk was given a chance to complete additional setup

You can also re-patch the stock image and flash it again.

---

### Modules cause problems

If a module causes issues:

- remove it from Magisk
- disable it
- restore stock `init_boot`
- reinstall Magisk cleanly if needed

Prefer systemless/overlay modules.

---

## Short command summary

```bash
adb devices
adb reboot bootloader
fastboot devices
fastboot flashing unlock

fastboot flash init_boot magisk_patched_init_boot.img

fastboot flash efisp "source/oneplus15/ABL_with_superfastboot(oneplus15).efi"
# or
fastboot flash efisp "source/oneplus15r/(oneplus 15R)ABL_with_superfastboot.efi"

fastboot reboot recovery

adb reboot bootloader
fastboot flashing lock
fastboot reboot
```
