<!--
SPDX-FileCopyrightText: 2026 shukon

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Configuring LibreBooking

This document describes how to configure LibreBooking using this Ansible role.

## Prerequisites

This service requires:

- A **MariaDB** database. LibreBooking does **not** support PostgreSQL.
  You can use an external MariaDB instance or the `mariadb` role of the MASH playbook.
- A reverse proxy (Traefik is integrated by default in MASH).

## Example configuration (`vars.yml`)

```yaml
########################################################################
#                                                                      #
# librebooking                                                         #
#                                                                      #
########################################################################

librebooking_enabled: true

librebooking_hostname: booking.example.com

# Protects the /Web/install/ setup wizard. Use a strong password.
librebooking_install_password: "your-strong-install-password-here"

# Optional: set the timezone
# librebooking_timezone: "Europe/Berlin"

# Optional: enable background cron jobs (for reminder emails, etc.)
# librebooking_cron_enabled: 'true'

# Optional: allow users to self-register accounts (disabled by default).
# Enable temporarily if you need to register your admin account manually.
# librebooking_self_registration_enabled: 'true'

# Optional: pass extra LB_ environment variables to configure the application.
# See: https://librebooking.readthedocs.io/en/stable/BASIC-CONFIGURATION.html
# librebooking_environment_variables_additional: |
#   LB_APP_TITLE='My Booking System'

########################################################################
#                                                                      #
# /librebooking                                                        #
#                                                                      #
########################################################################
```

## First-time setup

LibreBooking does **not** initialize its database schema automatically. You must run the
web-based install wizard on first install.

After running the playbook for the first time, retrieve your database credentials:

```bash
just run-tags print-librebooking-db-credentials
```

Then navigate to the install wizard:

```text
https://booking.example.com/Web/install/
```

You will be prompted for the **installation password** you set above. On the next page:

- Enter the database credentials printed above as the **MySQL User** and **Password**
- **Check "Import sample data"** — this creates the database schema and an initial `admin`/`password` account
- Do **not** check "Create the database" or "Create the database user" — both already exist

Once the installation is complete, keep `librebooking_install_password` set to a strong value to prevent unauthorized access to the setup page.

## Upgrading

To upgrade LibreBooking, update `librebooking_version` to a new version and re-run the playbook.
The application will pull the new image and restart.

```yaml
librebooking_version: "4.3.0"
```

After a major version upgrade, you may need to run database migrations. Navigate to:

```text
https://booking.example.com/Web/install/configure.php
```

If you need your database credentials again:

```bash
just run-tags print-librebooking-db-credentials
```

## Available configuration via environment variables

LibreBooking supports overriding any config key via environment variables. The naming convention is
`LB_` + the config key uppercased, with dots and dashes replaced by underscores. For example:

| Config key | Environment variable |
| --- | --- |
| `app.title` | `LB_APP_TITLE` |
| `phpmailer.smtp.host` | `LB_PHPMAILER_SMTP_HOST` |
| `authentication.oauth2.client.id` | `LB_AUTHENTICATION_OAUTH2_CLIENT_ID` |

See the [upstream docs](https://librebooking.readthedocs.io/en/latest/ADVANCED-CONFIGURATION.html#environment-variable-override)
for the full reference.

Pass extra variables using `librebooking_environment_variables_additional`.

## OAuth2 / SSO

LibreBooking supports OAuth2 authentication with any compliant IdP (e.g. Authentik, Keycloak).
See the [upstream OAuth2 docs](https://librebooking.readthedocs.io/en/stable/Oauth2-Configuration.html)
for the full list of settings.

**IdP setup:** create a confidential client and configure the redirect URI to:

```text
https://booking.example.com/Web/oauth2-auth.php
```

Pass settings via `librebooking_environment_variables_additional`:

```yaml
librebooking_environment_variables_additional: |
  LB_AUTHENTICATION_OAUTH2_LOGIN_ENABLED=true
  LB_AUTHENTICATION_OAUTH2_NAME=authentik
  LB_AUTHENTICATION_OAUTH2_URL_AUTHORIZE=https://auth.example.com/application/o/authorize/
  LB_AUTHENTICATION_OAUTH2_URL_TOKEN=https://auth.example.com/application/o/token/
  LB_AUTHENTICATION_OAUTH2_URL_USERINFO=https://auth.example.com/application/o/userinfo/
  LB_AUTHENTICATION_OAUTH2_CLIENT_ID=your-client-id
  LB_AUTHENTICATION_OAUTH2_CLIENT_SECRET=your-client-secret
  LB_AUTHENTICATION_OAUTH2_CLIENT_URI=/Web/oauth2-auth.php
```

To redirect straight to your IdP without showing the built-in login form:

```yaml
  LB_AUTHENTICATION_HIDE_LOGIN_PROMPT=true
```

Some IdPs require a trailing slash on the authorize URL — by default LibreBooking strips it.
To preserve it:

```yaml
  LB_AUTHENTICATION_OAUTH2_STRIP_TRAILING_SLASH=false
```

## Notes

- **Port**: LibreBooking listens on port `8080` internally (not `80`).
