.. sectnum::

Postgres Backup Operator
========================

Create and store PostgreSQL backups.

Features
--------

- Uses `pg_basebackup`_ utility to take binary backups.
- Uses `BorgBackup`_ to create versioned, deduplicated and encrypted backups.

Requirements
------------

- Debian

Role Vars
---------

``postgres_backup_operator_username``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Linux username.

``postgres_backup_operator_clusters``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

List of the PostgreSQL clusters to backup. See the schema in the examples below.

Example
-------

Configure the backup host
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: yaml

    postgres_backup_operator_clusters:
      - host: 1.2.3.4
        port: 5432
        username: postgres_backup_operator # PostgreSQL user to connect as.
        password: password # BEWARE! Stored in plain text in the ~/.pgpass file!
        backup_type: "physical" # "physical" or "logical"
        max_rate: 8M # Used as is in the "pg_basebackup --max-rate" option.
        borg_repository: /var/backups/postgres/1.2.3.4:5432/ # Path to the borg repository.
        borg_passphrase: passphrase # BEWARE! Stored in plain text in the backup script! Optional.
        cron_month: "*"
        cron_weekday: "*"
        cron_day: "*"
        cron_hour: "0"
        cron_minute: "0"
        suppress_mail_on_success: no # Use chronic to suppress unwanted mail from cron. Default: yes.
        retention_within: 1d # Keep all archives within this interval.
        retention_hourly: 24 # Number of hourly archives to keep.
        retention_daily: 30 # Number of daily archives to keep.
        retention_weekly: 12 # Number of weekly archives to keep.
        retention_monthly: 12 # Number of monthly archives to keep.
        retention_yearly: 3 # Number of yearly archives to keep.
        post_backup_hook: # Optional. Bash snippet to execute after a backup.

Configure the database host
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use the `geerlingguy.postgresql`_ role to create a user and pg_hba.conf entry for backup connections.

.. code:: yaml

    postgresql_users:
      - name: postgres_backup_operator
        password: password
        # REPLICATION is required by the pg_basebackup utility.
        # SUPERUSER is required by the pg_dumpall utility.
        role_attr_flags: LOGIN,REPLICATION,SUPERUSER
        state: present

    postgresql_hba_entries:
      - type: hostssl
        database: replication # Special value for replication connections.
        user: postgres_backup_operator
        address: "10.8.0.0/24" # E.g. connecting inside VPN.
        auth_method: md5

.. _geerlingguy.postgresql: https://github.com/geerlingguy/ansible-role-postgresql
.. _BorgBackup: https://github.com/borgbackup
.. _pg_basebackup: https://www.postgresql.org/docs/10/app-pgbasebackup.html
