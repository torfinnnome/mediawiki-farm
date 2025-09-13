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

## Directory structure

*   `ansible/`: Contains the Ansible playbook and related files.
    *   `playbook.yml`: The main playbook.
    *   `inventory`: The inventory file.
    *   `roles/`: Contains the Ansible roles.
        *   `podman/`: The role to install and configure Podman and Podman Compose.
        *   `webapp/`: The role to deploy the MediaWiki farm.

