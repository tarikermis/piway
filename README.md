<h1>
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="docs/brand/piway-logo-white.svg">
    <img src="docs/brand/piway-logo-black.svg" alt="piway" width="240">
  </picture>
</h1>

Self-hosted metrics agent and mini PaaS for Raspberry Pi, written in Go.

`git push` → build on the Pi → run in a container → served on a subdomain, with a live dashboard and an MCP server so Claude Code can operate it.

## Quick start

Requires Go (see `go.mod`) and Node.js 24 for the dashboard.

```sh
make build                            # builds web/, then a binary that embeds it
./bin/piway -addr :8080
curl localhost:8080/healthz
# dashboard: open http://<pi-address>:8080/ in a browser
curl localhost:8080/api/v1/snapshot   # see docs/API.md
go run ./cmd/wsload -clients 100      # WebSocket load test
```

`make build-go` builds the binary without the dashboard (API only, no Node
needed).

### Dashboard development

```sh
./bin/piway &                         # API on :8080
cd web && npm run dev                 # UI with hot reload on :5173, proxies the API
```

See [web/README.md](web/README.md).

## Install on a Raspberry Pi

On 64-bit Raspberry Pi OS:

```sh
curl -fsSL https://raw.githubusercontent.com/OFK0/piway/main/install.sh | sh
```

It installs the latest release (checksum verified) and runs `piway setup`,
which installs the service, rootless Docker, Caddy and mDNS, showing every
`sudo` command first. See [docs/INSTALL.md](docs/INSTALL.md) for the
options, uninstalling, and the manual steps.

## Run as a user service

```sh
install -m755 bin/piway ~/.local/bin/piway
mkdir -p ~/.config/systemd/user
cp deploy/systemd/piway.service ~/.config/systemd/user/
systemctl --user daemon-reload && systemctl --user enable --now piway
```

To change settings, copy `deploy/config.example.yaml` to
`~/.config/piway/config.yaml`, edit it, and restart the service. See
[docs/CONFIG.md](docs/CONFIG.md) for the full list, the `PIWAY_*` environment
variables, and the precedence rules.

## Check an installation

```sh
piway doctor                                # service, Docker, Caddy, disk, clock…
sudo ~/.local/bin/piway doctor --security   # the steps in docs/HARDENING.md
```

Each problem comes with the command that fixes it. `-json` prints the
results for scripts; the exit status is 1 when a check fails.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for the issue, branch, commit
(Conventional Commits) and pull request conventions.

## Roadmap

See [docs/ROADMAP.md](docs/ROADMAP.md) and the GitHub milestones.

## License

piway is released under the [MIT license](LICENSE). The binary also contains
third-party code and fonts under their own licenses; `make build` collects
them into `THIRD_PARTY_NOTICES.txt`, which ships with every release and is
served by the dashboard (footer › Licenses).
