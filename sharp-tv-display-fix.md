# Lab Reference: Sharp TV Display Fix for Raspberry Pi

Created: 2026-10-21 08:45 (America/New_York)
Verified working on: Raspberry Pi 4, Raspberry Pi OS 64-bit (Debian Trixie, Wayland/labwc)
Also verified on: Parrot OS on Pi 4 (X11), session 2026-10-20

## The problem

The Sharp TV crops the edges of the picture (overscan) at TV modes like 1280x720 and 1920x1080.
Symptoms: top bar cut off, left side clipped ("Wastebasket" label cut).

## The fix

Use **1360x768**. The TV treats it as a computer mode and shows it pixel for pixel, with no cropping.

## What does NOT work (don't waste time on these)

- Changing to 1920x1080: still cropped.
- Kernel margins in /boot/firmware/cmdline.txt (video=...,margin_left=...): the Wayland desktop ignores them.
- Making a ~/.config/labwc/autostart file: it REPLACES the system startup file and you lose the taskbar and desktop. Never create it.

## Step 1: Find out which display system the OS uses

```
echo $XDG_SESSION_TYPE
```

- Prints **wayland**: use the Wayland steps (wlr-randr).
- Prints **x11**: use the X11 steps (xrandr).

## Step 2: Find the output name

The name depends on the OS and which HDMI port is used.

Wayland:

```
wlr-randr
```

X11:

```
xrandr
```

Known names for this TV on the second HDMI port: **HDMI-A-2** (Raspberry Pi OS, Wayland) and **HDMI-2** (Parrot OS, X11).

## Step 3: Test the fix (temporary, resets at reboot)

Wayland:

```
wlr-randr --output HDMI-A-2 --mode 1360x768
```

X11:

```
xrandr --output HDMI-2 --mode 1360x768
```

If the screen goes blank, wait 15 seconds, then type the command again with 1280x720@60Hz (Wayland) or 1280x720 (X11).

## Step 4: Make it permanent (runs at every login)

1. Confirm the desktop runs user autostart files (Raspberry Pi OS). Look for a line containing **lxsession-xdg-autostart**:

```
cat /etc/xdg/labwc/autostart
```

2. Create the autostart folder:

```
mkdir -p ~/.config/autostart
```

3. Create the file. The path starts with a tilde (~), not a slash:

```
nano ~/.config/autostart/display-fix.desktop
```

4. Type these four lines (Wayland version):

```
[Desktop Entry]
Type=Application
Name=Display Fix
Exec=sh -c "sleep 3 && wlr-randr --output HDMI-A-2 --mode 1360x768"
```

For X11, change only the Exec line:

```
Exec=sh -c "sleep 3 && xrandr --output HDMI-2 --mode 1360x768"
```

5. Save and exit: Ctrl+O, Enter, Ctrl+X.

6. Check the file before rebooting:

```
cat ~/.config/autostart/display-fix.desktop
```

7. Reboot:

```
sudo reboot
```

Expected result: the login screen is still cropped (normal). The desktop snaps to fit about 3 seconds after login.

8. Confirm. The line marked "current" should say 1360x768:

```
wlr-randr
```

## Mistakes made during setup (watch for these)

- "firmare" instead of "firmware", "cmline" or "cdline" instead of "cmdline": use Tab completion.
- "HDMI-A2" instead of "HDMI-A-2": error says "unknown output".
- Missing closing double quote at the end of the Exec line: the command won't run.
- Leading slash instead of a tilde in the autostart path (Parrot session).

## Security lessons from this fix

- Back up a system file before editing it:

```
sudo cp /boot/firmware/cmdline.txt /boot/firmware/cmdline.txt.bak
```

- sudo remembers your password for about 15 minutes. Only type your password when you see a password prompt. If you type it at a normal prompt, it lands in command history. Recovery:

```
clear
```

```
history -c && history -w
```

```
passwd
```

## Sources

- [wlr-randr manual (Debian Trixie)](https://dyn.manpages.debian.org/trixie/wlr-randr/wlr-randr.1)
- [labwc-config manual (config file precedence)](https://man.archlinux.org/man/labwc-config.5)
- [wlroots issue #3597: DRM overscan margins handling](https://gitlab.freedesktop.org/wlroots/wlroots/-/issues/3597)

## Update 2026-10-28: making it survive screen sleep and port changes

- The output name depends on which micro-HDMI port the cable is in: **HDMI-A-1** or **HDMI-A-2**. If the fix stops working, run `wlr-randr` and check the first line for the current name.
- The autostart file only runs at login. **kanshi** (started by the desktop) reapplies settings when the screen wakes or reconnects.
- Working kanshi file (`~/.config/kanshi/config`, written by the Pi's Screen Configuration tool), covering both ports:

```
profile {
    output HDMI-A-1 enable scale 1.000000 mode 1360x768@60.015 position 0,0
}

profile {
    output HDMI-A-2 enable scale 1.000000 mode 1360x768@60.015 position 0,0
}
```

- Reload kanshi after editing:

```
killall -s SIGHUP kanshi
```

- Fix the autostart file's port name if needed:

```
sed -i 's/HDMI-A-2/HDMI-A-1/' ~/.config/autostart/display-fix.desktop
```

- Verified 2026-10-28: setting held through screen off/on and a reboot.
