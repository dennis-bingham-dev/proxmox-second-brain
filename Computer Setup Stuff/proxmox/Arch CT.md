### If you see an error like this `error: failed to commit transaction (invalid or corrupted package (PGP signature))` Then run these commands.

It is easiest to do this as root

run
```bash
pacman-key --init
```

then run
```bash
pacman-key --populate archlinux
```

then run
```bash
pacman-key --refresh-keys
```

then run updates
```bash
pacman -Syu
```

then install sudo
```bash
pacman -S sudo
```

Add user with permissions
`-m` is for generating the home dir for the new user
`-G wheel` is to add the new user to the wheel group
`-s /bin/bash` sets the shell for the new user

```bash
useradd -m -G wheel -s /bin/bash [username]
```

give the new user a password:
```bash
passwd [username]
```
follow the on screen prompts.

then allow the wheel group to use sudo for any command with password
run
```bash
visudo
```
or
```bash
vi /etc/sudoers
```

Then uncomment the line that says:
```bash
## Uncomment to allow members of group wheel to execute any command
# %wheel ALL=(ALL:ALL) ALL
```
you should be left with:
```bash
## Uncomment to allow members of group wheel to execute any command
%wheel ALL=(ALL:ALL) ALL
```
`Note: there is also a location to uncomment if you want to use sudo without the user password. If you want, be careful`
```bash
## Same thing without a password
# %wheel ALL=(ALL:ALL) NOPASSWD: ALL
```

let's test that it all works:
```bash
su - [username]
```

now run
```bash
sudo pacman -Syu
```

If it worked, congrats! You created a new sudo user in Arch Linux!


## Install Docker
```bash
sudo pacman -S docker
```

verify the install with
```bash
sudo docker version
```

lets start the service and verify it is up
```bash
sudo systemctl start docker.service
```
then
```bash
sudo systemctl status docker.service
```
hit `q` to exit

If that all worked lets enable the service to start at boot
```bash
sudo systemctl enable docker.service
```

If you are like me and don't like to use sudo to use docker we can add our user to the docker group
```bash
sudo usermod -aG docker $USER
```

then we need to logout and log back in we can check that it worked with
```bash
docker version
```
if you see the docker version and no errors, Congrats! You can now use docker without having to type sudo all the time.

## Portainer Install (if desired)
let's create the portainer volume:
```bash
docker volume create portainer_data
```

then we'll do:
```bash
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

Now lets check that it is running:
```bash
docker ps
```

We should see something like this:
```bash
CONTAINER ID   IMAGE    COMMAND  CREATED     STATUS      PORTS    NAMES
b943b408575b   portainer/portainer-ce:latest   "/portainer"   5 seconds ago   Up 2 seconds   0.0.0.0:8000->8000/tcp, :::8000->8000/tcp, 0.0.0.0:9443->9443/tcp, :::9443->9443/tcp, 9000/tcp   portainer
```

now navigate in your browser to `https://[your-server-ip]:9443` and continue the setup process there.

## Hook up an CIFS share at boot (if desired)

```bash
sudo pacman -Sy noconfirm cifs-utils
```

create a `.smbcredentials` file and add these lines using nano:
```
username=samba user in truenas
password=password for samba user in truenas
```
then hit `ctrl + o` then `ctrl + x`  that will write the file and exit.

then we need to change the credentials file permissions
```bash
chmod 600 .smbcredentials
```

create mount point:
```bash
sudo mkdir -p /mnt/[desired-directory]
```

now lets add your CIFS share to fstab to be mounted on boot
```bash
sudo vi /etc/fstab
```

we want to add a line:
```bash
//[your ip address]/[share name]/ /mnt/[mount name] cifs credentials=/home/[current logged-in user]/.smbcredentials 0 0 # for me it's dennis
```

For me it is:
```bash
//192.168.1.33/media-share-large/ /mnt/media cifs credentials=/home/dennis/.smbcredentials 0 0
```

We are going to check the config is correct with:
```bash
sudo mount -a
```

let's check it was successful:
```bash
ls /mnt/[mount-directory]
```

if everything has worked you should see your shares files/directories.

## Installing plex and jellyfin
```yaml
---
version: "2.1"
services:
  plex:
    image: lscr.io/linuxserver/plex:latest
    container_name: plex
    network_mode: host
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Boise
      - VERSION=docker
    volumes:
      - /var/lib/docker/volumes/plex-config:/config
      - /mnt/media/TV:/tv
      - /mnt/media/Movies:/movies
    restart: unless-stopped
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    network_mode: host
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Boise
    volumes:
      - /var/lib/docker/volumes/jellyfin-config:/config
      - /mnt/media/TV:/data/tvshows
      - /mnt/media/Movies:/data/movies
    ports:
      - 8096:8096
      - 8920:8920 #optional
      - 7359:7359/udp #optional
      - 1900:1900/udp #optional
    restart: unless-stopped
```

