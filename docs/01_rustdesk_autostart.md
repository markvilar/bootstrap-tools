# RustDesk Autostart Configuration

Goal: Make RustDesk available after a reboot, without needing to log in via the GUI first.


## Option A: Auto-login + RustDesk as a user service (recommended)

Configure your display manager to auto-login so a desktop session starts immediately on
boot, then configure RustDesk to start with that session.

### Step 1 — Enable auto-login (choose the section for your display manager):

```text
GDM (/etc/gdm3/custom.conf or /etc/gdm/custom.conf):
[daemon]
AutomaticLoginEnable=true
AutomaticLogin=yourusername

LightDM (/etc/lightdm/lightdm.conf):
[Seat:*]
autologin-user=yourusername
autologin-user-timeout=0

SDDM (/etc/sddm.conf):
[Autologin]
User=yourusername
Session=plasma   # or gnome, etc.
```

### Step 2 — Enable RustDesk as a systemd user service:

```shell
systemctl --user enable rustdesk
systemctl --user start rustdesk
sudo loginctl enable-linger yourusername
```

The enable-linger command makes user services persist after logout and start at boot.

Security note: Auto-login removes the lock-screen barrier on physical access. If the
machine is in a physically accessible location, weigh that trade-off.


### Configuring `rustdesk` as a systemd service

You need to create the user service unit file manually. Run these commands:

```shell
mkdir -p ~/.config/systemd/user
```

Then create `~/.config/systemd/user/rustdesk.service` with this content:

```text
[Unit]
Description=RustDesk
After=graphical-session.target

[Service]
Type=simple
ExecStart=/usr/bin/rustdesk
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

Then enable and start it:

```shell
systemctl --user daemon-reload
systemctl --user enable --now rustdesk
sudo loginctl enable-linger $USER
```

Check the RustDesk binary path first — if it's not at /usr/bin/rustdesk, adjust ExecStart accordingly:

```shell
which rustdesk
```


## Option B: RustDesk system service (access at login screen)

RustDesk can run as a system daemon that provides access even before login. This is
useful if you do not want auto-login, but note that you will only see the login screen
greeter until a user session is active.

### Step 1 — Enable the service if it is already installed:

```shell
sudo systemctl enable rustdesk
sudo systemctl start rustdesk
```

### Step 2 — If no service unit is installed, create /etc/systemd/system/rustdesk.service:

```text
[Unit]
Description=RustDesk
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/rustdesk --service
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Then reload and enable:

```shell
sudo systemctl daemon-reload
sudo systemctl enable --now rustdesk
```

Caveat: Option B gives access to the login screen only. If no session is active you can
see and interact with the greeter, but not a full desktop. Use Option A if you need full
desktop control after every reboot.
