# sonarr-for-tos

> TerraMaster TOS 7 App Center — **Docker application** package for
> [Sonarr](https://github.com/linuxserver/docker-sonarr).

## What this is

A four-file TOS Docker application archive (`sonarrdocker.tar.gz`) that installs
Sonarr on TOS 7 through the App Center:

- `config.ini` — TOS application metadata (`application_type: docker`, id `sonarrdocker`)
- `sonarrdocker.lang` — 23-language superset (TOS requires 14)
- `sonarrdocker.svg` — application icon (2226 bytes, 2 SVG elements)
- `docker-compose.yml` — one service, image `linuxserver/sonarr:4.0.20.3014-ls4`

## How it works

| | |
|---|---|
| Container image | `linuxserver/sonarr:4.0.20.3014-ls4` (Docker Hub, used unmodified) |
| Open | `http://<NAS-IP>:8017/` (container port 8989) |
| Runs as | non-root, `user: "1000:1000"` |
| Persistent data | `/config` → `/Volume1/DockerAppData/sonarrdocker/config`, `/media` → `/Volume1/DockerAppData/sonarrdocker/media`, `/downloads` → `/Volume1/DockerAppData/sonarrdocker/downloads` |
| Logs | `docker logs sonarrdocker` (stdout/stderr) |

`config` holds the database, settings and logs; `media` is the TV library; `downloads` is where completed downloads land. Back up `config`; the media folders are your own files.

## Design highlights

- **Non-root.** `user: "1000:1000"`; verified on real hardware (the container reports uid 1000).
- **Fixed image tag, never `:latest`**, Docker Hub only, no `privileged`, no `network_mode: host`, and no Docker socket, no `cap_add`.
- **Health check** probing the service on container port 8989.
- **Deterministic build**: GNU tar with uid/gid 0 and zeroed timestamps, gzip without a timestamp, plus the review-standards self-check above.
- **Runs as a non-root container.** Verified on real hardware as uid 1000.
- **Three volumes.** `/config` keeps the database and settings; `/media` and `/downloads` give the app a library and a download folder inside its own application data directory.
- **Health check** probes the web UI on container port 8989.
- **Fixed image tag, never `:latest`**, Docker Hub only, no `privileged`, no `network_mode: host`, no Docker socket, no `cap_add`.

## Runtime file manifest

Everything the application writes on the NAS lives inside its own data
directory; the container-internal `/tmp` is discarded with the container. The
table lists what is created and how it behaves. Nothing else is written.

| Path | Purpose | Created when | Growth bound | Lifecycle |
|---|---|---|---|---|
| `config/sonarr.db` | series, quality profiles, indexers and download-client settings (SQLite) | first start | grows with the library | never overwritten; back it up |
| `/Volume1/DockerAppData/sonarrdocker/config` | Sonarr application data (everything the container writes outside /tmp) | on first start / on demand | bounded by your own data | persistent; kept on uninstall unless you ask to delete it |
| `/Volume1/DockerAppData/sonarrdocker/media` | Sonarr application data (everything the container writes outside /tmp) | on first start / on demand | bounded by your own data | persistent; kept on uninstall unless you ask to delete it |
| `/Volume1/DockerAppData/sonarrdocker/downloads` | Sonarr application data (everything the container writes outside /tmp) | on first start / on demand | bounded by your own data | persistent; kept on uninstall unless you ask to delete it |

## Architectures

The TOS store takes one package per architecture. This release provides both,
built from the same sources by `scripts/build-dual.sh`:

| Architecture | Package | Size | SHA-256 |
|---|---|---|---|
| x86_64 | `sonarrdocker_x86_64.tar.gz` | 4820 B | `d98364f07e1f189464f69290d4c79de1aecfcc517f927ee22d229eee8aa3bf02` |
| aarch64 | `sonarrdocker_aarch64.tar.gz` | 4821 B | `6e5446084f3308a0be1c9de019698384092d0170eb147e7855a7f0365c949f34` |

The x86_64 package was verified with the full on-device E2E; the aarch64 package
passes the official `TOSAppSelfTestingTool -c` check and the developer platform's
automated validation (no ARM host was available for a container E2E).

## Build

```bash
scripts/build.sh                  # -> out/sonarrdocker.tar.gz + .sha256
scripts/build.sh 4.0.20-2   # packaging iteration bump
scripts/build.sh 4.0.20-1 aarch64
```

The build stages the four files, substitutes the version/platform placeholders,
removes CRLF/BOM/AppleDouble, then runs
`tools/verify.py`, a review-standards self-check (archive layout, JSON validity,
required fields, 14+ language coverage, version consistency, Docker-Hub-only
images, fixed tag, reserved/recommended host ports, non-root user, per-service
healthcheck/TZ/restart, data path below the application data root, never the
`/Volume*` wildcard, comment-free sequence blocks, `x-app-meta` position, no
literal secrets and the SVG size/element limits).

## Local verification

```bash
./TOSAppSelfTestingTool -c out/sonarrdocker.tar.gz   # official spec check
bash test/install-and-verify.sh                  # on-device E2E (needs root)
```

> Docker applications cannot be side-loaded through the App Center's manual
> install page (it accepts `.deb` only); the official self-testing tool's `-i`
> mode or `docker compose` are the only ways to install one for testing.

## Store submission checklist

1. Public GitHub repo (code + README only).
2. Release **tag `v4.0.20-1`**, assets `sonarrdocker.tar.gz` +
   `sonarrdocker.tar.gz.sha256` (Docker asset naming: `<app_id>.tar.gz`, **no
   platform suffix**).
3. Developer platform → Add Application: ID `sonarrdocker`, package type Docker,
   repository URL, architecture `x86_64`.
4. Version Management → Add Version `4.0.20-1` → automated validation → review.

## License / branding

Sonarr is distributed by its upstream authors under **GPL-3.0-only**. The container image is the official image from Docker Hub (`linuxserver/sonarr:4.0.20.3014-ls4`) and is used unmodified; no upstream source code is redistributed here. This packaging is an independent community submission by Moechz and is not affiliated with the upstream project or TerraMaster.

## Limitations

No download client and no indexer are included; add them in Settings.
`/media` and `/downloads` belong to this app only. They are not shared with the Deluge, Radarr or Lidarr packages - each TOS Docker app has its own data folder.
The page is plain HTTP on the LAN; use a TLS reverse proxy before exposing it to the internet.
Docker applications cannot be upgraded in place in the App Center: updating means uninstalling and reinstalling, which keeps the data folder.

---

Packaged for TOS 7 by Moechz. Upstream image used unmodified from Docker Hub.
