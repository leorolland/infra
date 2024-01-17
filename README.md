# My infra (WIP)

My personal infrastructure is composed of:
- 💾 **backup** server(s):
  - creates and stores `postgresql` backups (regular `pg_dump`)
  - (soon) syncs a remote server with rsync
- 🌐 **static** server(s):
  - serves my web static content (with http3 and SSL enabled using [Caddy](https://github.com/caddyserver/caddy))

## Pre-requirements
- Linux servers with `python` and `apt`
- your SSH public key in servers authorized keys (`./ssh/authorized-hosts`)

## Configuration

1. Install dependencies
```sh
ansible-galaxy collection install community.general
ansible-galaxy install -r requirements.yml
```

1. Configure `inventory.yml` with backup and static servers

1. Create `vars.yml`

```yml
pgdump:
  user: changeme
  password: changeme
  host: changeme
  database: changeme
```

## Installation
```sh
ansible-playbook -i inventory.yml site.yml --extra-vars "@vars.yml"
```
