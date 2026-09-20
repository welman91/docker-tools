# AGENTS.md

Local dev-tools stack via Docker Compose: MySQL 8, phpMyAdmin, Mailpit. No build, test, or lint tooling. Only Compose file matters.

## Key files

- `docker-compose.yml` — single source of truth. Backup `bkp-docker-compose.yml` is stale copy; don't edit both.
- `config.user.inc.php` — mounted into phpMyAdmin. Only phpMyAdmin config.
- `php.ini` — NOT referenced by compose; unused/stale. Don't rely on it.
- `data/mysql/` — live MySQL data (datadir), bind-mounted. Owned by `dnsmasq`, roughly 600MB+ with binlogs. Never commit.

## Commands

- Up: `docker compose up -d` (from repo root)
- Down (keep data): `docker compose down`
- Wipe + recreate data: delete `data/mysql/*` first, then `docker compose down -v` — irreversible, destroys all databases/binlogs.

## Gotchas

- `dockernet1` network is **external**, subnet `172.20.0.0/16`, static IPs `.2/.3/.4`. Already exists on this host. On fresh host, create first:
  `docker network create dockernet1 --subnet 172.20.0.0/16` or `up` fails.
- `mysql_data` volume is bind mount with absolute host path `device: /home/welman/repositories/docker-tools/data/mysql`. Machine-specific — update for other workspaces/hosts.
- MySQL root password empty (`MYSQL_ALLOW_EMPTY_PASSWORD: yes`); phpMyAdmin connects root/no pass. Dev only.
- Containers currently stopped (verified).

## Ports

| Service      | Port      |
|--------------|-----------|
| MySQL        | 9998 (host) → 3306 |
| phpMyAdmin   | 9999 → 80 |
| Mailpit SMTP | 1025 |
| Mailpit UI   | 8025 |

## Git state

- Branch `master`, one commit "Start". Uncommitted: `docker-compose.yml` (bind-mount change), untracked `data/` + `bkp-docker-compose.yml`.
- `data/` should be gitignored; don't stage it.