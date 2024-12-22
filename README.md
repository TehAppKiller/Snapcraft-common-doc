# Contents
- [FAQ](#FAQ)
- [Building](#Building)
- [Versionning](#Versionning)

# FAQ
### How to access `/home` folder ?
- 1/ Snap Applications are launched as `user` ; Snap Services/Deamons are always launched as `root`
- 2/ Canonical allows snaps to read/write folders only from the one who launched the snap ; thus Applications are allowed to read/write in `user`-owned folders only ; Servics/Daemons are allowed to read/write in `root`-owned folders only.
- 3/ Thus, for Services/Daemons Canonical doesn't allow snaps to read base `/home` folder (except special authorizations), but you can read subfolders owned by `root` with following trick:

`removable-media` interface allows direct access to `/media` and `/mnt` folders ; do not forget to connect it:
```
sudo snap connect <snap_name>:removable-media
```

Easiest way to access your `/home` subfolder is by mounting a bind in the `/media` folder:
```
sudo mkdir /home/$USER/myfolder
sudo mkdir -p /media/home
sudo mount --bind /home/$USER/myfolder /media/home
```
You can now access `/home/$USER/myfolder` in the app through `/media/home`.

**To make it permanent**, this mount bind must be stored in `/etc/fstab`
```
sudo nano /etc/fstab
```
Add this line to the end of the file with your appropriate folders' modifications :
```
# Content of /etc/fstab

/home/<your_username>/myfolder/      /media/home      none      bind      0      0
```
To apply changes without restarting the OS :
```
sudo mount -a
sudo systemctl daemon-reload
```

# Building
## Contents
- [Build locally](#Build-locally)
- [Troubleshoot](#Troubleshoot)
- [Cross compilation with Core20](#Cross-compilation-with-Core20)
- [Cross compilation with Core24](#Cross-compilation-with-Core24)

## Build locally
Go in base directory and build:
```
snapcraft -v
```

## Troubleshoot
To resolve problems, you may check [snapcraft documentation](https://snapcraft.io/docs).\
Easy resolution is often to perform a clean:
```
snapcraft clean
```

## Cross compilation with Core20
> [!WARNING]
> Core20 ONLY ! This won't work with Core22 or Core24 :-)

Cross compilation requires target architectures repositories to be able to find the snap's required packages.

Add target architecture sources (armhf, arm64):
```
sudo dpkg --add-architecture armhf
sudo dpkg --add-architecture arm64
```
Add package repository sources at end of sources.list.d:
```
sudo nano /etc/apt/sources.list.d/ubuntu.sources
```
Notice `Architectures: amd64` added to the 2 original parts to avoid non-resolved ports\
(armhf and arm64 does not exist in http://archive.ubuntu.com/ubuntu ).\
Noble (Core24) and Focal (Core20) repositories are then added\
(armhf, arm64 and other different ports are in http://ports.ubuntu.com/ubuntu-ports ).
```
# This is the content of /etc/apt/sources.list.d/ubuntu.sources

# System repository (amd64 system here)
Types: deb
URIs: http://archive.ubuntu.com/ubuntu
Suites: noble noble-updates noble-backports
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
Architectures: amd64

Types: deb
URIs: http://security.ubuntu.com/ubuntu
Suites: noble-security
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
Architectures: amd64

# X-Compilation noble (armhf, arm64)
Types: deb
URIs: http://ports.ubuntu.com/ubuntu-ports
Suites: noble noble-updates noble-backports
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
Architectures: armhf arm64

Types: deb
URIs: http://ports.ubuntu.com/ubuntu-ports
Suites: noble-security
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
Architectures: armhf arm64

# X-Compilation focal (armhf, arm64)
Types: deb
URIs: http://ports.ubuntu.com/ubuntu-ports
Suites: focal focal-updates focal-backports focal-security
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
Architectures: armhf arm64
```
- [ ] TODO: Add library dependencies checking info
      
Build with:
```
snapcraft --enable-experimental-target-arch --target-arch=armhf --destructive-mode
snapcraft --enable-experimental-target-arch --target-arch=arm64 --destructive-mode
```

## Cross compilation with Core24
LXD/LXC supports only virtual machine of same architecture, thus the idea of cross-building snaps has been dropped.
Yet still possible:

### Canonical Remote build
Extract from https://snapcraft.io/docs/remote-build :

There are two methods for building snaps on Canonical-hosted servers, and both are available to every snap publisher:
* **[Build from GitHub](https://snapcraft.io/docs/build-from-github)**\
This is a build service integrated into every publisher’s Developer Account on snapcraft.io. It works by linking a snap’s GitHub repository with our Launchpad build service. See Build from GitHub for further details.
* **Snapcraft remote-build**\
The snapcraft remote-build command offloads the snap build process to the Launchpad build farm, pushing the potentially multi-architecture snap back to your machine. See below for further details.
> [!WARNING]
> But beware of the version of Snapcraft on the servers ; the farm may fail on some ports while it will work locally (w/ or w/o cross-compilation).

### From a Virtual Machine : QEMU
- For building, it's possible, but good luck and you must be trustful in a system which can even not output the console properly :-)\
- For betatesting, it does quite the job.
- [ ] TODO: Add QEMU img section

### Using dedicated hardware
Given you have enough resources (especially RAM), what a very nice option to compile directly on the arch !

Some boards:
* For ARMHF and ARM64:
[Rasberry PI 5 or 4](https://www.raspberrypi.com) will do the job

* For RISC-V64:
[BeagleV®-Ahead](https://www.beagleboard.org/boards/beaglev-ahead) \
More or less supported/open/working boards on https://wiki.ubuntu.com/RISC-V/
> [!WARNING]
> Current version of Snapcraft (8.4.1) fails to pack snaps

# Versionning
Current versionning of my snaps follows the exact same one from the source.\
If a re-build is needed, new build version appends source version with a '-v2' ; following builds increment this one '-v3', '-v4',... etc\
Even if not ideal, to simplify the management with snapcraft, re-build increment is based on current stable published version for now (i.e. there can be several different -v2 rc/beta/edge releases published ; but each stable release has a different version)
- [ ] TODO: Modifiy if there is a way with snapcraft to have build versionning and update with final version
