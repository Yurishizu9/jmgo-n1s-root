# JMGO N1S 4K Root

Permanent root access for the JMGO N1S 4K laser projector.

**One script. No USB cable. No soldering. Takes 2 minutes.**

| | |
|---|---|
| **Device** | JMGO N1S 4K (Model S901/S913) |
| **OS** | Android 11 / BonfireOS |
| **Firmware** | 1.1.31.x |
| **Connection** | Wi-Fi (ADB over TCP) |

## What you get

- **Full root shell** — `su` from any ADB session
- **Install any app** — no JMGO restrictions
- **System partition writable** — modify anything
- **Survives reboots** — patches persist across power cycles and factory resets

## Requirements

- Python 3.6+
- ADB (`adb` in your PATH)
- Projector on the same Wi-Fi network as your computer

## Quick start

### 1. Install ADB on your computer

**macOS:**
```bash
brew install android-platform-tools
```
Don't have Homebrew? Install it from [brew.sh](https://brew.sh) first.

**Windows:** Download [SDK Platform-Tools](https://developer.android.com/tools/releases/platform-tools), unzip, click the address bar in the folder, type `cmd`, press Enter.

**Linux:**
```bash
sudo apt install adb   # Debian/Ubuntu
```

### 2. Find your projector's IP address

> **ADB is already on.** The JMGO N1S has ADB enabled by default — you do NOT need to turn on Developer Options or USB Debugging. There is nothing to enable.

1. Make sure the projector is on the **same Wi-Fi** as your computer
2. On the projector, open the **File Manager** app
3. Go to the **Local Network** section
4. The IP address is shown in the **top-right corner** (looks like `192.168.0.14`)
5. Write it down

### 3. Connect

```bash
adb connect YOUR_IP:5555
```

For example: `adb connect 192.168.0.14:5555`

You should see "connected" or "already connected".

### 4. Run the script

```bash
python3 jmgo_root.py YOUR_IP
```

On Windows use `python` instead of `python3`.

That's it. The script handles everything:

- Connects to your projector
- Installs a tiny helper app (embedded in the script)
- Backs up important data
- Applies three small patches to unlock root
- Reboots and verifies
- Sets up a convenient `su` command

### 5. Use root

```bash
# Root shell
adb shell su

# Run a command as root
adb shell su id
# → uid=0(root) gid=0(root) ...

# Install any app
adb install your-app.apk

# Make system writable
adb shell su blockdev --setrw /dev/block/dm-0
adb shell su mount -o rw,remount /
```

## How it works

The JMGO N1S has three exploitable weaknesses:

1. **Unprotected property trigger** — `init.jmgo.rc` contains a trigger that runs `chmod` and `chown` as root on any path, including raw block devices.

2. **Setuid-root `jsu` binary** — JMGO's custom `su` at `/system/xbin/jsu`, gated by two security functions in `libjmgosecury.so` and a boot flag check.

3. **No dm-verity** — The system partition has no integrity verification, so raw byte patches persist across reboots.

The script applies three patches (2–4 bytes each) to the raw eMMC super partition:

| # | Target | What | Bytes |
|---|--------|------|-------|
| 1 | `libjmgosecury.so` | `jmgo_check_security()` → return 0 | `00 20 70 47` |
| 2 | `libjmgosecury.so` | `jmgo_check_security_by_encrypt_code()` → return 0 | `00 20 70 47` |
| 3 | `jsu` | BNE (boot flag check) → NOP | `00 BF` |

Each patch is verified before and after writing. The script aborts if the bytes don't match, so it won't brick a device with different firmware.

## Reverting

Flash the official JMGO firmware via USB to fully restore the system partition.

## Troubleshooting

**"Can't connect"** — Make sure the projector is on the same Wi-Fi as your computer. Double-check the IP. Try `adb disconnect` then `adb connect YOUR_IP:5555` again.

**"adb: command not found"** — ADB isn't installed or isn't in your PATH. Go back to step 1.

**"unexpected bytes — wrong firmware?"** — Your firmware version has different binaries. The script only works on 1.1.31.x. Open an issue with your firmware version.

**"Projector didn't come back"** — Wait a full minute, then `adb connect YOUR_IP:5555` again.

**"su: not found"** — Use the full command: `adb shell /system/xbin/jsu --key x root`

## License

This is security research for use on your own hardware. Use at your own risk.

## Credits

Research, exploit development, and tooling by [Claude Code](https://claude.com/claude-code) (Anthropic).
