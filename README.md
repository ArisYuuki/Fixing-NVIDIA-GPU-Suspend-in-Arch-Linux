# Fixing-NVIDIA-GPU-Suspend-in-Arch-Linux
![Arch Linux](https://img.shields.io/badge/Arch-Linux-blue?logo=arch-linux)
![CachyOS](https://img.shields.io/badge/CachyOS-Optimized-brightgreen)
![NVIDIA](https://img.shields.io/badge/GPU-NVIDIA-76B900?logo=nvidia)

This is a fix for hybrid GPU laptops running NVIDIA GPU's. This will allow you to suspend your GPU fully when in hybrid graphics mode, allowing better battery life, and lower power consumption
# NVIDIA Runtime Power Management (RTX 3080 Mobile Example)

This guide enables **runtime power management (Runtime PM)** for a specific NVIDIA GPU using a udev rule.

⚠️ Make sure you use the correct PCI device ID for YOUR GPU.

---

## Step 1 — Create udev Rule File

```bash
sudo nano /etc/udev/rules.d/99-nvidia-runtimepm.rules
```

---

## Step 2 — Add This To The File

```bash
# Enable runtime PM for RTX 3080 Mobile
ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{device}=="0x24dc", ATTR{power/control}="auto"
```

---

## Step 3 — Find Your Correct NVIDIA PCI Device ID

Run:

```bash
lspci -nn | grep -i nvidia
```

### Example Output

```text
01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GA104M [GeForce RTX 3080 Mobile / Max-Q 8GB/16GB] [10de:24dc] (rev a1)

01:00.1 Audio device [0403]: NVIDIA Corporation GA104 High Definition Audio Controller [10de:228b] (rev a1)
```

### Understanding The IDs

- `10de` → NVIDIA’s vendor ID  
- `24dc` → Your GPU’s device ID  

So your rule should contain:

```bash
ATTR{vendor}=="0x10de"
ATTR{device}=="0x24dc"
```

Replace `24dc` with your actual GPU device ID if different.

---

## What This Does

When this exact NVIDIA GPU appears on the PCI bus, it enables:

```bash
ATTR{power/control}="auto"
```

This tells the system to use **runtime power management** for the device.

---

## Step 4 — Reload udev Rules

```bash
sudo udevadm control --reload
sudo udevadm trigger
```

Reboot recommended after applying changes.

# GPU Idle Monitor (Avoid Accidental Wakeups)

This script monitors NVIDIA runtime power management **without waking the GPU**. This is useful (yet completely optional) as using programs like BTOP will call on the GPU and wake it up while it is running. you can also check without this by simply typing "sensors" in your terminal while nothing is running (close steam or any other GPU processes first) and the GPU temperature should in theory read 0 or single digits when suspended.

It safely reads:
- `runtime_status`
- `runtime_active_time`
- `runtime_suspended_time`

---

## Create Idle Monitor Script

```bash
nano ~/gpu_idle_monitor.sh
```

---

## Add This Script

```bash
#!/bin/bash

GPU="/sys/bus/pci/devices/0000:01:00.0/power"

echo "Monitoring NVIDIA RTX 3080 Mobile runtime PM (safe, does NOT wake GPU)"
echo "Press Ctrl+C to stop"
echo

while true; do
    STATUS=$(cat $GPU/runtime_status)
    ACTIVE_US=$(cat $GPU/runtime_active_time)
    SUSPENDED_US=$(cat $GPU/runtime_suspended_time)

    ACTIVE_SEC=$(echo "scale=2; $ACTIVE_US / 1000000" | bc)
    SUSPENDED_SEC=$(echo "scale=2; $SUSPENDED_US / 1000000" | bc)

    printf "Status: %-10s | Active: %6.2fs | Suspended: %6.2fs\r" \
        "$STATUS" "$ACTIVE_SEC" "$SUSPENDED_SEC"

    sleep 1
done
```

---

## Save and Exit

- CTRL+O → Enter  
- CTRL+X  

---

## Make Script Executable

```bash
chmod +x ~/gpu_idle_monitor.sh
```

---

## Run Monitor

```bash
~/gpu_idle_monitor.sh
```

Press **Ctrl+C** to stop monitoring.

---

## ⚠️ Important

If your GPU PCI address is different, check with:

```bash
lspci | grep -i nvidia
```

Replace:

```bash
0000:01:00.0
```

with your actual PCI device path if needed.
