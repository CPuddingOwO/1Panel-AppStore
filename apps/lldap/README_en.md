# Light LDAP

Lightweight LDAP authentication server with a built-in web UI, providing unified accounts and SSO for your self-hosted services.

## Features

- Single process and low resource usage, much easier to get started with than OpenLDAP: users, groups and passwords are managed in the web UI, no LDIF files needed
- Standard LDAP protocol, works out of the box with Nextcloud, Gitea, Jellyfin, Authelia and any other LDAP-aware application
- Creates the admin account and data directory automatically on first start, no extra bootstrap step

## Installation

Install **Light LDAP** from the 1Panel App Store and fill in the form:

| Field | Description |
|-------|-------------|
| LDAP Port | Public port for the LDAP protocol, defaults to `3890` |
| Web Port | Public port for the web UI, defaults to `17170` |
| LDAP Base DN | Root of the directory tree, defaults to `dc=example,dc=com`. Use your own domain and **never change it after the first start** |
| Admin Password | Initial password of the built-in `admin` user, written to the database on first start |
| JWT Secret | Session signing secret, keep the randomly generated value |

## First login

1. Open `http://<server-ip>:<web-port>` (default: `http://<server-ip>:17170`)
2. The username is always `admin`, the password is the **Admin Password** from the form
3. Create users and groups in the web UI, then connect other applications with the settings below

## Connecting other applications

| Setting | Value |
|---------|-------|
| Server address | `<server-ip>` (applications on the same 1Panel network can use the container name `lldap`) |
| Port | LDAP port, defaults to `3890` |
| Base DN | The Base DN from the form, e.g. `dc=example,dc=com` |
| Bind DN | `uid=admin,ou=people,<Base DN>` |
| Bind password | Admin password |
| User filter | `(uid=%s)`, some applications require `(&(uid=%s)(objectClass=person))` |
| User DN template | `uid={username},ou=people,<Base DN>` |
| Group DN template | `cn={group},ou=groups,<Base DN>` |

> Users live under `ou=people` and groups under `ou=groups` — this structure is fixed in Light LDAP.

## Persistent data

Application data is persisted in `./data` (mounted to `/data` inside the container): users, groups and passwords in an SQLite database plus the session keys. **Back up this directory before upgrading or migrating** — the Base DN and admin password live there once initialized.

## Environment variables

| Variable | Description | Source |
|----------|-------------|--------|
| `PANEL_APP_PORT_LDAP` | Public LDAP port, mapped to container `3890` | Install form |
| `PANEL_APP_PORT_HTTP` | Public web UI port, mapped to container `17170` | Install form |
| `LLDAP_LDAP_BASE_DN` | Root DN of the directory tree | Install form |
| `LLDAP_LDAP_USER_PASS` | Password of the built-in admin user | Install form |
| `LLDAP_JWT_SECRET` | Session signing secret | Install form |
| `TZ` | Container timezone, defaults to `Asia/Shanghai` | compose default |

Upstream supports more variables that are not exposed in the form (mail notifications, external database, key seed, ...). Add them to the compose `environment` section if you need them.

## Notes

- **Do not change the Base DN after initialization**: it would invalidate the user DNs configured in every connected application.
- LDAP is a plaintext protocol. Decide for yourself whether you need TLS before exposing it publicly; the web UI should be published through a 1Panel reverse proxy with HTTPS.
- When changing a port, update the form value and any firewall or reverse proxy rule together.

## Official resources

- Website / GitHub: https://github.com/lldap/lldap
- Installation and configuration docs: https://github.com/lldap/lldap/blob/main/docs/install.md
