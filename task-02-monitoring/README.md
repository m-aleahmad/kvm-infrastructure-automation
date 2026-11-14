# Task 02 – KVM Monitoring and Alerting

## Overview

This task deploys monitoring exporters on the KVM host, configures restricted firewall access, and defines Prometheus alerting rules.

## Components

| Component | Port | Purpose |
|---|---:|---|
| Node Exporter | 9100 | Host CPU, memory, disk and network metrics |
| Libvirt Exporter | 9177 | Libvirt metrics |

Both exporters are managed by systemd and enabled at boot.

## Firewall

A dedicated nftables chain restricts access to exporter ports 9100 and 9177 to the authorized monitoring host.

## Metrics and Alert Rules

Exporter metrics were inspected and researched to identify the available metrics and their actual names. Alert expressions were selected based on the metrics exposed by the installed exporters.

Eight alert rules cover:
- High CPU load
- High and critical memory usage
- Disk read and write latency
- Network packet drops
- Node Exporter availability
- Libvirt metrics collection status

Some expected metrics were not exposed by the exporters in the test environment, so alert rules were limited to metrics that could be identified and verified. RAID-related metrics, for example, were unavailable on the test host.

## Validation

- Both exporter endpoints returned HTTP 200.
- Exporter metrics were inspected.
- All 8 alert rules passed `promtool check rules`.
- Ansible deployment was verified as idempotent (`changed=0` on the second run).

## Deployment

```bash
ansible-playbook site.yml --ask-vault-pass
```
