# Kimai (Kimai2) Ansible Role (Ansible-NAS style)

Deploys the official Kimai Docker image behind Traefik with a MariaDB database, no host ports,
and idempotent first-run bootstrap (migrations + admin user).

## Usage

1. Copy the `roles/kimai` directory into your repo.
2. Set variables in your inventory/host_vars, e.g.:

```yaml
kimai_enabled: true
ansible_nas_domain: example.com
kimai_hostname: kimai
kimai_db_password: "supersecretdb"
kimai_app_secret: "use_a_long_random_string"
kimai_admin_email: "you@example.com"
kimai_admin_password: "changeme"
```

3. Include the role in your play:

```yaml
- hosts: ansible_nas_host
  roles:
    - role: kimai
```

Kimai will be available at `https://{{ kimai_hostname }}.{{ ansible_nas_domain }}`.
The role never publishes host ports; Traefik handles ingress via labels on the `traefik` network.

## Variables (key ones)

- `kimai_hostname` / `ansible_nas_domain` → FQDN used by Traefik.
- `kimai_available_externally` (bool) → if true, adds Homepage labels and expects access via the FQDN.
- `kimai_image` (default: `ghcr.io/kimai/kimai2:latest`).
- `kimai_db_image` (default: `mariadb:10.11`).
- `kimai_db_password`, `kimai_app_secret` → **override in inventory**.
- `kimai_extra_env` / `kimai_labels_extra` → advanced overrides.

## First-run

The role waits for DB, starts the app, runs `kimai:install -n`, then ensures an admin user
(create only if missing). All steps are safe to re-run (idempotent).

## Data & Backups

Persistent data:
- `{{ docker_home }}/kimai` → Kimai `var/`
- `{{ docker_home }}/kimai-db` → MariaDB datadir

See `scripts/kimai-backup.sh` for a restic-driven backup helper that performs a mysqldump before pushing. 
Adapt retention and repository to your environment.
