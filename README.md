# My infra (WIP)

My personal infrastructure is composed of:
- 💾 **backup** server(s):
  - creates and stores `postgresql` backups (regular `pg_dump`)
  - (soon) syncs a remote server with rsync
- 🌐 **static** server(s):
  - serves my web static content (with http3 and SSL enabled using [Caddy](https://github.com/caddyserver/caddy))
  - runs the latest version of [sortir.in](https://github.com/leorolland/sortir.in) API service
    - automatically re-populates the DB every 12 hours

## Pre-requirements
- Linux servers with `python` and `apt`
- your SSH public key in servers authorized keys (`./ssh/authorized-hosts`)

## Configuration

1. Install dependencies
```sh
ansible-galaxy collection install community.general
ansible-galaxy install -r requirements.yml
```

2. Configure `inventory.yml` with backup and static servers

3. Create `vars.yml`

```yml
pgdump:
  user: changeme
  password: changeme
  host: changeme
  database: changeme
```

4. Configure sortir.in service parameters in `site.yml` (optional)

The sortir.in service is configured with the following default parameters:
```yml
- role: sortir
  sortir_populate_cron_hour: "*/12"     # Run populate job every 12 hours
  sortir_populate_cron_minute: "0"      # At minute 0
  sortir_populate_locations_quantity: "15"  # Get and populate DB with events of top 15 France locations
```

## Installation
```sh
ansible-playbook -i inventory.yml site.yml --extra-vars "@vars.yml"
```
