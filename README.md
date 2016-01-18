# Ansible Role: InfluxDB

[![Build Status](https://img.shields.io/travis/rwanyoike/ansible-role-influxdb.svg)](https://travis-ci.org/rwanyoike/ansible-role-influxdb) [![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/rwanyoike/ansible-role-influxdb/master/LICENSE)

Installs and configures InfluxDB on RHEL/CentOS ~~or Debian/Ubuntu~~.

## Requirements

None

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
# Version that should be installed
influxdb_version: 0.9.6.1

# Use the included config template
influxdb_conf_use_template: true

# Once every 24 hours InfluxDB will report anonymous data to m.influxdb.com.
# The data includes raft id (random 8 bytes), os, arch, version, and metadata.
# We don't track ip addresses of servers reporting. This is only used to track
# the number of instances running and the versions, which is very helpful for
# us. Change this option to true to disable reporting
influxdb_conf_reporting_disabled: "false"

# Controls the parameters for the Raft consensus group that stores metadata
# about the InfluxDB cluster
influxdb_conf_meta_dir: /var/lib/influxdb/meta
influxdb_conf_meta_hostname: localhost
influxdb_conf_meta_bind_address: :8088
influxdb_conf_meta_retention_autocreate: "true"
influxdb_conf_meta_election_timeout: 1s
influxdb_conf_meta_heartbeat_timeout: 1s
influxdb_conf_meta_leader_lease_timeout: 500ms
influxdb_conf_meta_commit_timeout: 50ms
influxdb_conf_meta_cluster_tracing: "false"
# If enabled, when a Raft cluster loses a peer due to a `DROP SERVER` command,
# the leader will automatically ask a non-raft peer node to promote to a raft
# peer. This only happens if there is a non-raft peer node available to
# promote. This setting only affects the local node, so to ensure if operates
# correctly, be sure to set it in the config of every node
influxdb_conf_meta_raft_promotion_enabled: "true"
influxdb_conf_meta_other: []

# Controls where the actual shard data for InfluxDB lives and how it is flushed
# from the WAL. "dir" may need to be changed to a suitable place for your
# system, but the WAL settings are an advanced configuration. The defaults
# should work for most systems
influxdb_conf_data_dir: /var/lib/influxdb/data
# The following WAL settings are for the b1 storage engine used in 0.9.2. They
# won't apply to any new shards created after upgrading to a version > 0.9.3
influxdb_conf_data_max_wal_size: 104857600 # Maximum size the WAL can reach before a flush. Defaults to 100MB
influxdb_conf_data_wal_flush_interval: 10m # Maximum time data can sit in WAL before a flush
influxdb_conf_data_wal_partition_flush_delay: 2s # The delay time between each WAL partition being flushed
# These are the WAL settings for the storage engine >= 0.9.3
influxdb_conf_data_wal_dir: /var/lib/influxdb/wal
influxdb_conf_data_wal_logging_enabled: "true"
influxdb_conf_data_data_logging_enabled: "true"
influxdb_conf_data_other: []

# Controls the hinted handoff feature, which allows nodes to temporarily store
# queued data when one node of a cluster is down for a short period of time
influxdb_conf_hh_enabled: "true"
influxdb_conf_hh_dir: /var/lib/influxdb/hh
influxdb_conf_hh_max_size: 1073741824
influxdb_conf_hh_max_age: 168h
influxdb_conf_hh_retry_rate_limit: 0
influxdb_conf_hh_retry_interval: 1s
influxdb_conf_hh_retry_max_interval: 1m
# Interval between running checks for data that should be purged. Data is
# purged from hinted-handoff queues for two reasons. 1) The data is older than
# the max age, or 2) the target node has been dropped from the cluster. Data is
# never dropped until it has reached max-age however, for a dropped node or not
influxdb_conf_hh_purge_interval: 1h
influxdb_conf_hh_other: []

# Config other sections
influxdb_conf_other:
  # Controls non-Raft cluster behavior, which generally includes how data is
  # shared across shards
  cluster:
      # The time within which a shard must respond to write
    - shard-writer-timeout = "5s"
      # The time within which a write operation must complete on the cluster
    - write-timeout = "10s"
  # Controls the enforcement of retention policies for evicting old data
  retention:
    - enabled = true
    - check-interval = "30m"
  # Controls the precreation of shards, so they are created before data
  # arrives. Only shards that will exist in the future, at time of creation,
  # are precreated
  shard-precreation:
    - enabled = true
    - check-interval = "10m"
    - advance-period = "30m"
  # Controls the system self-monitoring, statistics and diagnostics
  monitor:
      # Whether to record statistics internally
    - store-enabled = true
      # The destination database for recorded statistics
    - store-database = "_internal"
      # The interval at which to record statistics
    - store-interval = "10s"
  # Controls the availability of the built-in, web-based admin interface. If
  # HTTPS is enabled for the admin interface, HTTPS must also be enabled on the
  # [http] service
  admin:
    - enabled = true
    - bind-address = ":8083"
    - https-enabled = false
    - https-certificate = "/etc/ssl/influxdb.pem"
  # Controls how the HTTP endpoints are configured. These are the primary
  # mechanism - for getting data into and out of InfluxDB
  http:
    - enabled = true
    - bind-address = ":8086"
    - auth-enabled = false
    - log-enabled = true
    - write-tracing = false
    - pprof-enabled = false
    - https-enabled = false
    - https-certificate = "/etc/ssl/influxdb.pem"
  # Controls one or many listeners for Graphite data
  "[graphite]":
    - enabled = false
  # Controls the listener for collectd data
  collectd:
    - enabled = false
  # Controls the listener for OpenTSDB data
  opentsdb:
    - enabled = false
  # Controls the listeners for InfluxDB line protocol data via UDP
  "[udp]":
    - enabled = false
  # Controls how continuous queries are run within InfluxDB
  continuous_queries:
    - log-enabled = true
    - enabled = true
    - recompute-previous-n = 2
    - recompute-no-older-than = "10m"
    - compute-runs-per-interval = 10
    - compute-no-more-than = "2m"
```

Default values for the `influxdb.conf.j2` template are based on the [0.9.6.1 config](https://github.com/influxdata/influxdb/blob/v0.9.6.1/etc/config.sample.toml).

## Dependencies

None

## Example Playbook

```yaml
- hosts: webservers

  vars_files:
    - vars/main.yml

  roles:
    - role: rwanyoike.influxdb
```

Inside `vars/main.yml:`

```yaml
influxdb_version: 0.9.6.1

# ... etc ...
```

## License

MIT

## Author Information

- This role was created in 2015 by [Raymond Wanyoike](https://github.com/rwanyoike).
