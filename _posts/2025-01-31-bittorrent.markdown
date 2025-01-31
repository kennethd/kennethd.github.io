# Headless torrents on the console with deluge
## install
`sudo apt install deluged deluge-console`
## Add override for systemd service to require mount point
From https://deluge.readthedocs.io/en/latest/how-to/systemd-service.html :

*"If you have a USB disk drive or network drive that may not be immediately
available on boot or disconnected at random then you may want the deluged
service to wait for mount point to be ready before starting. If they are
unmounted or disconnected then deluged is stopped. When they become available
again deluged is started."*

Package defaults are defined in `/lib/systemd/system/deluged.service`

To create a file to override/define new settings, use `systemctl edit`
```
kenneth@fado $ sudo -E systemctl edit deluged
```
This will create `/etc/systemd/system/deluged.service.d/override.conf` &
launch your editor to modify it.  Note `sudo -E` preserves environment, used
here to allow me to edit with vim, I couldn't get `systemctl edit` to respect
`EDITOR`, `VISUAL`, or `SYSTEMD_EDITOR` as set in `/root/.bashrc`, even with
`env_keep` mods to `visudo` (grumble grumble fuckin goddamn systemd pile of shit)

Edit the **[Unit]** & **[Install]** sections to look like below, `mnt-seagate`
is my USB drive where torrent files are written, to see the mount points
systemd recognizes on your system use the command `sudo systemctl -t mount`
```
kenneth@fado $ cat /etc/systemd/system/deluged.service.d/override.conf
[Unit]
Description=Deluge Bittorrent Client Daemon
Documentation=man:deluged
# Start after network and specified mounts are available.
After=network-online.target mnt-seagate.mount
Requires=mnt-seagate.mount
# Stops deluged if mount points disconnect
BindsTo=mnt-seagate.mount

[Install]
WantedBy=multi-user.target mnt-seagate.mount
```

