# Changelog

All notable user-visible changes to the Sonarr TOS application package.

## 4.0.20.3014-1 — 2026-11-08

### Added
- First release. Sonarr (`linuxserver/sonarr:4.0.20.3014-ls4`, Docker Hub official image, unmodified)
  packaged as a TOS 7 Docker application, published on port 8017 and running
  as a non-root user (1000:1000).
- Persistent data is kept under `/Volume1/DockerAppData/sonarrdocker/`.

### Architectures
- Dual-architecture release: `sonarrdocker_x86_64.tar.gz` (4820 B, `d98364f07e1f189464f69290d4c79de1aecfcc517f927ee22d229eee8aa3bf02`) and
  `sonarrdocker_aarch64.tar.gz` (4821 B, `6e5446084f3308a0be1c9de019698384092d0170eb147e7855a7f0365c949f34`), built by `scripts/build-dual.sh`.
  x86_64 has the full on-device E2E; aarch64 passes the official check and the
  platform validation (no ARM host available for a container E2E).
