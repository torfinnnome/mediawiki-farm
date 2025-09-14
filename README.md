# MediaWiki Farm with Podman and Ansible

This repository contains an Ansible playbook to deploy a MediaWiki farm using Podman.

## Features

*   Automated deployment of a MediaWiki farm.
*   Uses rootless Podman and Podman Compose for containerization.
*   Uses Caddy as a reverse proxy.
*   Supports multiple wikis in a farm configuration.
*   Each wiki has its own MariaDB/MySQL database (configurable).

## Requirements

*   Ansible 2.9 or later.
*   A Rocky Linux 9 host (or other RHEL-based distribution).
*   SSH access to the host.

## How to use

1.  Clone this repository.
2.  Update the `ansible/inventory` file with your host information.
3.  Create and edit the vault file by running `ansible-vault edit ansible/vault.yml` and add the following content:

    ```yaml
    db_root_password: "verysecret"
    db_host: "host.containers.internal"
    db_user: "mediawiki"
    db_password: "secret"
    mw_default_admin_user: "Admin"
    mw_default_admin_password: "my-secret-password-very-42"
    ```

4.  Customize the wikis by editing the `wikis` variable in `ansible/playbook.yml`.
5.  Run the playbook:

    ```bash
    ansible-playbook -i ansible/inventory --ask-vault-pass ansible/playbook.yml
    ```
6.  Create a `.env` file in the `/opt/wiki` folder on the web server with your database and MediaWiki defaults:

    ```bash
    # Database settings
    DB_ROOT_PASSWORD=verysecret
    DB_NAME_WIKI1=wiki1
    DB_USER=mediawiki
    DB_PASSWORD=secret

    # MediaWiki defaults
    MW_DB_TYPE=mysql
    MW_DB_HOST=mariadb
    ```
7.  Run `podman compose up -d` in the `/opt/wiki` folder on the web server.

## Backup

The playbook includes a backup role that performs daily backups of the MediaWiki farm.

### Backup Strategy

*   **What is backed up:**
    *   A full logical backup of all MariaDB databases.
    *   User-uploaded files (images, documents, etc.).
    *   MediaWiki configuration files (`LocalSettings.php`).
    *   Caddy and Apache configuration files.
    *   The `docker-compose.yml` file.
*   **When:** Backups are performed daily at 2:00 AM.
*   **Where:** Backups are stored in `/backup/wiki` on the host.
*   **Rotation:** Backups are organized into `daily`, `weekly`, and `monthly` folders. The following number of backups are kept:
    *   Daily: 7
    *   Weekly: 4
    *   Monthly: 12

### How to restore from a backup

1.  Identify the backup you want to restore from in the `/backup/wiki` directory.
2.  Unpack the tarball to a temporary location.
3.  Restore the database from the SQL dump.
4.  Copy the files to their original locations.
5.  Restart the webapp services: `podman-compose -f /opt/wiki/docker-compose.yml restart`

### Configuration

The backup strategy can be configured using the following variables in `ansible/playbook.yml`:

*   `backup_path`: The path where backups are stored. Default: `/backup/wiki`.
*   `backup_keep_daily`: The number of daily backups to keep. Default: `7`.
*   `backup_keep_weekly`: The number of weekly backups to keep. Default: `4`.
*   `backup_keep_monthly`: The number of monthly backups to keep. Default: `12`.

## Directory structure

*   `ansible/`: Contains the Ansible playbook and related files.
    *   `playbook.yml`: The main playbook.
    *   `inventory`: The inventory file.
    *   `roles/`: Contains the Ansible roles.
        *   `podman/`: The role to install and configure Podman and Podman Compose.
        *   `webapp/`: The role to deploy the MediaWiki farm.
        *   `backup/`: The role to configure backups.
