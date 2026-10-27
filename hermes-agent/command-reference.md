# Command Reference: Hermes Agent on Raspberry Pi 4

*Running log of every command used to get the system up and running. Last updated 2026-10-22 02:07 ET.*

## Rules

- Run commands one at a time; wait for the prompt to return before the next.
- Paste in the Pi Terminal: Ctrl+Shift+V. Paste in Windows PowerShell (SSH): right-click.
- Check where you are: 'duppy_coder@hermes-pi' = on the Pi. 'PS C:\...' = on your Windows PC.
- Only type your password at a real password prompt ('[sudo] password for ...' or '...'s password:').
- Never put API keys, passwords, or tokens on the command line, in notes, or in photos.
- Use Tab completion for paths to avoid typos.

## 1. Basic navigation and checks

**Where:** Pi

**Clear the screen (or press Ctrl+L)**

```
clear
```

**Show free disk space**

```
df -h /
```

**Show memory**

```
free -h
```

**Show the Pi's network address**

```
hostname -I
```

**Show settings the Pi booted with**

```
cat /proc/cmdline
```

## 2. System updates

**Where:** Pi

**Refresh the list of available software**

```
sudo apt update
```

**Install all updates**

```
sudo apt full-upgrade -y
```

Note: Can take 10-20 minutes.

**Confirm nothing is left to update**

```
apt list --upgradable
```

Note: No output = fully up to date.

**Restart the Pi**

```
sudo reboot
```

## 3. Display fix (Sharp TV, Wayland)

**Where:** Pi (on the Pi's own Terminal, not SSH)

**List displays and supported modes**

```
wlr-randr
```

Note: Output name for this TV: HDMI-A-2.

**Set the no-crop mode (temporary until reboot)**

```
wlr-randr --output HDMI-A-2 --mode 1360x768
```

**Fallback if the screen goes blank**

```
wlr-randr --output HDMI-A-2 --mode 1280x720@60Hz
```

**Confirm the desktop runs user autostart files**

```
cat /etc/xdg/labwc/autostart
```

Note: Look for lxsession-xdg-autostart.

**Create the autostart folder and file**

```
mkdir -p ~/.config/autostart
nano ~/.config/autostart/display-fix.desktop
```

Note: Never create ~/.config/labwc/autostart (it replaces the system file).

**File contents (4 lines)**

```
[Desktop Entry]
Type=Application
Name=Display Fix
Exec=sh -c "sleep 3 && wlr-randr --output HDMI-A-2 --mode 1360x768"
```

Note: Save: Ctrl+O, Enter, Ctrl+X.

**Check the file**

```
cat ~/.config/autostart/display-fix.desktop
```

Note: Known gap: resets after screen sleep. kanshi fix pending.

## 4. Backing up a system file

**Where:** Pi

**Back up the boot command line before editing**

```
sudo cp /boot/firmware/cmdline.txt /boot/firmware/cmdline.txt.bak
```

**Confirm the backup exists**

```
ls /boot/firmware/cmdline.txt.bak
```

## 5. Password incident recovery

**Where:** Pi

**Clear shell history and change the password**

```
clear
history -c && history -w
passwd
```

Note: Use if a password is ever typed at a normal prompt.

## 6. Install Hermes Agent (safe method)

**Where:** Pi

**Download the installer without running it**

```
curl -fsSL https://hermes-agent.nousresearch.com/install.sh -o install.sh
ls -l install.sh
```

**Read it (q to quit)**

```
less install.sh
```

**Check no download-redirect variable is set (should print blank)**

```
echo $HERMES_REPO_URL
```

**List every web address the script contacts**

```
grep -oE 'https?://[^" )]+' install.sh | sort -u
```

Note: Expect only github.com/NousResearch, github.com/astral-sh/uv, *.nousresearch.com.

**Run the installer, reload the shell, health check**

```
bash install.sh
source ~/.bashrc
hermes doctor
```

## 7. SSH from Windows PC to the Pi

**Where:** Pi, then PC

**On the Pi: confirm SSH is running**

```
systemctl is-active ssh
```

Note: Should say active.

**On the Pi: get the host key fingerprint**

```
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

Note: Expected: SHA256 starting AGVlkj, ending ZyWNwg.

**On the PC (PowerShell): connect**

```
ssh duppy_coder@<Pi LAN address>
```

Note: Type 'yes' only if the fingerprint matches. Type the password promptly.

## 8. Install Docker Engine (official repo)

**Where:** Pi

**Remove conflicting packages**

```
sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-doc docker-buildx podman-docker containerd runc | cut -f1)
```

**Add Docker's signing key**

```
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

**Verify the key fingerprint**

```
gpg --show-keys /etc/apt/keyrings/docker.asc
```

Note: Must be 9DC858229FC7DD38854AE2D88D81803C0EBFCD88.

**Add Docker's repository (paste all lines at once)**

```
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
sudo apt update
```

Note: Expect Suites: trixie, Architectures: arm64.

**Install and test**

```
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo docker run hello-world
```

**Allow your user to run Docker without sudo**

```
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world
```

Note: Risk: docker group = root-level privileges.

## 9. Hermes management

**Where:** Pi

**Health check**

```
hermes doctor
```

**Change model/provider (re-enter API key)**

```
hermes setup model
```

Note: Paste the key once; nothing else in the field.

**Change terminal backend**

```
hermes setup terminal
```

**Configure messaging (Telegram later)**

```
hermes setup gateway
```

**Configure tool providers**

```
hermes setup tools
```

**Turn tools on/off**

```
hermes tools
```

**Telemetry setting**

```
hermes setup telemetry
```

**View or edit config**

```
hermes config
hermes config edit
```

Note: Secrets live in ~/.hermes/.env. Never photograph it.

**Start chatting**

```
hermes
```

## 10. Hermes gateway service checks

**Where:** Pi

**Should it start at boot? / Is it running?**

```
systemctl --user is-enabled hermes-gateway
systemctl --user is-active hermes-gateway
```

Note: Current target: disabled / inactive.

## 11. GitHub (Windows PC)

**Where:** PC

**Check Git and identity**

```
git --version
git config --global --list
```

**Sign in and clone**

```
git credential-manager github login
git clone https://github.com/duppy-c/cybersecurity-labs.git
```

**Push notes**

```
cd "$env:USERPROFILE\Documents\cybersecurity-labs"
git status
git add session-notes/<file>
git commit -m "<message>"
git push
```

## 12. Shut down or restart the Pi

**Where:** Pi

**Shut down safely (wait until the green light stops blinking before unplugging)**

```
sudo shutdown now
```

**Restart**

```
sudo reboot
```

## 13. Update Hermes (check for security fixes)

**Where:** Pi

**Update Hermes to the latest version**

```
hermes update
```

Note: Run regularly; the npm vulnerabilities found 2026-10-22 are fixed upstream, not locally. Do not run `hermes doctor --fix` until its changes are reviewed.
