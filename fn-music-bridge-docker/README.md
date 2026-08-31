# fn-music-bridge — Docker edition

A minimal Docker packaging of [qianlipp/fn-music-bridge](https://github.com/qianlipp/fn-music-bridge), pinned to upstream commit `ad9ce203cbb7fdb54f9404a7abb2df605893f388`.

This packaging keeps the core two-layer architecture and protocol functionality while removing the fnOS FPK packaging layer:

- Yinqiao API bridge -> official FeiNiu Music Unix socket
- OpenSubsonic / Subsonic
- Jellyfin Audio
- Synology Audio Station compatible API
- Ampache API 6
- built-in management UI
- encrypted account credentials and settings persisted under `./data`

## Important requirement

This container still needs the official FeiNiu Music backend. On the Docker host, this socket must exist:

```bash
ls -l /var/run/trim_music.socket
```

The Compose file bind-mounts it read/write at the same path inside the API container. If your fnOS installation exposes the socket somewhere else, change the left side of that volume mapping.

## Start

```bash
cd fn-music-bridge-docker
cp .env.example .env
docker compose up -d --build
```

Then open:

```text
http://NAS_IP:14040/yinqiao/ui/
```

For compatible music clients, use:

```text
http://NAS_IP:14040
```

Do not append `/rest` for Subsonic clients.

## Persistent data

The following files are written to `./data` and survive container recreation:

- `yinqiao-credentials.enc`
- `yinqiao-credentials.key`
- `yinqiao-settings.json`
- protocol session data

Back up the whole `data` directory together. In particular, do not lose the credential key file if you want to keep using the encrypted credentials.

## Architecture

```text
Music client
    |
    v
service :14040
    |
    | HTTP /yinqiao/v1
    v
api :18090 (internal Docker network only)
    |
    | Unix socket bind mount
    v
/var/run/trim_music.socket
    |
    v
Official FeiNiu Music service
```

Only port 14040 is published by default. The lower-level API is intentionally kept on the private Compose network.

## Updating upstream

Change `UPSTREAM_REF` in `.env` to a reviewed upstream commit SHA and rebuild:

```bash
docker compose build --no-cache
docker compose up -d
```

Using a commit SHA rather than `main` makes builds reproducible and prevents an upstream change from silently entering a deployment.

## Notes

- The original project targets fnOS and requires the official FeiNiu Music service; Docker does not remove that dependency.
- Keep port 14040 on a trusted LAN, VPN, or authenticated HTTPS reverse proxy. Do not expose it directly to the public Internet.
- The Docker build runs the upstream Go test suites before producing the binaries.
- Source license remains GNU AGPL-3.0; see the upstream repository for license and third-party notices.
