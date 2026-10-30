# Ansible Role: promtail

[![Test](https://github.com/aruhier/ansible-role-promtail/actions/workflows/test.yml/badge.svg)](https://github.com/aruhier/ansible-role-promtail/actions/workflows/test.yml)
[![License](https://img.shields.io/badge/license-MIT%20License-brightgreen.svg)](https://opensource.org/licenses/MIT)

## Description

Fork of https://github.com/patrickjahns/ansible-role-promtail.

This fork adds support for installation from distribution repositories (Arch Linux and Alpine use their packages by default).

Deploy [promtail](https://github.com/grafana/loki) using ansible. Supports amd64 and arm architectures.

> [!WARNING]
> Promtail is end-of-life upstream; Grafana recommends migrating to [Grafana Alloy](https://grafana.com/docs/alloy/latest/).
> Loki **3.6.11** is the last release publishing promtail binaries, so the release install is pinned to it.
> `promtail_version: latest` is no longer supported.


## Requirements

- ansible-core >= 2.15

## Role Variables

All variables which can be overridden are stored in [defaults/main.yml](defaults/main.yml) file as well as in table below.

| Name                                             | Default Value                                                    | Description                                                                                                            |
|--------------------------------------------------|------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------|
| `promtail_version`                               | "3.6.11"                                                         | promtail release version (3.6.11 is the last one published upstream)                                                   |
| `promtail_custom_checksum`                       | ""                                                               | Custom checksum for custom build promtail binaries                                                                     |
| `promtail_config_dir`                            | /etc/promtail                                                    | Directory for storing promtail configuration file                                                                      |
| `promtail_config_expand_env`                     | false                                                            | value of promtail [-config.expand-env](https://grafana.com/docs/loki/latest/send-data/promtail/configuration/#use-environment-variables-in-the-configuration) option |
| `promtail_config_file_sd_dir`                    | "{{ promtail_config_dir }}/file_sd"                              | Default directory for `file_sd` discovery                                                                              |
| `promtail_config_file`                           | "{{ promtail_config_dir }}/promtail.yml"                         | Configuration file used by promtail                                                                                    |
| `promtail_system_user`                           | promtail                                                         | User the promtail process will run at                                                                                  |
| `promtail_system_group`                          | "{{ promtail_system_user }}"                                     | Group of the *promtail* user                                                                                           |
| `promtail_user_additional_groups`                | "adm"                                                            | Additional groups to be added to *promtail* user to give access to allow scraping of specific log files                |
| `promtail_config_clients`                        | see [defaults/main.yml](defaults/main.yml)                       | promtail [clients](https://grafana.com/docs/loki/latest/send-data/promtail/configuration/#clients) section              |
| `promtail_loki_server_url`                       | http://127.0.0.1:3100                                            | Server url where promtail will push its result                                                                         |
| `promtail_config_server`                         | see [defaults/main.yml](defaults/main.yml)                       | promtail [server](https://grafana.com/docs/loki/latest/send-data/promtail/configuration/#server) section                 |
| `promtail_positions_directory`                   | `/var/lib/promtail`                                              | Path to the directory where promtail tracks scraped log positions                                                       |
| `promtail_config_positions`                      | {"filename": "{{ promtail_positions_directory }}/positions.yml"} | promtail [positions](https://grafana.com/docs/loki/latest/send-data/promtail/configuration/#positions) section           |
| `promtail_config_scrape_configs`                 | []                                                               | promtail [scrape_configs](https://grafana.com/docs/loki/latest/send-data/promtail/configuration/#scrape_configs) section |
| `promtail_target_config`                         | {}                                                               | promtail [target_config](https://grafana.com/docs/loki/latest/send-data/promtail/configuration/#target_config) section   |
| `promtail_log_level`                             | "info"                                                           | Loglevel of promtail (one of: `debug`,`info`,`warn`,`error` )                                                          |
| `promtail_config_include_default_file_sd_config` | true                                                             | When set to false, the default `file_sd` will not be provisioned                                                       |
| `promtail_install_from_repo`                     | false (true on Arch Linux and Alpine)                            | Install promtail from the distribution repositories instead of the GitHub release                                      |
| `promtail_packages`                              | ["promtail"]                                                     | Packages installed when `promtail_install_from_repo` is true                                                           |
| `promtail_service`                               | promtail                                                         | Name of the promtail service                                                                                           |
| `promtail_service_template_file`                 | service.j2                                                       | Template used for the systemd unit when installing from release                                                        |

For each section (`promtail_config_clients`, `promtail_config_server`,`promtail_config_positions`,`promtail_config_scrape_configs`,`promtail_target_config`) the configuration can be passed accordingly to the [official promtail configuration](https://grafana.com/docs/loki/latest/send-data/promtail/configuration/).
The role will convert the ansible vars into the respective yaml configuration for loki.

## Example Playbook

Basic playbook that will assume that loki will be listening at `http://127.0.0.1:3100` and a simple configuration to scrape `/var/log` logs:

```yaml
---
- hosts: all
  roles:
    - role: aruhier.promtail
      vars:
        promtail_config_scrape_configs:
          - job_name: system
            static_configs:
            - targets:
                - localhost
              labels:
                job: varlogs
                __path__: /var/log/*log
```

A more complex example, that overrides server, client, positions configuration and provides a scrape configuration for `/var/log`:

```yaml
---
- hosts: all
  roles:
    - role: aruhier.promtail
      vars:
        promtail_config_server:
          http_listen_port: 9080
          grpc_listen_port: 9081
        promtail_config_clients:
          - url: "http://prometheus.domain.tld:3100/loki/api/v1/push"
            external_labels:
              host: "{{ ansible_hostname }}"
        promtail_config_positions:
          filename: "{{ promtail_positions_directory }}/positions.yaml"
          sync_period: "60s"

        promtail_config_scrape_configs:
          - job_name: system
            static_configs:
            - targets:
                - localhost
              labels:
                job: varlogs
                __path__: /var/log/*log
```

## License

This project is licensed under MIT License. See [LICENSE](/LICENSE) for more details.

## Maintainers and Contributors

- [Anthony Ruhier](https://github.com/aruhier)
- [Patrick Jahns](https://github.com/patrickjahns) (original author)
