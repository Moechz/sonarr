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
| Open | `http://<NAS-IP>:8021/` (container port 8989) |
| Runs as | non-root, `user: "1000:1000"` |
| Persistent data | `/config` → `/Volume1/DockerAppData/sonarrdocker/config`, `/media` → `/Volume1/DockerAppData/sonarrdocker/media`, `/downloads` → `/Volume1/DockerAppData/sonarrdocker/downloads` |
| Logs | `docker logs sonarrdocker` (stdout/stderr) |

`config` holds the database, settings and logs; `media` is the TV library; `downloads` is where completed downloads land. Back up `config`; the media folders are your own files.

## Privacy

This application runs entirely on your NAS. **The developer does not collect, receive or
have access to any data processed by the application.** There is no telemetry, no analytics
and no account.

| Topic | Detail |
|---|---|
| Data processed | The application's configuration, database, caches and logs, plus any files you create with it. |
| Where it is stored | The application data folder (`/Volume1/DockerAppData/sonarrdocker/`) on your NAS. Nothing is uploaded to the developer. |
| Retention period | Until **you** delete it. Logs are rotated or size-limited by the application; uninstalling keeps the data unless you tick “delete data”. |
| Your rights | **Access / export**: copy the data folder. **Correct**: change the values in the web UI. **Delete**: uninstall with “delete data”, or remove the folder after stopping the container. |
| Deletion channel | Delete the folder yourself, or email **moechz@163.com** (answered within 30 days). |
| Security measures | Container runs as uid 1000 (never root), no `privileged`, no host network, no `cap_add`, no Docker socket, one published port; image tag pinned and refreshed on upstream fixes. |
| Children | Not directed at children. |

**Full privacy policy:** [PRIVACY.md](PRIVACY.md) —
<https://github.com/Moechz/sonarr/blob/main/PRIVACY.md> — and the website version at
<https://www.moechz.com/privacy.html>.

## Third-party services (API disclosure)

The developer operates no server in this path. The application contacts only the services
**you configure**, plus (optionally) an update check and the image registry.

| Service | Endpoint | Data sent | Destination / region |
|---|---|---|---|
| Services you configure in the app | the address you enter | only the data that feature needs | normally your own LAN or the provider you chose |
| Update check (if enabled) | the upstream project's release feed | current version string | GitHub, US |
| Image pull | `registry-1.docker.io` or your registry mirror | anonymous pull | Docker Hub, US / your mirror |

See `PRIVACY.md` §6 for the full table. Disable any integration in the app's settings if you
do not want it.

## Licence and attributions

| Component | Licence | Notes |
|---|---|---|
| Sonarr | GPL-3.0-only | upstream application, used unmodified inside the image |
| Container image `linuxserver/sonarr:4.0.20.3014-ls4` | see the image's own metadata | pulled from Docker Hub at install time |
| This packaging | GPL-3.0 (see `LICENSE`) | compose file, metadata, translations, icon |

“Sonarr”, “TerraMaster” and “TOS” are trademarks of their respective owners. This is an
independent community package by Moechz and is not affiliated with the upstream project or
TerraMaster. Full details: [NOTICE.md](NOTICE.md).

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
| x86_64 | `sonarrdocker_x86_64.tar.gz` | 4820 B | `400b14ed67c84a8b2f646d1f11a594bfdb237983dcd2f9376dcfa762935cab66` |
| aarch64 | `sonarrdocker_aarch64.tar.gz` | 4821 B | `25c14e7f4f15021c868ab185cc1bc07b8c816836ad887013cc4eeae3631a913f` |

The x86_64 package was verified with the full on-device E2E; the aarch64 package
passes the official `TOSAppSelfTestingTool -c` check and the developer platform's
automated validation (no ARM host was available for a container E2E).

## Build

```bash
scripts/build.sh                  # -> out/sonarrdocker.tar.gz + .sha256
scripts/build.sh 4.0.20-3   # packaging iteration bump
scripts/build.sh 4.0.20-2 aarch64
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
2. Release **tag `v4.0.20-2`**, assets `sonarrdocker.tar.gz` +
   `sonarrdocker.tar.gz.sha256` (Docker asset naming: `<app_id>.tar.gz`, **no
   platform suffix**).
3. Developer platform → Add Application: ID `sonarrdocker`, package type Docker,
   repository URL, architecture `x86_64`.
4. Version Management → Add Version `4.0.20-2` → automated validation → review.


## Limitations

No download client and no indexer are included; add them in Settings.
`/media` and `/downloads` belong to this app only. They are not shared with the Deluge, Radarr or Lidarr packages - each TOS Docker app has its own data folder.
The page is plain HTTP on the LAN; use a TLS reverse proxy before exposing it to the internet.
Docker applications cannot be upgraded in place in the App Center: updating means uninstalling and reinstalling, which keeps the data folder.

---

Packaged for TOS 7 by Moechz. Upstream image used unmodified from Docker Hub.

## Support

- Issues and questions: <https://github.com/Moechz/sonarr/issues>
- Email: moechz@163.com
- Privacy: [PRIVACY.md](PRIVACY.md) · Licence: [NOTICE.md](NOTICE.md)
