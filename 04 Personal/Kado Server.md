# KADO — Home Media Server

**Status:** Operational  
**Last documented:** October 2026  
**Purpose:** Private, local-network storage and streaming of photos, videos, and other files.

## 1. Hardware & Operating System

|Component|Configuration|
|---|---|
|Server|Raspberry Pi 4 Model B|
|RAM|4 GB|
|Operating System|Raspberry Pi OS Lite 64-bit (Debian 13)|
|System Storage|128 GB USB SSD|
|Media Storage|4 TB WD My Passport HDD|
|HDD Encryption|Windows BitLocker|
|Network|Gigabit Ethernet|
|Hostname|`kado`|
|Local IP|`192.168.0.211` (DHCP reserved)|
|Administration|SSH|

The OS boots directly from the SSD. No microSD card is required.

## 2. Storage Structure

```
128 GB SSD
/
└── srv/
    ├── docker/
    │   ├── jellyfin/
    │   │   ├── config/
    │   │   └── cache/
    │   └── photoprism/
    │       └── storage/
    │
    └── media/
        ├── uploads/
        │   └── iphone/       (Read/Write)
        └── hdd/              (Read-only)

4 TB Encrypted HDD
/dev/disk/by-uuid/
  e3602cdb-bb9a-4ab7-8d47-a245acaec392
```

**HDD mounting:**

- Encrypted with BitLocker, containing an NTFS filesystem.
    
- Unlocked using `dislocker`.
    
- Mounted read-only at `/srv/media/hdd`.
    
- Requires manual password entry after each reboot.
    

Manual unlock script:

```
~/mount-hdd.sh
```

## 3. Installed Services

|Service|Purpose|Access|
|---|---|---|
|Jellyfin|Video streaming|`http://192.168.0.211:8096`|
|PhotoPrism|Photo management and gallery|`http://192.168.0.211:2342`|
|Samba|Network file sharing|`smb://192.168.0.211`|
|Docker|Application containers|Local administration|
|UFW|Network firewall|LAN-only rules|
|SSH|Remote administration|Port 22|

### Jellyfin

- Runs inside Docker.
    
- Reads videos from the encrypted HDD.
    
- Media mounted read-only.
    
- Available through browser, Android TV, and supported clients.
    
- Direct playback preferred because Raspberry Pi 4 has limited transcoding capability.
    

### PhotoPrism

- Runs inside Docker.
    
- Photo library stored on the encrypted HDD.
    
- SSD upload folder exposed as `/photoprism/import`.
    
- Source folders mounted read-only.
    
- Thumbnails and application data stored on SSD.
    

### Samba

Two network shares:

|Share|Directory|Permission|
|---|---|---|
|`Media`|`/srv/media/hdd`|Read-only|
|`iPhoneUpload`|`/srv/media/uploads/iphone`|Read/Write|

Authentication uses the Linux username `hossein` with a separate Samba password.

## 4. System Architecture

```
                HOME NETWORK
                      |
               Raspberry Pi 4
               192.168.0.211
                      |
           +----------+----------+
           |                     |
      128 GB SSD           4 TB HDD
      OS + Docker          BitLocker
      App databases        Read-only
      iPhone uploads       Media archive
           |                     |
           +----------+----------+
                      |
               Applications
                      |
          +-----------+-----------+
          |           |           |
       Jellyfin   PhotoPrism    Samba
          |           |           |
       Android TV   iPhone     Ubuntu /
       Browsers     Browsers   iPhone Files
```

**iPhone photo workflow:**

1. iPhone Photos → SMB → `iPhoneUpload`.
    
2. Photos are stored on the SSD.
    
3. PhotoPrism can import photos from that location.
    
4. The original encrypted HDD remains read-only.
    

Automatic background photo upload has **not yet been configured**.

## 5. Security & Privacy

- Services are hosted locally.
    
- No router port forwarding configured.
    
- UFW allows access from `192.168.0.0/24`.
    
- SSH and Samba use password authentication.
    
- PhotoPrism and Jellyfin have application authentication.
    
- BitLocker encryption protects the HDD while locked.
    
- The main media archive is mounted read-only.
    
- BitLocker password is not stored for automatic unlocking.
    

**Important:** Local HTTP connections are not encrypted. Avoid exposing the services directly to the Internet.

## 6. Common Administration Commands

**Connect to server:**

```
ssh hossein@192.168.0.211
```

**Unlock and mount encrypted HDD:**

```
~/mount-hdd.sh
```

**Check server status:**

```
~/status-server.sh
```

**View Docker containers:**

```
docker ps
```

**Restart Jellyfin:**

```
docker restart jellyfin
```

**Restart PhotoPrism:**

```
docker restart photoprism
```

**Check HDD mount:**

```
findmnt /srv/media/hdd
```

**Check free storage:**

```
df -h
```

## 7. Current Status & Remaining Improvements

**Completed**

- Raspberry Pi OS installed on SSD.
    
- Docker configured and tested.
    
- BitLocker HDD accessible through manual unlock.
    
- Jellyfin video playback tested.
    
- PhotoPrism photo gallery tested.
    
- Samba shares working on Ubuntu and iPhone.
    
- Firewall enabled.
    
- Local IP reserved.
    
- Server status script created.
    
- HEIC viewing issue on Ubuntu addressed using Flatpak gThumb.
    

**Remaining**

- Automatic iPhone photo synchronization.
    
- Reliable backup strategy.
    
- SSH key authentication.
    
- Optional local HTTPS.
    
- Improve HDD mount script error handling.
    
- Verify PhotoPrism import and indexing workflow for SSD uploads.
    
- Consider deferring media-container startup until the encrypted HDD is mounted.
    

**Overall:** The server is functional as a private home media server. The main outstanding items are backup reliability and automatic photo uploads.