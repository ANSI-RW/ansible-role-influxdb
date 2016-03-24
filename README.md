# Ansible Role: InfluxDB

[![Build Status](https://img.shields.io/travis/ANSI-RW/ansible-role-influxdb.svg)](https://travis-ci.org/ANSI-RW/ansible-role-influxdb) [![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/ANSI-RW/ansible-role-influxdb/master/LICENSE)

Installs and configures InfluxDB on RHEL/CentOS or Debian/Ubuntu.

## Requirements

None

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
# Version that should be installed
influxdb_version: 0.11.0
# Use the included config template
influxdb_conf_use_template: true

influxdb_conf_reporting_disabled: "false"

influxdb_conf_meta_enabled: "true"
influxdb_conf_meta_dir: /var/lib/influxdb/meta
influxdb_conf_meta_bind_address: :8088
influxdb_conf_meta_http_bind_address: :8091
influxdb_conf_meta_https_enabled: "false"
influxdb_conf_meta_https_certificate: ""
influxdb_conf_meta_retention_autocreate: "true"
influxdb_conf_meta_election_timeout: 1s
influxdb_conf_meta_heartbeat_timeout: 1s
influxdb_conf_meta_leader_lease_timeout: 500ms
influxdb_conf_meta_commit_timeout: 50ms
influxdb_conf_meta_cluster_tracing: "false"
influxdb_conf_meta_raft_promotion_enabled: "true"
influxdb_conf_meta_logging_enabled: "true"
influxdb_conf_meta_pprof_enabled: "false"
influxdb_conf_meta_lease_duration: 1m0s
# Other [meta] config
influxdb_conf_meta_other: []

influxdb_conf_data_enabled: "true"
influxdb_conf_data_dir: /var/lib/influxdb/data
influxdb_conf_data_wal_dir: /var/lib/influxdb/wal
influxdb_conf_data_wal_logging_enabled: "true"
influxdb_conf_data_data_logging_enabled: "true"
# Other [data] config
influxdb_conf_data_other: []

influxdb_conf_hh_enabled: "true"
influxdb_conf_hh_dir: /var/lib/influxdb/hh
influxdb_conf_hh_max_size: 1073741824
influxdb_conf_hh_max_age: 168h
influxdb_conf_hh_retry_rate_limit: 0
influxdb_conf_hh_retry_interval: 1s
influxdb_conf_hh_retry_max_interval: 1m
influxdb_conf_hh_purge_interval: 1h
# Other [hinted-handoff] config
influxdb_conf_hh_other: []

# Other section config
influxdb_conf_other:
  cluster:
    - shard-writer-timeout = "5s"
    - write-timeout = "10s"
  retention:
    - enabled = true
    - check-interval = "30m"
  shard-precreation:
    - enabled = true
    - check-interval = "10m"
    - advance-period = "30m"
  monitor:
    - store-enabled = true
    - store-database = "_internal"
    - store-interval = "10s"
  admin:
    - enabled = true
    - bind-address = ":8083"
    - https-enabled = false
    - https-certificate = "/etc/ssl/influxdb.pem"
  http:
    - enabled = true
    - bind-address = ":8086"
    - auth-enabled = false
    - log-enabled = true
    - write-tracing = false
    - pprof-enabled = false
    - https-enabled = false
    - https-certificate = "/etc/ssl/influxdb.pem"
  "[graphite]":
    - enabled = false
  collectd:
    - enabled = false
  opentsdb:
    - enabled = false
  "[udp]":
    - enabled = false
  continuous_queries:
    - log-enabled = true
    - enabled = true
```

Default values for the `influxdb.conf.j2` template are based on the [0.11.0 config](https://github.com/influxdata/influxdb/blob/v0.11.0/etc/config.sample.toml).

## Dependencies

None

## Example Playbook

```yaml
- hosts: servers

  vars_files:
    - vars/main.yml

  roles:
    - { role: ANSI-RW.influxdb }
```

Inside `vars/main.yml`:

```yaml
influxdb_conf_data_dir: /mnt/influxdb/data
influxdb_conf_data_wal_dir: /mnt/influxdb/wal
influxdb_conf_hh_dir: /mnt/influxdb/hh
influxdb_conf_meta_dir: /mnt/influxdb/meta

# ... etc ...
```

## License

MIT
