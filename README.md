# selfhosted-home-server

My home server config for selfhosting a bunch of services.
I use this in combination with [selfhosted-vps](https://github.com/Bhambya/selfhosted-vps).

## Installation

1. Install Proxmox VE server
1. Follow [Overriding network device names](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#network_override_device_names) so that network device names do not change after an update. Otherwise you might lose remote access to the server after reboot.
1. Create a debian VM for docker containers

## Docker

Login to the VM created above for docker containers

Install [Docker](https://docs.docker.com/engine/install/)

[Set up docker daemon.json](https://www.reddit.com/r/selfhosted/comments/1az6mqa/psa_adjust_your_docker_defaultaddresspool_size/).  Otherwise, you may end up with subnet ranges inside your containers that overlap with the real LAN and make hosts unreachable.
Edit the `/etc/docker/daemon.json` file:
``` json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "default-address-pools": [
    {
      "base" : "172.16.0.0/12",
      "size" : 24
    }
  ]
}
```

Clone this repo

```
cd selfhosted-home-server/docker
cp .env.example .env # edit this
docker compose up -d
```

## Backups
Set-up backups using [backrest](https://github.com/garethgeorge/backrest) (restic).

In case of data loss, restore the latest snapshot using [restic](https://github.com/restic/restic).

Backing up of following paths is recommended:

- Apprise config: `/var/lib/docker-data/apprise/config`
- Backrest config: `/var/lib/docker-data/backrest/config`
- Immich (only DB): `$IMMICH_UPLOAD_PATH/backups`

The following paths can have large amounts of data and should require different backup setup because the costs can be huge. I use Google cloud archive storage tier and disable the backrest `prune` and `check` schedules because of the huge egress costs.

- $IMMICH_UPLOAD_PATH/library - This is where Immich stores the media.  
- $IMMICH_UPLOAD_PATH/profile - Immich avatar images

If you do end up restoring the Immich backup. Run jobs in immich to regenerate thumbnails since they are not backed up.

