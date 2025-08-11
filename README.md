[![License](https://img.shields.io/github/license/FreeGameHosting/cncnet-server?style=for-the-badge&color=green "License")](https://github.com/FreeGameHosting/cncnet-server/blob/main/LICENSE)[![Amount of contributers](https://img.shields.io/github/contributors/FreeGameHosting/cncnet-server?style=for-the-badge&color=blue "Amount of contributers")](https://github.com/FreeGameHosting/cncnet-server/graphs/contributors)[![Forks](https://img.shields.io/github/forks/FreeGameHosting/cncnet-server?style=for-the-badge&color=blue "Forks")](https://github.com/FreeGameHosting/cncnet-server/forks)  \
[![Last commit](https://img.shields.io/github/last-commit/FreeGameHosting/cncnet-server?style=for-the-badge "Last commit")](https://github.com/FreeGameHosting/cncnet-server/commits/main)[![Commit activity](https://img.shields.io/github/commit-activity/m/FreeGameHosting/cncnet-server?style=for-the-badge "Commit activity")](https://github.com/FreeGameHosting/cncnet-server/graphs/commit-activity)[![State of the latest container build](https://img.shields.io/github/actions/workflow/status/FreeGameHosting/cncnet-server/container_refresh.yml?style=for-the-badge "State of the latest container build")](https://github.com/FreeGameHosting/cncnet-server/actions/workflows/container_refresh.yml) \
[![Open issues](https://img.shields.io/github/issues/FreeGameHosting/cncnet-server?style=for-the-badge "Open issues")](https://github.com/FreeGameHosting/cncnet-server/issues)[![Open Pull Requests](https://img.shields.io/github/issues-pr/FreeGameHosting/cncnet-server?style=for-the-badge "Open Pull Requests")](https://github.com/FreeGameHosting/cncnet-server/pulls)

# CnCNet server / tunnel container image
This repository contains a Containerfile which builds a container image with the CnCNet server binary installed in it.

The container is based on the alpine image and is therefor a lot smaller compared to the ubuntu version. The binary is running as the `cncnet` user within the container instead of the `root` user.

# The published container image
The container image is built and published to the Github container registry automatically. This workflow is scheduled every day to guarantee that both the cncnet server binary and system packages are up-to-date.

# Installation
You can use the Containerfile to build the container or use the published container image from this repository.

## Build the container image yourself.
Make sure that you checked out and changed directory to this repository.

Run the following command to build the container (I will use `podman`):
```bash
podman build -f Containerfile-alpine -t cncnet-server:latest
```

## Pull the container image from the Github registry.
Run the following command to pull the container image from the Github registry
```bash
podman pull ghcr.io/freegamehosting/cncnet-server:latest
```

# Start the container image
To start the container image, you can use the following arguments:
```bash
podman run -d \
    -p 50000:50000/tcp \
    -p 50000:50000/udp \
    -p 50001:50001/udp \
    -p 8054:8054/udp \
    -p 3478:3478/udp \
    --restart unless-stopped \
    --pull newer \
    --name cncnet-server \
    cncnet-server:latest \
    --name "My CnCNet tunnel"
```
> [!NOTE]
> [Tunnel v2 ports: `50000/tcp` & `50000/udp`](https://github.com/Rans4ckeR/cncnet-server?tab=readme-ov-file#how-to-runinstall) \
> [Tunnel v3 ports: `50001/udp`](https://github.com/Rans4ckeR/cncnet-server?tab=readme-ov-file#how-to-runinstall)

## List all arguments
For a full list of arguments, use the following command:
```bash
podman run \
    --rm -ti \
    cncnet-server:latest \
    -h
```

## Automatically update and start with OS
In case you're using `podman`, make sure you created the `cncnet` user on your server and that it is allowed to run containers using `podman`.

Install the `systemd` unit file, using the follwing command (adjust accordingly where needed):
```bash
cat << 'EOF' > /etc/systemd/system/cncnet-server.service && systemctl daemon-reload && systemctl --now enable cncnet-server.service
[Unit]
Description=CnCNet tunnel server
After=network-online.target
     
[Service]
Type=exec
User=cncnet
Restart=always
ExecStart=/usr/bin/podman run \
    -ti --rm --replace \
    -p 50000:50000/tcp \
    -p 50000:50000/udp \
    -p 50001:50001/udp \
    -p 8054:8054/udp \
    -p 3478:3478/udp \
    --pull newer \
    --name cncnet-server \
    ghcr.io/freegamehosting/cncnet-server:latest \
    --name "My CnCNet tunnel"
ExecStop=/usr/bin/podman stop cncnet-server
ExecStopPost=/usr/bin/podman system prune \
    --filter "label=org.opencontainers.image.title=CnCNet tunnel server" \
    --force
     
[Install]
WantedBy=multi-user.target
EOF
```

## Contact
[![My Discord (YOURUSERID)](https://img.shields.io/badge/My-Discord-%235865F2.svg)](https://discord.com/users/370120292665917443)
