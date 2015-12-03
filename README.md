# Ansible Role: InfluxDB

Installs and configures InfluxDB on RHEL/CentOS.

[InfluxDB](https://github.com/influxdb/influxdb) is an open source distributed
time series database with no external dependencies. It's useful for recording
metrics, events, and performing analytics.

## Requirements

None.

## Role Variables

Available variables are listed below, along with default values (see
`defaults/main.yml`):

```yaml
influxdb_version: 0.9.5.1
```

InfluxDB version that should be installed. See [https://github.com/influxdb/influxdb/tags](https://github.com/influxdb/influxdb/tags)
for a listing of available versions (e.g. ` 0.9.4.2`, `0.9.5.1`).

```yaml
influxdb_config_path: /etc/influxdb
```

The path in which InfluxDB config files will be stored.

```yaml
influxdb_config_use_template: true
```

Whether to use the included InfluxDB config template. Set this to `false` and
copy your own `influxdb.conf` file into the `influxdb_config_path` if you'd
like to use a more complicated setup. If this variable is set to `true`, the
setup will be derived from defined variables.

```yaml
influxdb_config_template_path: influxdb.conf.j2
```

The config file to be copied (if `influxdb_config_use_template` is `true`).
Defaults to the template inside `templates/influxdb.conf.j2`. This path should
be relative to the directory from which you run your playbook.

**FIXME!**

## Dependencies

None.

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
influxdb_version: 0.9.5.1

# ... etc ...
```

**FIXME!**

## License

MIT.

## Author Information

This role was created in 2015 by [Raymond Wanyoike](https://github.com/rwanyoike).
