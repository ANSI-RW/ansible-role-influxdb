# Ansible Role: InfluxDB

[![Build Status](https://img.shields.io/travis/rwanyoike/ansible-role-influxdb.svg)](https://travis-ci.org/rwanyoike/ansible-role-influxdb) [![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/rwanyoike/ansible-role-influxdb/master/LICENSE)

Installs and configures InfluxDB on RHEL/CentOS ~~or Debian/Ubuntu~~.

## Requirements

None

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
# Version that should be installed.
influxdb_version: 0.9.6.1

# Use the included config template.
influxdb_config_use_template: true
# Where config files will be stored.
influxdb_config_path: /etc/influxdb
# Config [meta] section.
influxdb_config_meta_dir: /var/lib/influxdb/meta
# Controls the parameters for the Raft consensus group that stores metadata
# about the InfluxDB cluster.
influxdb_config_meta:
  - hostname = "localhost"
  - bind-address = ":8088"
  - retention-autocreate = true
  - election-timeout = "1s"
  - heartbeat-timeout = "1s"
  - leader-lease-timeout = "500ms"
  - commit-timeout = "50ms"
  - cluster-tracing = false
  # If enabled, when a Raft cluster loses a peer due to a `DROP SERVER`
  # command, the leader will automatically ask a non-raft peer node to promote
  # to a raft peer. This only happens if there is a non-raft peer node
  # available to promote. This setting only affects the local node, so to
  # ensure if operates correctly, be sure to set it in the config of every
  # node.
  - raft-promotion-enabled = true
# Config [data] section.
influxdb_config_data_dir: /var/lib/influxdb/data
influxdb_config_wal_dir: /var/lib/influxdb/wal
# Controls where the actual shard data for InfluxDB lives and how it is flushed
# from the WAL. "dir" may need to be changed to a suitable place for your
# system, but the WAL settings are an advanced configuration. The defaults
# should work for most systems.
influxdb_config_data:
  # The following WAL settings are for the b1 storage engine used in 0.9.2.
  # They won't apply to any new shards created after upgrading to a version >
  # 0.9.3.
  # Maximum size the WAL can reach before a flush. Defaults to 100MB.
  - max-wal-size = 104857600
  # Maximum time data can sit in WAL before a flush.
  - wal-flush-interval = "10m"
  # The delay time between each WAL partition being flushed.
  - wal-partition-flush-delay = "2s"
  - wal-enable-logging = true
# Config [hinted-handoff] section.
influxdb_config_hh_dir: /var/lib/influxdb/hh
# Controls the hinted handoff feature, which allows nodes to temporarily store
# queued data when one node of a cluster is down for a short period of time.
influxdb_config_hh:
  - enabled = true
  - max-size = 1073741824
  - max-age = "168h"
  - retry-rate-limit = 0
  # Hinted handoff will start retrying writes to down nodes at a rate of once
  # per second. If any error occurs, it will backoff in an exponential manner,
  # until the interval reaches retry-max-interval. Once writes to all nodes are
  # successfully completed the interval will reset to retry-interval.
  - retry-interval = "1s"
  - retry-max-interval = "1m"
  # Interval between running checks for data that should be purged. Data is
  # purged from hinted-handoff queues for two reasons. 1) The data is older
  # than the max age, or 2) the target node has been dropped from the cluster.
  # Data is never dropped until it has reached max-age however, for a dropped
  # node or not.
  - purge-interval = "1h"
# Config other sections.
influxdb_config_other:
  # Once every 24 hours InfluxDB will report anonymous data to m.influxdb.com
  # The data includes raft id (random 8 bytes), os, arch, version, and
  # metadata. We don't track ip addresses of servers reporting. This is only
  # used to track the number of instances running and the versions, which is
  # very helpful for us. Change this option to true to disable reporting.
  - reporting-disabled = false
  # Enterprise registration control.
  - section: registration
  # Controls non-Raft cluster behavior, which generally includes how data is
  # shared across shards.
  - section: cluster
    config:
      # The time within which a shard must respond to write.
      - shard-writer-timeout = "5s"
      # The time within which a write operation must complete on the cluster.
      - write-timeout = "10s"
  # Controls the enforcement of retention policies for evicting old data.
  - section: retention
    config:
      - enabled = true
      - check-interval = "30m"
  # Controls the precreation of shards, so they are created before data
  # arrives. Only shards that will exist in the future, at time of creation,
  # are precreated.
  - section: shard-precreation
    config:
      - enabled = true
      - check-interval = "10m"
      - advance-period = "30m"
  # Controls the system self-monitoring, statistics and diagnostics.
  - section: monitor
    config:
      # Whether to record statistics internally.
      - store-enabled = true
      # The destination database for recorded statistics.
      - store-database = "_internal"
      # The interval at which to record statistics.
      - store-interval = "10s"
  # Controls the availability of the built-in, web-based admin interface. If
  # HTTPS is enabled for the admin interface, HTTPS must also be enabled on the
  # [http] service.
  - section: admin
    config:
      - enabled = true
      - bind-address = ":8083"
      - https-enabled = false
      - https-certificate = "/etc/ssl/influxdb.pem"
  # Controls how the HTTP endpoints are configured. These are the primary
  # mechanism - for getting data into and out of InfluxDB.
  - section: http
    config:
      - enabled = true
      - bind-address = ":8086"
      - auth-enabled = false
      - log-enabled = true
      - write-tracing = false
      - pprof-enabled = false
      - https-enabled = false
      - https-certificate = "/etc/ssl/influxdb.pem"
  # Controls one or many listeners for Graphite data.
  - section: "[graphite]"
    config:
      - enabled = false
  # Controls the listener for collectd data.
  - section: collectd
    config:
      - enabled = false
  # Controls the listener for OpenTSDB data.
  - section: opentsdb
    config:
      - enabled = false
  # Controls the listeners for InfluxDB line protocol data via UDP.
  - section: "[udp]"
    config:
      - enabled = false
  # Controls how continuous queries are run within InfluxDB.
  - section: continuous_queries
    config:
      - log-enabled = true
      - enabled = true
      - recompute-previous-n = 2
      - recompute-no-older-than = "10m"
      - compute-runs-per-interval = 10
      - compute-no-more-than = "2m"
```

Default values for the `influxdb.conf.j2` template are based on the [0.9.6.1 sample config](https://github.com/influxdata/influxdb/blob/v0.9.6.1/etc/config.sample.toml).

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
