# My infra (WIP)

My personal infrastructure is composed of:
- 🌐 **static** server(s):
  - serves my web static content (with http3 and SSL enabled using [Caddy](https://github.com/caddyserver/caddy))
  - runs the latest version of [sortir.in](https://github.com/leorolland/sortir.in) API service
    - automatically re-populates the DB every 12 hours

## Pre-requirements
- Linux servers with `python` and `apt`
- your SSH public key in the servers' authorized keys

## Configuration

1. Install dependencies
```sh
ansible-galaxy install -r requirements.yml
```

2. Configure `inventory.yml` with the static server(s)

3. (Optional) Override the sortir.in service defaults from `roles/sortir/defaults/main.yml` in `site.yml`:
```yml
- role: sortir
  sortir_populate_cron_hour: "*/12"          # Run populate job every 12 hours
  sortir_populate_cron_minute: "0"           # At minute 0
  sortir_populate_locations_quantity: "15"   # Get and populate DB with events of top 15 France locations
```
The target host architecture can also be overridden per-host in `inventory.yml`:
```yml
static:
  hosts:
    sortir_in:
      ansible_host: sortir.in
      ansible_user: debian
      sortir_arch: "amd64"                   # Binary architecture of the target host
```

## Installation
```sh
ansible-playbook -i inventory.yml site.yml
```
