# ns8-chiefonboarding

A [NethServer 8](https://github.com/NethServer/ns8-core) module for ChiefOnboarding - employee onboarding and management platform.

## Table of Contents

- [Install](#install)
- [Configure](#configure)
- [Get Configuration](#get-the-configuration)
- [Uninstall](#uninstall)
- [Update](#update)
- [Smarthost Setting Discovery](#smarthost-setting-discovery)
- [Debug](#debug)
- [Testing](#testing)
- [UI Translation](#ui-translation)

---

## Install

Instantiate the module with:

```bash
add-module ghcr.io/geniusdynamics/chiefonboarding:latest 1
```

The output of the command will return the instance name.

**Output example:**

```json
{"module_id": "chiefonboarding1", "image_name": "chiefonboarding", "image_url": "ghcr.io/geniusdynamics/chiefonboarding:latest"}
```

---

## Configure

Launch `configure-module` by setting the required and optional parameters.

Assume the instance is named `chiefonboarding1`.

### Core Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `host` | string | Yes | Fully qualified domain name for the application |
| `http2https` | boolean | Yes | Enable HTTP to HTTPS redirection (`true`/`false`) |
| `lets_encrypt` | boolean | Yes | Enable Let's Encrypt certificate (`true`/`false`) |

### Email Settings

Email configuration is automatically discovered from the system smarthost settings. The following SMTP settings are configured automatically:

- SMTP host, port, username, and password
- SMTP encryption method (TLS/SSL)
- TLS verification settings

### Google SSO (Optional)

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `allow_google_sso` | boolean | `false` | Enable Google SSO authentication |
| `google_sso_client_id` | string | `""` | Google OAuth client ID |
| `google_sso_secret` | string | `""` | Google OAuth client secret |

### SSO Auto-Creation (Optional)

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `sso_auto_create_user` | boolean | `true` | Automatically create users from SSO authentication |

### OIDC Role Patterns (Optional)

Configure role mapping patterns for OpenID Connect authentication:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `oidc_role_new_hire_pattern` | string | `""` | Regex pattern to match new hire roles |
| `oidc_role_admin_pattern` | string | `""` | Regex pattern to match admin roles |
| `oidc_role_manager_pattern` | string | `""` | Regex pattern to match manager roles |
| `oidc_role_path_in_return` | string | `"groups"` | JSON path in OIDC response for roles |

### Authentication Providers (Optional)

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `socialaccount_providers` | object | `{}` | JSON object defining social authentication providers |
| `allauth_providers` | string | `""` | JSON configuration for additional authentication providers |

### Example Configuration

```bash
api-cli run configure-module --agent module/chiefonboarding1 --data - <<EOF
{
  "host": "chiefonboarding.domain.com",
  "http2https": true,
  "lets_encrypt": false,
  "allow_google_sso": true,
  "google_sso_client_id": "your-client-id",
  "google_sso_secret": "your-client-secret",
  "sso_auto_create_user": true,
  "oidc_role_new_hire_pattern": ".*new_hire.*",
  "oidc_role_admin_pattern": ".*admin.*",
  "oidc_role_manager_pattern": ".*manager.*",
  "oidc_role_path_in_return": "groups",
  "socialaccount_providers": {},
  "allauth_providers": ""
}
EOF
```

**The above command will:**

- Configure the chiefonboarding instance
- Configure a virtual host for traefik to access the instance

---

## Get the Configuration

Retrieve the configuration with:

```bash
api-cli run get-configuration --agent module/chiefonboarding1
```

---

## Uninstall

To uninstall the instance:

```bash
remove-module --no-preserve chiefonboarding1
```

---

## Update

To update the instance:

```bash
api-cli run update-module --data '{"module_url":"ghcr.io/geniusdynamics/chiefonboarding:latest","instances":["chiefonboarding1"],"force":true}'
```

---

## Smarthost Setting Discovery

Some configuration settings, like the smarthost setup, are not part of the `configure-module` action input: they are discovered by looking at some Redis keys. To ensure the module is always up-to-date with the centralized [smarthost setup](https://geniusdynamics.github.io/ns8-core/core/smarthost/) every time chiefonboarding starts, the command `bin/discover-smarthost` runs and refreshes the `state/smarthost.env` file with fresh values from Redis.

Furthermore if smarthost setup is changed when chiefonboarding is already running, the event handler `events/smarthost-changed/10reload_services` restarts the main module service.

See also the `systemd/user/chiefonboarding.service` file.

> This setting discovery is just an example to understand how the module is expected to work: it can be rewritten or discarded completely.

---

## Debug

### Check Environment Variables

The module runs under an agent that initiates a lot of environment variables (in `/home/chiefonboarding1/.config/state`). Verify them on the root terminal:

```bash
runagent -m chiefonboarding1 env
```

### Use runagent

Become runagent for testing scripts and initiate all environment variables:

```bash
runagent -m chiefonboarding1
```

The path becomes:

```
echo $PATH
/home/chiefonboarding1/.config/bin:/usr/local/agent/pyenv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/
```

### List Running Containers

```bash
runagent -m chiefonboarding1
podman ps
```

**Output example:**

```
CONTAINER ID  IMAGE                                      COMMAND               CREATED        STATUS        PORTS                    NAMES
d292c6ff28e9  localhost/podman-pause:4.6.1-1702418000                          9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  80b8de25945f-infra
d8df02bf6f4a  docker.io/library/postgres:15.5-alpine3.19          --character-set-s...  9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  postgresql-app
9e58e5bd676f  docker.io/library/nginx:stable-alpine3.17  nginx -g daemon o...  9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  chiefonboarding-app
```

### Check Container Environment Variables

```bash
podman exec chiefonboarding-app env
```

**Output example:**

```
TERM=xterm
container=podman
NGINX_VERSION=1.24.0
PKG_RELEASE=1
NJS_VERSION=0.7.12
NGINX_IMAGE=docker.io/nginx:stable-alpine3.17
CONFIG_DATABASE_URI="postgresql://postgres:Nethesis,1234@127.0.0.1:5432/toto"
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOME=/root
```

### Access Container Shell

Run a shell inside the container:

```bash
podman exec -ti chiefonboarding-app sh
/ #
```

---

## Testing

Test the module using the `test-module.sh` script:

```bash
./test-module.sh <NODE_ADDR> ghcr.io/geniusdynamics/chiefonboarding:latest
```

The tests are made using [Robot Framework](https://robotframework.org/).

---

## UI Translation

Translated with [Weblate](https://hosted.weblate.org/projects/ns8/).

To set up the translation process:

1. Add [GitHub Weblate app](https://docs.weblate.org/en/latest/admin/continuous.html#github-setup) to your repository
2. Add your repository to [hosted.weblate.org](https://hosted.weblate.org) or ask a NethServer developer to add it to ns8 Weblate project
