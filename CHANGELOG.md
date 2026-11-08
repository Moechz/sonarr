# Changelog

All notable user-visible changes to the Sonarr TOS application package.

## 4.0.20-2 — 2026-11-08

### Added
- First release. Sonarr (`linuxserver/sonarr:4.0.20.3014-ls4`, Docker Hub official image, unmodified)
  packaged as a TOS 7 Docker application, published on port 8021 and running
  as a non-root user (1000:1000).
- Persistent data is kept under `/Volume1/DockerAppData/sonarrdocker/`.

### Architectures
- Dual-architecture release: `sonarrdocker_x86_64.tar.gz` (4820 B, `400b14ed67c84a8b2f646d1f11a594bfdb237983dcd2f9376dcfa762935cab66`) and
  `sonarrdocker_aarch64.tar.gz` (4821 B, `25c14e7f4f15021c868ab185cc1bc07b8c816836ad887013cc4eeae3631a913f`), built by `scripts/build-dual.sh`.
  x86_64 has the full on-device E2E; aarch64 passes the official check and the
  platform validation (no ARM host available for a container E2E).
