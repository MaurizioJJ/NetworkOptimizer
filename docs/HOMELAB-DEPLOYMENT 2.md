# Homelab Deployment

Network Optimizer is deployed as a Docker Compose project inside Proxmox LXC
200. Docker workloads must not be run directly on the Proxmox host.

## Location and endpoints

- Proxmox host: `192.168.1.137`
- LXC: `200` (`docker`, `192.168.1.135`)
- Compose directory: `/mnt/docker/network-optimizer`
- Application: `http://192.168.1.135:8042`
- Browser speed test: `http://192.168.1.135:3005`
- Health endpoint: `http://192.168.1.135:8042/api/health`

The application uses the published `ghcr.io/ozark-connect/network-optimizer`
and `ghcr.io/ozark-connect/speedtest` images. Persistent application data,
SSH keys, and logs are stored below the Compose directory. The generated
application password is stored only in the root-readable `.env` file there.

iperf3 server mode and InfluxDB monitoring are not enabled in the initial
deployment. The application timezone is `Europe/Prague`.

## Operations

Run these commands from the Proxmox host:

```bash
pct exec 200 -- bash -lc 'cd /mnt/docker/network-optimizer && docker compose ps'
pct exec 200 -- bash -lc 'cd /mnt/docker/network-optimizer && docker compose logs -f'
pct exec 200 -- curl -fsS http://localhost:8042/api/health
```

Update the published images and recreate the services with:

```bash
pct exec 200 -- bash -lc 'cd /mnt/docker/network-optimizer && docker compose pull && docker compose up -d'
```

Back up `/mnt/docker/network-optimizer/data` and the root-readable `.env`
file. Treat both as secrets: the data directory contains the credential
encryption material, and `.env` contains the application password.

## MariaDB resource limit

The existing `mariadb-ha` container has a 1 GiB memory limit and a 1.25 GiB
combined memory-and-swap limit. The limit was applied to the existing
container with `docker update`; it survives container restarts but must be
reapplied if the container is deleted and recreated.
