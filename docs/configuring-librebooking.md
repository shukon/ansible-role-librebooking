<!--
SPDX-FileCopyrightText: 2026 shukon (https://github.com/shukon)

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

# Protects the /Web/install/ setup wizard. Required on first run.
# Remove or change this after completing the initial setup — see below.
librebooking_install_password: "your-strong-install-password-here"

# Optional: set the timezone
# librebooking_timezone: "Europe/Berlin"

# Optional: enable background cron jobs (for reminder emails, etc.)
# librebooking_cron_enabled: 'true'

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

```
https://booking.example.com/Web/install/
```

You will be prompted for the **installation password** you set above. On the next page:

- Enter the database credentials printed above as the **MySQL User** and **Password**
- **Check "Import sample data"** — this creates the database schema and an initial `admin`/`password` account
- Do **not** check "Create the database" or "Create the database user" — both already exist

Once the installation is complete, **remove or change `librebooking_install_password`** in your
`vars.yml` to prevent unauthorized access to the setup page, and re-run the playbook.

## Upgrading

To upgrade LibreBooking, update `librebooking_version` to a new version and re-run the playbook.
The application will pull the new image and restart.

```yaml
librebooking_version: "4.3.0"
```

After a major version upgrade, you may need to re-visit the `/Web/install/` page to run database
migrations. You will need to set `librebooking_install_password` again to access the wizard.

## Available configuration via environment variables

LibreBooking supports configuration via environment variables using the pattern `LB_<KEY>`.
The full list is documented at: https://librebooking.readthedocs.io/en/stable/BASIC-CONFIGURATION.html

Pass extra variables using `librebooking_environment_variables_additional`.

## Notes

- **Port**: LibreBooking listens on port `8080` internally (not `80`).
