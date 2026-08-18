# Homelab Deployment

Network Optimizer is deployed as a Docker Compose project inside Proxmox LXC
200. Docker workloads must not be run directly on the Proxmox host.

## Location and endpoints

- Proxmox host: `192.168.1.137`
- LXC: `200` (`docker`, `192.168.1.135`)
- LXC Tailscale address: `100.120.205.82`
- Compose directory: `/mnt/docker/network-optimizer`
- Application: `http://192.168.1.135:8042`
- Application over Tailscale: `http://100.120.205.82:8042`
- Browser speed test: `http://192.168.1.135:3005`
- Health endpoint: `http://192.168.1.135:8042/api/health`

The application follows the upstream Docker preview channel using
`ghcr.io/ozark-connect/network-optimizer:preview` and
`ghcr.io/ozark-connect/speedtest:preview`. The preview tag receives preview and
stable releases. Persistent application data, SSH keys, and logs are stored
below the Compose directory. The generated application password is stored only
in the root-readable `.env` file there.

iperf3 server mode is disabled. InfluxDB monitoring is enabled against the
separate `influxdb` container on the same LXC. The application timezone is
`Europe/Prague`.

## Operations

Run these commands from the Proxmox host:

```bash
pct exec 200 -- bash -lc 'cd /mnt/docker/network-optimizer && docker compose ps'
pct exec 200 -- bash -lc 'cd /mnt/docker/network-optimizer && docker compose logs -f'
pct exec 200 -- curl -fsS http://localhost:8042/api/health
```

The same checks can be run directly over Tailscale:

```bash
ssh root@100.120.205.82 'cd /mnt/docker/network-optimizer && docker compose ps'
curl -fsS http://100.120.205.82:8042/api/health
```

Update the published images and recreate the services with:

```bash
pct exec 200 -- bash -lc 'cd /mnt/docker/network-optimizer && docker compose pull && docker compose up -d'
```

Do not replace the `:preview` tags with the upstream production Compose file's
`:latest` tags during routine updates. Returning to stable requires an explicit
channel change of both image tags followed by a pull and recreate.

Before an update, stop the application briefly and archive `data`, `logs`,
`ssh-keys`, `.env`, and `docker-compose.yml` so the SQLite database and its WAL
are captured consistently. Store the archive outside the Compose directory in
`/mnt/docker/network-optimizer-backups`. After the update, verify both service
containers, `/api/health`, the application login redirect, the speed-test HTTP
endpoint, container restart/OOM counts, and recent logs.

Rollback is to restore the archived persistent files and Compose file, retag
the previous image digest recorded by `docker image inspect`, then run
`docker compose up -d` and repeat the same checks.

Back up `/mnt/docker/network-optimizer/data` and the root-readable `.env`
file. Treat both as secrets: the data directory contains the credential
encryption material, and `.env` contains the application password.

## MariaDB resource limit

The existing `mariadb-ha` container has a 1 GiB memory limit and a 1.25 GiB
combined memory-and-swap limit. The limit was applied to the existing
container with `docker update`; it survives container restarts but must be
reapplied if the container is deleted and recreated.
