# Hyprland Automatic Wallpaper Rotation

A simple setup for automatically changing wallpapers every **10 minutes** on Hyprland using `awww`.

## 1. Install `awww`

On Arch Linux:

```bash
sudo pacman -S awww
```

Verify the installation:

```bash
which awww
awww --version
```

Check the daemon:

```bash
which awww-daemon
```

---

## 2. Create the Wallpaper Directory

Create a dedicated directory for your wallpapers:

```bash
mkdir -p ~/Pictures/Wallpapers
```

Place your wallpapers inside:

```text
~/Pictures/Wallpapers/
├── wallpaper1.jpg
├── wallpaper2.png
├── wallpaper3.webp
└── wallpaper4.jpg
```

> **Important:** Linux is case-sensitive. `Wallpapers` and `wallpapers` are different directories.

---

## 3. Create the Wallpaper Rotation Script

Create the script:

```bash
mkdir -p ~/.local/bin
nano ~/.local/bin/wallpaper-rotate.sh
```

Add:

```bash
#!/bin/bash

WALLPAPER_DIR="$HOME/Pictures/Wallpapers"
INTERVAL=600

# Check wallpaper directory
if [ ! -d "$WALLPAPER_DIR" ]; then
    echo "ERROR: Wallpaper directory does not exist:"
    echo "$WALLPAPER_DIR"
    exit 1
fi

# Start awww daemon if it isn't already running
if ! pgrep -x awww-daemon >/dev/null; then
    echo "Starting awww daemon..."
    awww-daemon &
    sleep 2
fi

while true; do

    # Select a random wallpaper
    WALLPAPER=$(find "$WALLPAPER_DIR" -type f \
        \( -iname "*.jpg" \
        -o -iname "*.jpeg" \
        -o -iname "*.png" \
        -o -iname "*.webp" \) \
        -print0 | shuf -z -n 1 | tr -d '\0')

    if [ -z "$WALLPAPER" ]; then
        echo "ERROR: No wallpapers found."
        sleep "$INTERVAL"
        continue
    fi

    echo "Changing wallpaper to:"
    echo "$WALLPAPER"

    if [ -f "$WALLPAPER" ]; then
        awww img "$WALLPAPER" \
            --transition-type grow \
            --transition-duration 1.5 \
            --transition-fps 60
    fi

    sleep "$INTERVAL"
done
```

---

## 4. Make the Script Executable

Run:

```bash
chmod +x ~/.local/bin/wallpaper-rotate.sh
```

You can verify:

```bash
ls -l ~/.local/bin/wallpaper-rotate.sh
```

It should contain an `x` permission, for example:

```text
-rwxr-xr-x
```

---

## 5. Test the Script

Before using a 10-minute interval, temporarily change:

```bash
INTERVAL=600
```

to:

```bash
INTERVAL=10
```

This makes the wallpaper change every **10 seconds** while testing.

Run:

```bash
~/.local/bin/wallpaper-rotate.sh
```

You should see:

```text
Changing wallpaper to:
/home/lord/Pictures/Wallpapers/wallpaper1.png

Changing wallpaper to:
/home/lord/Pictures/Wallpapers/wallpaper2.jpg
```

If everything works, press:

```text
Ctrl+C
```

Then change the interval back to:

```bash
INTERVAL=600
```

`600 seconds = 10 minutes`.

---

## 6. Configure Hyprland Startup

Open your Hyprland configuration:

```bash
nano ~/.config/hypr/hyprland.conf
```

Add:

```ini
exec-once = awww-daemon
exec-once = sleep 2 && ~/.local/bin/wallpaper-rotate.sh
```

The `sleep 2` gives the `awww` daemon a moment to start before the rotation script begins.

Reload Hyprland:

```bash
hyprctl reload
```

---

## 7. Verify `awww`

Check whether the daemon is running:

```bash
pgrep -a awww
```

You should see the `awww-daemon` process.

You can also manually test a wallpaper:

```bash
awww img "$HOME/Pictures/Wallpapers/wallpaper1.png"
```

---

## 8. Troubleshooting

### `awww: command not found`

Check:

```bash
which awww
```

If nothing is returned:

```bash
sudo pacman -S awww
```

---

### `awww-daemon: command not found`

Check:

```bash
which awww-daemon
```

Then verify what the package installed:

```bash
pacman -Ql awww | grep -E '/bin/'
```

---

### Wallpaper directory doesn't exist

Check:

```bash
ls ~/Pictures/
```

Create it if necessary:

```bash
mkdir -p ~/Pictures/Wallpapers
```

---

### Case-sensitive paths

This is valid:

```text
/home/lord/Pictures/Wallpapers/
```

This is a different path:

```text
/home/lord/Pictures/wallpapers/
```

Linux treats uppercase and lowercase letters as different.

---

### Multiple `awww-daemon` processes

Don't run several daemons simultaneously.

Check:

```bash
pgrep -a awww
```

If necessary:

```bash
pkill -x awww-daemon
```

Then start it again:

```bash
awww-daemon &
```

---

## 9. Change the Rotation Time

The interval is controlled by:

```bash
INTERVAL=600
```

Examples:

| Interval   |  Value |
| ---------- | -----: |
| 30 seconds |   `30` |
| 1 minute   |   `60` |
| 5 minutes  |  `300` |
| 10 minutes |  `600` |
| 15 minutes |  `900` |
| 30 minutes | `1800` |
| 1 hour     | `3600` |

For example, 30 minutes:

```bash
INTERVAL=1800
```

---

## 10. Final Configuration

Your setup should look like:

```text
~/
├── .config/
│   └── hypr/
│       └── hyprland.conf
│
├── .local/
│   └── bin/
│       └── wallpaper-rotate.sh
│
└── Pictures/
    └── Wallpapers/
        ├── wallpaper1.jpg
        ├── wallpaper2.png
        ├── wallpaper3.webp
        └── wallpaper4.jpg
```

### Hyprland

```ini
exec-once = awww-daemon
exec-once = sleep 2 && ~/.local/bin/wallpaper-rotate.sh
```

### Script

```bash
WALLPAPER_DIR="$HOME/Pictures/Wallpapers"
INTERVAL=600
```

This gives you:

* Random wallpaper selection
* JPG/JPEG/PNG/WEBP support
* Automatic wallpaper changes
* 10-minute rotation
* Smooth transitions
* Automatic `awww` daemon startup
* Hyprland startup integration
