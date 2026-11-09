# chromium (fork)

A Docker image running the [Chromium](https://www.chromium.org/chromium-projects/) browser, streamed to your own browser through [Selkies](https://github.com/linuxserver/docker-baseimage-selkies).

This is a fork of [linuxserver/docker-chromium](https://github.com/linuxserver/docker-chromium), maintained at [PabloVTM/docker-chromium](https://github.com/PabloVTM/docker-chromium). The Selkies base image and the launch scripts come from upstream.

## Image

Images are published to GitHub Container Registry for `linux/amd64` and `linux/arm64`:

| Tag | Content |
| --- | --- |
| `ghcr.io/pablovtm/chromium:latest` | Latest build of `master`. |
| `ghcr.io/pablovtm/chromium:YYYY-MM-DD` | Dated version, e.g. `2026-11-07`. Each build upgrades the Debian packages, so a new dated version also brings the latest security fixes and Chromium release. |

## Quick start

```yaml
---
services:
  chromium:
    image: ghcr.io/pablovtm/chromium:latest
    container_name: chromium
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - CHROME_CLI=https://www.example.com/ #optional
    volumes:
      - ./config:/config
    ports:
      - 3000:3000
      - 3001:3001
    shm_size: "1gb"
    restart: unless-stopped
```

```bash
docker compose up -d
```

Then open **https://localhost:3001** (note the `https`) and accept the self-signed certificate warning.

To build the image yourself, replace `image:` with `build: .`.

For a quick throwaway test:

```bash
docker run --rm -it --shm-size=1gb -p 3001:3001 ghcr.io/pablovtm/chromium:latest bash
```

## Security

> [!WARNING]
> There is **no authentication by default**, and the web interface includes a terminal with passwordless `sudo`. Anyone who can reach the GUI can get root inside the container. Do not expose it to the Internet as is.

* `CUSTOM_USER` and `PASSWORD` enable HTTP basic auth. This is only suitable for a trusted local network.
* For Internet access, put the container behind a reverse proxy with real authentication.
* HTTPS is required for video and audio (WebCodecs). Port `3001` serves HTTPS with a self-signed certificate; port `3000` is plain HTTP and is meant only for a reverse proxy. If your proxy validates certificates, disable that check for this container.

## Environment variables

| Variable | Default | Description |
| --- | --- | --- |
| `PUID` / `PGID` | `1000` | User and group that own files in `/config`. |
| `TZ` | `Etc/UTC` | Timezone, see the [tz database list](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List). |
| `CHROME_CLI` | — | Extra Chromium command-line flags or a start URL, passed as is. |
| `CUSTOM_USER` / `PASSWORD` | — | Enable HTTP basic auth. |
| `LC_ALL` | — | Session language, e.g. `fr_FR.UTF-8`, `de_DE.UTF-8`, `ja_JP.UTF-8`. |
| `AUTO_GPU` | `true` | Set to `false` to disable automatic GPU detection. |
| `PIXELFLUX_WAYLAND` | `true` | Run Chromium on Wayland. |
| `UMASK` | — | Umask for services, e.g. `022`. |
| `FILE__<VAR>` | — | Read `<VAR>` from a file, e.g. for Docker secrets. |

All other Selkies settings (ports, subfolder, stream and UI options) are listed in the [Selkies configuration reference](https://docs.linuxserver.io/selkies/user-guide/configuration/).

## Volumes and ports

| Setting | Purpose |
| --- | --- |
| `-p 3000:3000` | HTTP GUI, for use behind a reverse proxy only. |
| `-p 3001:3001` | HTTPS GUI. |
| `./config:/config` | Home directory: Chromium profile, downloads and settings. |
| `--shm-size=1gb` | Shared memory; required for most modern websites (video in particular). |

## GPU acceleration

A GPU is optional. When one is passed in, it is detected and used automatically.

* **Intel / AMD**: add `--device /dev/dri` (compose: `devices: [/dev/dri:/dev/dri]`).
* **Nvidia**: requires host driver 580 or newer. Add `--runtime nvidia --gpus all --device /dev/nvidia-modeset`. See the [GPU guide](https://docs.linuxserver.io/selkies/user-guide/gpu/) for older drivers.

## Operations

```bash
# Shell in the container
docker exec -it chromium /bin/bash

# Follow logs
docker logs -f chromium
```

To update, pull the new image (`docker compose pull`) and recreate the container (`docker compose up -d`). Your `/config` volume is kept.

## Releasing

Images are built by GitHub Actions. A push to `master` publishes `latest`; a dated version is published by pushing a tag named after the day:

```bash
git tag 2026-11-09
git push origin 2026-11-09
```

The workflow tests the image first, then publishes `ghcr.io/pablovtm/chromium:2026-11-09` and `latest` for `linux/amd64` and `linux/arm64`. There is no scheduled rebuild: tag a new version when you want one.

## Building

```bash
docker build --no-cache --pull -t chromium:latest .
```

For ARM images on x86 hardware (or the reverse), register QEMU first:

```bash
docker run --rm --privileged lscr.io/linuxserver/qemu-static --reset
docker build -f Dockerfile.aarch64 -t chromium:arm64 .
```

## Credits

Chromium is developed by the Chromium project. The Selkies base image, desktop streaming and original setup are from [LinuxServer.io](https://linuxserver.io), upstream at [linuxserver/docker-chromium](https://github.com/linuxserver/docker-chromium).
