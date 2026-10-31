# Subsurface Docker

Run [Subsurface](https://subsurface-divelog.org/), the open-source dive log, in your browser using Docker.

This project packages the Subsurface desktop application in a lightweight Linux container and provides browser access through noVNC. You don't need to install Subsurface or a desktop environment on your computer.

**This is an independent, community-maintained Docker deployment. It is not an official Subsurface project or an officially supported distribution.**

## Features

- Browser-based access to the Subsurface desktop application.
- Based on the official Subsurface upstream source code.
- Qt 6 desktop build.
- Persistent application configuration and dive data.
- Runs as a non-root user inside the container.
- Automated Docker image builds and publishing through GitHub Actions.
- Published images available through GitHub Container Registry (GHCR).

## Quick start

The easiest way to run Subsurface is with Docker Compose.

```yaml
services:
  subsurface:
    image: ghcr.io/kellenwiltshire/subsurface-docker:latest
    container_name: subsurface
    restart: unless-stopped
    ports:
      - "6080:6080"
    volumes:
      - ./data:/data
      - ./config:/home/subsurface
```

Start the container:

```bash
docker compose up -d
```

Open the application in your browser:

[http://localhost:6080](http://localhost:6080)

The first startup may take a little longer while the desktop environment initializes.

## Persistent storage

Two directories are mounted into the container:

| Container path     | Purpose                                            |
| ------------------ | -------------------------------------------------- |
| `/data`            | Persistent application data and dive-related files |
| `/home/subsurface` | User configuration and application settings        |

Both directories should be persisted outside the container so that your data and configuration survive container updates and recreation.

### Permissions

The application runs as a non-root user:

- UID: `10001`
- GID: `10001`

When using bind mounts, ensure the host directories are writable by this user.

For example:

```bash
sudo chown -R 10001:10001 ./data ./config
```

Dockerfile ownership settings do not automatically change the ownership of host directories mounted into the container.

## Docker image

Images are published to GitHub Container Registry:

```text
ghcr.io/kellenwiltshire/subsurface-docker
```

Available tags:

| Tag      | Description                                     |
| -------- | ----------------------------------------------- |
| `latest` | Latest successful build from the default branch |
| `1.0.0`  | Specific Docker project release                 |
| `1.0`    | Latest patch release within a minor version     |
| `1`      | Latest minor release within a major version     |

Replace `latest` with a specific version tag if you prefer to control upgrades.

## Building locally

Clone this repository:

```bash
git clone https://github.com/kellenwiltshire/subsurface-docker.git
cd subsurface-docker
```

Build the image:

```bash
docker build -t subsurface-docker:latest .
```

Run it:

```bash
docker run -d \
  --name subsurface \
  -p 6080:6080 \
  -v ./data:/data \
  -v ./config:/home/subsurface \
  subsurface-docker:latest
```

### Upstream source

The Docker build retrieves the Subsurface source directly from the official upstream repository:

https://github.com/subsurface/subsurface

The upstream repository and revision can be overridden using build arguments:

```bash
docker build \
  --build-arg SUBSURFACE_REF=master \
  -t subsurface-docker:latest .
```

## Architecture and limitations

This deployment runs the Subsurface desktop application inside a virtual X11 desktop, which is streamed to the browser using noVNC.

Current limitations:

- USB device passthrough is not supported.
- Bluetooth device passthrough is not supported.
- The desktop uses software rendering.
- The image currently targets AMD64/x86-64.

Subsurface features that depend on direct access to hardware connected to the host may therefore be unavailable.

## Security

The browser interface is exposed on port `6080`.

**There is no built-in authentication on the noVNC interface.** Do not expose this port directly to the public internet.

For remote access, use a trusted private network such as Tailscale, or put the service behind an appropriately authenticated reverse proxy.

## Project status

This is an independently maintained Docker packaging project. The application itself is built from the official Subsurface source repository.

- Docker packaging and deployment issues: [Open an issue](https://github.com/kellenwiltshire/subsurface-docker/issues)
- Upstream Subsurface application: [subsurface/subsurface](https://github.com/subsurface/subsurface)

Contributions and suggestions are welcome.

## License

Subsurface is licensed under the GNU General Public License. See the upstream repository for its licensing information.

This repository's Docker-specific files are distributed under the license specified in this repository.
