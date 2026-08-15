# Custom Ubuntu-MATE Webtop image

This directory builds an amd64 Ubuntu-MATE Webtop image with the GTK2 SpaceFM build and the selected applications. Build dependencies exist only in the intermediate SpaceFM builder image and are not copied into the final image.

The base is pinned to the current LinuxServer Ubuntu-MATE version tag:

```text
lscr.io/linuxserver/webtop:ubuntu-mate-version-37509c6b
```

Update `WEBTOP_VERSION` in `Dockerfile` when deliberately moving to a newer LinuxServer Webtop release.

## Build

Run from this directory on an amd64 machine with Docker Buildx:

```bash
mkdir -p spacefm-dist

docker buildx build \
  --platform linux/amd64 \
  --output type=local,dest=spacefm-dist \
  -f Dockerfile.spacefm \
  .

docker buildx build \
  --platform linux/amd64 \
  -t vttc08/webtop-ubuntu-mate:custom \
  -f Dockerfile \
  .
```

The first command creates `spacefm-dist/spacefm-gtk2-custom_1.0.6+gtk2.20260815_amd64.deb`. The second command installs that package and all runtime applications into the final image.

To use another SpaceFM package version, pass the same value to both builds:

```bash
export SPACEFM_VERSION='1.0.6+gtk2.20260815'

docker buildx build --platform linux/amd64 \
  --build-arg SPACEFM_VERSION="$SPACEFM_VERSION" \
  --output type=local,dest=spacefm-dist \
  -f Dockerfile.spacefm .

docker buildx build --platform linux/amd64 \
  --build-arg SPACEFM_VERSION="$SPACEFM_VERSION" \
  -t vttc08/webtop-ubuntu-mate:custom \
  -f Dockerfile .
```

## Save, copy, and load

On the build machine:

```bash
docker save vttc08/webtop-ubuntu-mate:custom | gzip > webtop-ubuntu-mate-custom.tar.gz
```

Copy the archive to the target server, then run:

```bash
gunzip -c webtop-ubuntu-mate-custom.tar.gz | docker load
```

The image can then be used in Docker Compose or with `docker run`. Example:

```bash
docker run -d \
  --name=webtop-custom \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Etc/UTC \
  -p 3000:3000 \
  -p 3001:3001 \
  --shm-size=1gb \
  -v /path/to/webtop-config:/config \
  --restart unless-stopped \
  vttc08/webtop-ubuntu-mate:custom
```

Use a persistent `/config` volume for MATE and application settings. Packages are part of the image, so recreating the container does not reinstall them.

## Validate SpaceFM

After starting the container:

```bash
docker exec webtop-custom sh -lc 'command -v spacefm; ldd /usr/bin/spacefm | grep -E "gtk|gdk"'
```

The output should show `/usr/bin/spacefm` and GTK2 libraries such as `libgtk-x11-2.0.so.0` and `libgdk-x11-2.0.so.0`.
