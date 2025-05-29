# Elasticsearch Ansible Role

An Ansible role for installing and configuring Elasticsearch on various operating systems.

## Features

- Supports both Elasticsearch 8.x and 9.x
- Dynamic OS support (Debian/Ubuntu and RedHat/CentOS)
- Configurable security settings
- Optional Vault integration for certificate management and credential storage
- Support for cluster and single-node deployment modes
- Customizable JVM options and memory settings
- Index Lifecycle Management (ILM) policy configuration

## Requirements

- Ansible 2.9 or higher
- One of the supported operating systems:
  - Ubuntu 20.04 or higher
  - Debian 10 or higher
  - CentOS 8 or higher
  - RHEL 8 or higher
- Sufficient memory and disk space for Elasticsearch
- (Optional) HashiCorp Vault for certificate management and credential storage

## Role Variables

### Basic Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `elasticsearch_major_version` | `9` | Major version of Elasticsearch to install (8 or 9) |
| `elasticsearch_minor_version` | `0.1` | Minor version of Elasticsearch |
| `elasticsearch_version` | `"{{ elasticsearch_major_version }}.{{ elasticsearch_minor_version }}"` | Full version string |
| `elasticsearch_cluster_name` | `""` | Cluster name |
| `elasticsearch_dns_name` | `""` | DNS name for the Elasticsearch node |
| `elasticsearch_use_os_detection` | `true` | Whether to detect OS automatically |

### Directory Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `elasticsearch_data_dir` | `/data/elasticsearch` | Data directory path |
| `elasticsearch_log_dir` | `/var/log/elasticsearch` | Log directory path |
| `elasticsearch_conf_dir` | `/etc/elasticsearch` | Configuration directory path |
| `elasticsearch_certs_dir` | `/etc/elasticsearch/certs` | Certificates directory path |

### Security Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `elasticsearch_security_enabled` | `no` | Whether to enable security features |
| `elasticsearch_https_enabled` | `no` | Whether to enable HTTPS for the REST API |
| `elasticsearch_disable_firewall` | `yes` | Whether to disable the host firewall |

### JVM Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `elasticsearch_jvm_xms` | `4g` | Initial heap size |
| `elasticsearch_jvm_xmx` | `4g` | Maximum heap size |
| `elasticsearch_vm_max_map_count` | `262144` | Maximum number of memory map areas |
| `elasticsearch_vm_swappiness` | `1` | Virtual memory swappiness parameter |

### Vault Integration

| Variable | Default | Description |
|----------|---------|-------------|
| `elasticsearch_use_vault` | `true` | Whether to use Vault for certificates and credentials |
| `elasticsearch_vault_token` | From env | Vault token for authentication |
| `elasticsearch_vault_addr` | From env | Vault server address |
| `elasticsearch_vault_path` | From env | Path where secrets are stored in Vault |

### Local Certificate Configuration (when not using Vault)

| Variable | Default | Description |
|----------|---------|-------------|
| `elasticsearch_local_cert_country` | `AZ` | Country code for certificate |
| `elasticsearch_local_cert_state` | `BAKU` | State for certificate |
| `elasticsearch_local_cert_locality` | `BAKU` | Locality for certificate |
| `elasticsearch_local_cert_organization` | `ORGANIZATION` | Organization for certificate |
| `elasticsearch_local_cert_common_name` | `*.elasticsearch.local` | Common name for certificate |

### Local User Credentials (when not using Vault)

| Variable | Default | Description |
|----------|---------|-------------|
| `elasticsearch_local_superuser_username` | `admin` | Superuser username |
| `elasticsearch_local_superuser_password` | Auto-generated | Superuser password |
| `elasticsearch_local_developer_username` | `developer` | Developer username |
| `elasticsearch_local_developer_password` | Auto-generated | Developer password |
| `elasticsearch_local_kibana_system_username` | `kibana_system` | Kibana system username |
| `elasticsearch_local_kibana_system_password` | Auto-generated | Kibana system password |

## Example Playbook

### Basic installation with Elasticsearch 9

```yaml
- hosts: elasticsearch_nodes
  become: true
  roles:
    - role: elasticsearch
      elasticsearch_cluster_name: "my-cluster"
      elasticsearch_dns_name: "elasticsearch.example.com"
```

### Configuring Elasticsearch 8 with security and without Vault

```yaml
- hosts: elasticsearch_nodes
  become: true
  roles:
    - role: elasticsearch
      elasticsearch_major_version: 8
      elasticsearch_minor_version: 12.0
      elasticsearch_cluster_name: "secure-cluster"
      elasticsearch_dns_name: "es.example.com"
      elasticsearch_security_enabled: yes
      elasticsearch_https_enabled: yes
      elasticsearch_use_vault: no
      elasticsearch_local_cert_organization: "MyCompany"
      elasticsearch_local_cert_common_name: "*.es.example.com"
      elasticsearch_local_superuser_username: "es_admin"
      elasticsearch_local_superuser_password: "secure_password"
```

### Multi-node cluster with Vault integration

```yaml
- hosts: elasticsearch_master
  become: true
  roles:
    - role: elasticsearch
      elasticsearch_cluster_name: "prod-cluster"
      elasticsearch_dns_name: "es-master.example.com"
      elasticsearch_security_enabled: yes
      elasticsearch_https_enabled: yes
      elasticsearch_use_vault: yes
      elasticsearch_vault_addr: "https://vault.example.com:8200"
      elasticsearch_vault_path: "secret/elasticsearch"
      elasticsearch_jvm_xms: "8g"
      elasticsearch_jvm_xmx: "8g"

- hosts: elasticsearch_data
  become: true
  roles:
    - role: elasticsearch
      elasticsearch_cluster_name: "prod-cluster"
      elasticsearch_dns_name: "es-data.example.com"
      elasticsearch_security_enabled: yes
      elasticsearch_https_enabled: yes
      elasticsearch_use_vault: yes
      elasticsearch_vault_addr: "https://vault.example.com:8200"
      elasticsearch_vault_path: "secret/elasticsearch"
      elasticsearch_jvm_xms: "16g"
      elasticsearch_jvm_xmx: "16g"
      elasticsearch_data_dir: "/data/elasticsearch-data"
```

## License

MIT

## Author Information

Created in May 2025.
