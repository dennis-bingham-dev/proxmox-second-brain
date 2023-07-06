## Setting up

## Installing ProxMox

I flashed the Proxmox ISO and installed it. was pretty painless. The default user is `root` (NOTE: in proxmox 8 it can be `root` or `admin` depending on how you install) from installing. And then you use the password provided during installation.

### Extra Tweaks

There are some amazing scripts to tweak Proxmox [here](https://tteck.github.io/Proxmox/). Some notably of use are:

- Proxmox VE 7 Post Install - lets you remove the popups to buy Proxmox and issues when updating the software
	- `bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/update-lxcs.sh)"`
- Proxmox Discord Dark Theme - Because dark :)
	- `bash <(curl -s https://raw.githubusercontent.com/Weilbyte/PVEDiscordDark/master/PVEDiscordDark.sh ) install`
- Plex Media Server LXC - Which has a Plex server with Hardware Accelerated Decoding
	- `bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/plex-v4.sh)"`

### Adding Container Templates

You can manually download ISO's and upload them to the server, but it has a lot ready to go with some command line stuff. You can use this command to list out all the available images:
```bash
root@homelab# pveam available --section system
unable to open file '/var/lib/pve-manager/apl-info/releases.turnkeylinux.org' - No such file or directory
...
system          alpine-3.15-default_20211202_amd64.tar.xz
system          archlinux-base_20211202-1_amd64.tar.zst
system          ubuntu-22.04-standard_22.04-1_amd64.tar.zst
...
```

And then you can download it like so with the filename:
```bash
root@homelab# pveam download local [image-name you select]
```

```bash
root@homelab# pveam download local archlinux-base_20211202-1_amd64.tar.zst
```

