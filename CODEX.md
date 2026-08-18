# Homelab Environment

## Infrastructure

Primary workstation:
- macOS
- VS Code
- GitHub

Virtualization:
- Proxmox VE

Docker:
- Runs inside LXC 200
- Never deploy Docker directly on the Proxmox host

Primary services:
- Portainer
- Network Optimizer
- InfluxDB
- MariaDB
- Other Docker services

Timezone:
- Europe/Prague

## Deployment Rules

Before deploying:

1. Inspect existing containers.
2. Inspect networks.
3. Inspect volumes.
4. Detect port conflicts.
5. Reuse existing conventions.

Always prefer:

- Docker Compose
- Named volumes
- restart: unless-stopped
- Healthchecks
- Portainer-compatible stacks

Never:

- Delete existing containers without explicit approval
- Overwrite existing configuration
- Expose unnecessary ports
- Deploy Docker workloads directly on the Proxmox host

## SSH

SSH access to the Docker LXC is passwordless over Tailscale.

Connect using:

```bash
ssh root@100.120.205.82
```

Docker runs inside LXC 200 (`docker`, LAN address `192.168.1.135`). The
Proxmox host is `root@192.168.1.137`; do not assume a local `proxmox` SSH alias
exists.

From the Proxmox host, enter the LXC using:

```bash
pct enter 200
```

## Expectations

When asked to deploy software:

1. Read repository documentation.
2. Inspect the repository.
3. Inspect the target Docker environment.
4. Identify port, volume, network, and container-name conflicts.
5. Explain the deployment plan.
6. Wait for approval before risky or destructive changes.
7. Deploy the application.
8. Verify containers, logs, healthchecks, ports, and application access.
9. Document the final configuration and maintenance procedure.

Use published container images when available and verified.

Prefer idempotent deployments that can be updated with:

```bash
docker compose pull
docker compose up -d
```
