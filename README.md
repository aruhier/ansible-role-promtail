# Ansible Role: promtail

[![Test](https://github.com/patrickjahns/ansible-role-promtail/workflows/Test/badge.svg)](https://github.com/patrickjahns/ansible-role-promtail/actions?query=workflow%3ATest+branch%3Amaster)
[![License](https://img.shields.io/badge/license-MIT%20License-brightgreen.svg)](https://opensource.org/licenses/MIT)
[![Ansible Role](https://img.shields.io/badge/ansible%20role-patrickjahns.promtail-blue.svg)](https://galaxy.ansible.com/patrickjahns/promtail/)
[![GitHub tag](https://img.shields.io/github/tag/patrickjahns/ansible-role-promtail.svg)](https://github.com/patrickjahns/ansible-role-promtail/tags)

## Description

Fork of https://github.com/patrickjahns/ansible-role-promtail.

This fork adds support for installation from repository.

Deploy [promtail](https://github.com/grafana/loki) using ansible. Supports amd64 and arm architectures.


## Requirements

- Ansible >= 2.7

## Role Variables

All variables which can be overridden are stored in [defaults/main.yml](defaults/main.yml) file as well as in table below.

| Name                                             | Default Value                                                    | Description                                                                                                            |
|--------------------------------------------------|------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------|
| `promtail_version`                               | "2.7.3"                                                          | promtail package version. Also accepts *latest* as parameter.                                                          |
| `promtail_custom_checksum`                       | ""                                                               | Custom checksum for custom build promtail binaries                                                                     |
| `promtail_config_dir`                            | /etc/promtail                                                    | Directory for storing promtail configuration file                                                                      |
| `promtail_config_expand_env`                     | "false"                                                          | value of promtail [-config.expand-env](https://grafana.com/docs/loki/latest/clients/promtail/configuration/#use-environment-variables-in-the-configuration) option |
| `promtail_config_file_sd_dir`                    | "{{ promtail_config_dir }}/file_sd"                              | Default directory for `file_sd` discovery                                                                              |
| `promtail_config_file`                           | "{{ promtail_config_dir }}/promtail.yml"                         | Configuration file used by promtail                                                                                    |
| `promtail_system_user`                           | promtail                                                         | User the promtail process will run at                                                                                  |
| `promtail_system_group`                          | "{{ promtail_system_user }}"                                     | Group of the *promtail* user                                                                                           |
| `promtail_user_additional_groups`                | "adm"                                                            | Additional groups to be added to *promtail* user to give access to allow scraping of specific log files                |
| `promtail_config_clients`                        | see [defaults/main.yml](defaults/main.yml)                       | promtail [clients](https://grafana.com/docs/loki/latest/clients/promtail/configuration/#clientsg) section              |
| `promtail_loki_server_url`                       | http://127.0.0.1:3100                                            | Server url where promtail will push its result                                                                         |
| `promtail_config_server`                         | see [defaults/main.yml](defaults/main.yml)                       | promtail [server](https://grafana.com/docs/loki/latest/clients/promtail/configuration/#server) section                 |
| `promtail_positions_directory`                   | `/var/lib/promtail`                                              | Path to the directory where promtail tracks scraped log positons                                                       |
| `promtail_config_positions`                      | {"filename": "{{ promtail_positions_directory }}/positions.yml"} | promtail [positions](https://grafana.com/docs/loki/latest/clients/promtail/configuration/#positions) section           |
| `promtail_config_scrape_configs`                 | []                                                               | promtail [scrape_configs](https://grafana.com/docs/loki/latest/clients/promtail/configuration/#scrape_configs) section |
| `promtail_target_config`                         | {}                                                               | promtail [target_config](https://grafana.com/docs/loki/latest/clients/promtail/configuration/#target_config) section   |
| `promtail_log_level`                             | "info"                                                           | Loglevel of promtail (one of: `debug`,`info`,`warn`,`error` )                                                          |
| `promtail_config_include_default_file_sd_config` | "True"                                                           | When set to false, the default `file_sd` will not be provisioned                                                       |
| `promtail_apt_update_cache`                      | "True"                                                           | When set to false the role will not update the APT cache on its own                                                    |

For each section (`promtail_config_clients`, `promtail_config_server`,`promtail_config_positions`,`promtail_config_scrape_configs`,`promtail_target_config`) the configuration can be passed accrodingly to the [official promtail configuration](https://github.com/grafana/loki/blob/master/docs/clients/promtail/configuration.md).
The role will converte the ansible vars into the respective yaml configuration for loki.

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

A more complex example, that overrides server, client, positions configuration and provides a scrap configuration for `/var/log`:

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

- [Patrick Jahns](https://github.com/patrickjahns)
