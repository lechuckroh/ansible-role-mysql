# Ansible role `mysql`

An ansible role for managing MySQL

* Install MySQL packages.
* Set/Reset root password.
* Remove `test` database.
* Remove anonymoyus users.
* Create users and databases.
* Manage configuration file `my.cnf`
* Replication

Fixed settings:
* Storage engine : InnoDB

Tested against:
* CentOS 7.4
* Ubuntu 16.04 (Xenial)
* Ubuntu 18.04 (Bionic)


## Requirements

No requirements

## Role Variables

| Variable                               | Default           | Comments                                                                                                    |
|:---------------------------------------|:------------------|:------------------------------------------------------------------------------------------------------------|
| `mysql_bind_address`                   | '127.0.0.1'       | IP address of the network interface to listen on. '0.0.0.0' for all interfaces. |
| `mysql_binlog_format`                  | 'ROW'             | binary logging format (`STATEMENT`, `ROW`, `MIXED`) |
| `mysql_charset`                        | 'utf8'            | |
| `mysql_collation`                      | 'utf8_general_ci' | |
| `mysql_databases`                      | []                | databases to add. See [mysql_db](http://docs.ansible.com/ansible/devel/modules/mysql_db_module.html#parameters) |
| `mysql_enabled_on_startup`             | true              | 'true': enable MySQL on startup. |
| `mysql_expire_logs_days`               | 10                | the number of days before automatic removal of binary log files. |
| `mysql_group_concat_max_len`           | 1024              | |
| `mysql_innodb_buffer_pool_size`        | 256M              | |
| `mysql_innodb_file_per_table`          | ON                | |
| `mysql_innodb_flush_log_at_trx_commit` | 1                 | |
| `mysql_innodb_log_buffer_size`         | 8M                | |
| `mysql_innodb_log_file_size`           | 64M               | |
| `mysql_innodb_lock_wait_timeout`       | 50                | |
| `mysql_join_buffer_size`               | 256K              | |
| `mysql_key_buffer_size`                | 256M              | |
| `mysql_lower_case_table_names`         | 1                 | 0: case-sensitive(Linux), 1: stored in lowercase, case-insensitive(Windows), 2: stored as declared, compared in lowercase(OSX) |
| `mysql_max_allowed_packet`             | 64M               | |
| `mysql_max_binlog_size`                | 100M              | max binary log size (4096 byte ~ 1GB) |
| `mysql_max_connections`                | 151               | |
| `mysql_max_heap_table_size`            | 16M               | |
| `mysql_myisam_sort_buffer_size`        | 64M               | |
| `mysql_mysqldump_max_allowed_packet`   | 64M               | |
| `mysql_overwrite_global_mycnf`         | true              | `true`: global `my.cnf` should be overwritten each time this role is run. |
| `mysql_port`                           | 3306              | port number |
| `mysql_read_buffer_size`               | 1M                | |
| `mysql_read_rnd_buffer_size`           | 4M                | |
| `mysql_replication_master`             | ''                | replication master |
| `mysql_replication_master_host`        | ''                | replication master host (default: `hostvars[mysql_replication_master]['ansible_host']`) |
| `mysql_replication_role`               | ''                | replication role (`master`, `slave`) |
| `mysql_replication_user`               | []                | replication user (`name`, `password` required) |
| `mysql_root_password`                  | 'root'            | root password |
| `mysql_root_remote`                    | `no`              | `yes`: enable remote root login |
| `mysql_root_remote_host`               | '%'               | remote root login enabled host |
| `mysql_server_id`                      | ''                | server id (replication) |
| `mysql_skip_name_resolve`              | 1                 | |
| `mysql_sort_buffer_size`               | 1M                | |
| `mysql_sql_mode`                       | NO_AUTO_CREATE_USER, NO_ENGINE_SUBSTITUTION, STRICT_TRANS_TABLES, ERROR_FOR_DIVISION_BY_ZERO | |
| `mysql_table_open_cache`               | 256               | |
| `mysql_thread_cache_size`              | 8                 | |
| `mysql_tmp_table_size`                 | 16M               | |
| `mysql_users`                          | []                | users to add |
| `mysql_wait_timeout`                   | 28800             | |

## Dependencies

No dependencies

## Example Playbook

### Create database and users

```yaml
---
- name: example
  hosts: all
  become: true
  vars:
    mysql_bind_address: '0.0.0.0'
    mysql_root_password: 'root'
    mysql_root_remote: yes
    mysql_databases:
    - name: dev
    mysql_users:
    - name: dev
      password: 'devdev'
      priv: "dev.*:ALL"
      host: "%"
  roles:
  - lechuckroh.mysql
```

### Replication
host inventory
```ini
mysql-master ansible_host=192.168.50.11 ansible_user=vagrant
mysql-slave ansible_host=192.168.50.12 ansible_user=vagrant
```

Master playbook
```yaml
---
- name: Install mysql on Master
  hosts: mysql-master
  become: true
  vars:
    mysql_bind_address: '0.0.0.0'
    mysql_root_password: 'root'
    mysql_databases:
    - name: dev1
    - name: dev2
      replicate: yes
    - name: dev3
      replicate: no
    mysql_users:
    - name: dev
      password: 'devdev'
      priv: "dev*.*:ALL"
      host: "%"

    mysql_server_id: 1
    mysql_max_binlog_size: 100M
    mysql_binlog_format: ROW
    mysql_expire_logs_days: 10
    mysql_replication_role: master
    mysql_replication_master: mysql-master
    mysql_replication_user:
      name: repl
      password: repl

  tasks:
  - include_role:
      name: lechuckroh.mysql
```

Slave playbook
```yaml
---
- name: Install mysql on Master
  hosts: mysql-slave
  become: true
  vars:
    mysql_bind_address: '0.0.0.0'
    mysql_root_password: 'root'
    mysql_databases:
    - name: dev1
    - name: dev2

    mysql_server_id: 2
    mysql_replication_role: master
    mysql_replication_master: mysql-master
    mysql_replication_master_host: '192.168.50.12'
    mysql_replication_user:
      name: repl
      password: repl

  tasks:
  - include_role:
      name: lechuckroh.mysql
```

## License
MIT
