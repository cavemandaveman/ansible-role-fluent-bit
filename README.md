# Ansible Role: Fluent Bit

An Ansible Role that installs [Fluent Bit](https://fluentbit.io/) on Linux.

## Requirements

N/A

## Role Variables

For a deep dive on configuration, the [Fluent Bit Docs](https://docs.fluentbit.io/manual/) are a great resource.

The repo to download Fluent Bit from (Debian based systems only):
```yaml
fluentbit_apt_repo: deb https://packages.fluentbit.io/ubuntu/focal focal main
```

A list of name, value hashes for the Service section of the [configuration file](https://docs.fluentbit.io/manual/administration/configuring-fluent-bit/configuration-file) (these are the default install values):
```yaml
fluentbit_conf_service_settings:
  - name: flush
    value: 5
  - name: daemon
    value: 'Off'
  - name: log_level
    value: info
  - name: parsers_file
    value: parsers.conf
  - name: plugins_file
    value: plugins.conf
  - name: http_server
    value: 'Off'
  - name: http_listen
    value: 0.0.0.0
  - name: http_port
    value: 2020
  - name: storage.metrics
    value: 'on'
```

A list of hashes to use as Inputs in the configuration file. The key/value pairs are dictated by the availble values of the different [types of inputs](https://docs.fluentbit.io/manual/pipeline/inputs):
```yaml
fluentbit_conf_inputs:
  - name: cpu
    tag: cpu.local
    interval_sec: 1
```

A list of hashes to use as Outputs in the configuration file. The key/value pairs are dictated by the availble values of the different [types of outputs](https://docs.fluentbit.io/manual/pipeline/outputs):
```yaml
fluentbit_conf_outputs:
  - name: stdout
    match: '*'
```

Similar to the above settings, a list of hashes to use as Filters in the configuration file. The key/value pairs are dictated by the availble values of the different [types of filters](https://docs.fluentbit.io/manual/pipeline/filters):
```yaml
fluentbit_conf_filters: []
```

A list of custom configuration files to copy to the host. Files will be copied to `/etc/td-agent-bit/custom/conf/` and included in the main configuration file.
```yaml
fluentbit_custom_conf_files: []
```

A list of custom parsers files to copy to the host. Files will be copied to `/etc/td-agent-bit/custom/parsers/`.
```yaml
fluentbit_custom_parsers_files: []
```

If you use custom parsers files, you must add additional `parsers_file` settings with `custom/parsers/` as the directory in the value, like so:
```yaml
fluentbit_conf_service_settings:
  ...
  - name: parsers_file
    value: custom/parsers/<name of parsers file 1>
  - name: parsers_file
    value: custom/parsers/<name of parsers file 2>
  ...
```

## Dependencies

N/A

## Example Playbook

Specify custom config and parsers files, grab logs from a file as well as the Docker service systemd logs, use a filter to add the hostname, and send them to a webserver:

```yaml
- hosts: logging_servers
  become: yes
  roles:
    - role: cavemandaveman.fluentbit
      vars:
        fluentbit_conf_service_settings:
          - name: parsers_file
            value: custom/parsers/my_parsers.conf
        fluentbit_conf_inputs:
          - name: tail
            path: /var/log/myapp.log
          - name: systemd
            tag: host.*
            systemd_filter: _SYSTEMD_UNIT=docker.service
        fluentbit_conf_outputs:
          - name: http
            match: '*'
            host: 192.168.1.10
            port: 80
            uri: /something
        fluentbit_conf_filters:
          - name: record_modifier
            match: '*'
            record: hostname ${HOSTNAME}
        fluentbit_custom_conf_files:
          - resources/my_conf.conf
        fluentbit_custom_parsers_files:
          - resources/my_parsers.conf
```

## License

GPLv3

## Author Information

This role was created in 2021 by cavemandaveman.
