# ansible-role-grafana-agent
Install and Configure [Grafana-Agent](https://github.com/grafana/agent) with Ansible

## [Example Playbook](#example-playbook)

This example is taken from `molecule/resources/converge.yml` and is tested on each push, pull request and release.
```yaml
---
- name: converge
  hosts: all
  become: yes
  gather_facts: yes

  roles:
    - role: ansible-role-grafana-exporter
```

## [Role Variables](#role-variables)

for agent configuration have a look at [Grafana Agent Documentation](https://github.com/grafana/agent/blob/main/docs/configuration/_index.md)

These variables are set in `defaults/main.yml`:
```yaml
---
# defaults for grafana-agent

agent_version: "0.21.2"
agent_arch: amd64
agent_url: "https://github.com/grafana/agent/releases/download/v{{ agent_version }}/grafana-agent-{{ agent_version }}-1.{{ agent_arch }}.deb"
agent_runasroot: false

agent_config:
  server:
    http_listen_address: '127.0.0.1'
    http_listen_port: 9099
    grpc_listen_address: '127.0.0.1'
    grpc_listen_port: 19095
  metrics:
    global:
      scrape_interval: 15s
      remote_write:
        - url: http://cortex.service.consul:9009/api/v1/push
    wal_directory: '/var/lib/grafana-agent'
    configs:
    - name: agent
      host_filter: false
      scrape_configs:
        - job_name: agent
          static_configs:
            - targets: ['127.0.0.1:9099']
  integrations:
    agent:
      enabled: true
    node_exporter:
      enabled: true
      include_exporter_metrics: true
      enable_collectors:
        - "systemd"
      disable_collectors:
        - "mdadm"
    consul_exporter:
      enabled: true
  logs:
    configs:
    - name: default
      positions:
        filename: /tmp/positions.yaml
      scrape_configs:
        - job_name: varlogs
          static_configs:
            - targets: [localhost]
              labels:
                job: varlogs
                __path__: /var/log/*log
        - job_name: journal
          journal:
            max_age: 12h
            labels:
              job: systemd-journal
          relabel_configs:
            - source_labels: ['__journal__systemd_unit']
              target_label: 'unit'
      clients:
        - url: http://loki.service.consul:3100/loki/api/v1/push
```

## [Compatibility](#compatibility)

This role has been tested on these [container images](https://hub.docker.com/u/robertdebock):

|container|tags|
|---------|----|
|debian|buster|
|ubuntu|bionic, focal|
|RHEL|7, 8|
|Fedora|33, 34|
|FreeBSD|12, 13|

The minimum version of Ansible required is 2.10, tests have been done to:

- The previous version.
- The current version.
- The development version.



If you find issues, please register them in [GitHub](https://github.com/langerma/ansible-role-grafana-agent/issues)
