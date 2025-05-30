# Elasticsearch Ansible Role

[![GitHub](https://img.shields.io/github/license/smraze96/elasticsearch)](https://github.com/smraze96/elasticsearch/blob/main/LICENSE)
[![Ansible Galaxy](https://img.shields.io/badge/ansible--galaxy-elasticsearch-blue.svg)](https://galaxy.ansible.com/smraze96/elasticsearch)

A comprehensive Ansible role for deploying and managing Elasticsearch clusters on various operating systems. This role supports both Elasticsearch 8.x and 9.x versions with extensive configuration options.

## 🚀 Features

### ✅ Version Support
- **Elasticsearch 8.x** and **9.x** versions
- Dynamic version selection and automatic configuration
- JVM and performance optimizations

### ✅ Operating System Support
- **Ubuntu 20.04+**
- **Debian 10+**
- **CentOS 8+**
- **RHEL 8+**
- Automatic OS detection

### ✅ Security Features
- X-Pack Security integration
- HTTPS/TLS support
- Vault integration (for certificates and credentials)
- Self-signed certificate generation
- User management and role-based access

### ✅ Cluster Support
- Single-node and multi-node clusters
- Master/Data/Coordinating node roles
- Automatic cluster discovery
- Transport layer security

### ✅ Management Tools
- Index Lifecycle Management (ILM)
- User management
- Index template configuration
- Monitoring and logging

## 📋 Requirements

### System Requirements
- **Ansible**: 2.9 or higher
- **Python**: 3.6+
- **Memory**: Minimum 4GB RAM (8GB+ recommended for production)
- **Disk**: Minimum 20GB (additional space for data)
- **Java**: Automatically installed (OpenJDK 21)

### Network Requirements
- **Port 9200**: HTTP/HTTPS REST API
- **Port 9300**: Transport layer (cluster communication)
- **Internet**: For downloading Elasticsearch packages

## 🔧 Role Variables

### Basic Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `elasticsearch_major_version` | `9` | Major version (8 or 9) |
| `elasticsearch_minor_version` | `"0.0"` | Minor version |
| `elasticsearch_cluster_name` | `""` | Cluster name |
| `elasticsearch_security_enabled` | `no` | Enable X-Pack Security |
| `elasticsearch_https_enabled` | `no` | Enable HTTPS for REST API |

### Directory Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `elasticsearch_data_dir` | `/data/elasticsearch` | Data directory |
| `elasticsearch_log_dir` | `/var/log/elasticsearch` | Log directory |
| `elasticsearch_conf_dir` | `/etc/elasticsearch` | Configuration directory |
| `elasticsearch_certs_dir` | `/etc/elasticsearch/certs` | Certificates directory |

### JVM Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `elasticsearch_jvm_xms` | `4g` | Initial heap size |
| `elasticsearch_jvm_xmx` | `4g` | Maximum heap size |
| `elasticsearch_vm_max_map_count` | `262144` | Maximum memory map areas |

### Vault Integration

| Variable | Default | Description |
|----------|---------|-------------|
| `elasticsearch_use_vault` | `true` | Use Vault for certificates/credentials |
| `elasticsearch_vault_token` | From env | Vault token |
| `elasticsearch_vault_addr` | From env | Vault server address |
| `elasticsearch_vault_path` | From env | Vault secret path |

## 📁 Inventory Configuration

Based on your provided inventory structure, here's the recommended setup:

### Multi-Node Production Cluster

```yaml
all:
  hosts:
    elastic-coordinating01.example.local:
      ansible_host: 192.168.1.91
      elastic_ip_address: 192.168.1.91
      dns_names:
        - elastic-coordinating01.example.local
    elastic-coordinating02.example.local:
      ansible_host: 192.168.1.92
      elastic_ip_address: 192.168.1.92
      dns_names:
        - elastic-coordinating02.example.local
    elastic-master01.example.local:
      ansible_host: 192.168.1.93
      elastic_ip_address: 192.168.1.93
      dns_names:
        - elastic-master01.example.local
    elastic-master02.example.local:
      ansible_host: 192.168.1.94
      elastic_ip_address: 192.168.1.94
      dns_names:
        - elastic-master02.example.local
    elastic-master03.example.local:
      ansible_host: 192.168.1.95
      elastic_ip_address: 192.168.1.95
      dns_names:
        - elastic-master03.example.local
    elastic-data01.example.local:
      ansible_host: 192.168.1.96
      elastic_ip_address: 192.168.1.96
      dns_names:
        - elastic-data01.example.local
    elastic-data02.example.local:
      ansible_host: 192.168.1.97
      elastic_ip_address: 192.168.1.97
      dns_names:
        - elastic-data02.example.local

  children:
    elasticsearch_coordinating:
      hosts:
        elastic-coordinating01.example.local:
        elastic-coordinating02.example.local:
    elasticsearch_master:
      hosts:
        elastic-master01.example.local:
        elastic-master02.example.local:
        elastic-master03.example.local:
    elasticsearch_data:
      hosts:
        elastic-data01.example.local:
        elastic-data02.example.local:
    elasticsearch:
      children:
        elasticsearch_coordinating:
        elasticsearch_master:
        elasticsearch_data:
    kibana:
      hosts:
        elastic-master01.example.local:
    nginx:
      hosts:
        elastic-master01.example.local:
```

### Group Variables Configuration

Create `inventory/group_vars/elasticsearch/main.yml`:

```yaml
---
# Elasticsearch Configuration
elasticsearch_major_version: 9
elasticsearch_minor_version: "0.0"
elasticsearch_cluster_name: "production-elastic-cluster"

# Security Configuration
elasticsearch_security_enabled: yes
elasticsearch_https_enabled: yes

# Vault Configuration (set to false if not using Vault)
elasticsearch_use_vault: false

# Local Certificate Configuration (when not using Vault)
elasticsearch_local_cert_country: "AZ"
elasticsearch_local_cert_state: "BAKU"
elasticsearch_local_cert_locality: "BAKU"
elasticsearch_local_cert_organization: "IT"
elasticsearch_local_cert_organizational_unit: "IT Department"

# Local User Credentials (when not using Vault)
elasticsearch_local_superuser_username: "admin"
elasticsearch_local_superuser_password: "SecurePassword123!"
elasticsearch_local_developer_username: "developer"
elasticsearch_local_developer_password: "DevPassword123!"
elasticsearch_local_kibana_system_username: "kibana_system"
elasticsearch_local_kibana_system_password: "KibanaPassword123!"

# System Configuration
elasticsearch_disable_firewall: yes
elasticsearch_vm_max_map_count: "262144"
elasticsearch_vm_swappiness: "1"

# Index Management
elasticsearch_index_pattern: "logstash"
elasticsearch_index_keep_days: "30d"
```

## 🎯 Playbook Examples

### 1. Basic Elasticsearch 9 Installation

```yaml
---
- name: Deploy Elasticsearch Cluster
  hosts: elasticsearch
  become: true
  vars:
    elasticsearch_major_version: 9
    elasticsearch_cluster_name: "production-cluster"
    elasticsearch_security_enabled: yes
    elasticsearch_https_enabled: yes
    
  roles:
    - elasticsearch

  post_tasks:
    - name: Wait for Elasticsearch to be ready
      uri:
        url: "https://{{ ansible_host }}:9200/_cluster/health"
        method: GET
        validate_certs: no
        user: "{{ elasticsearch_local_superuser_username }}"
        password: "{{ elasticsearch_local_superuser_password }}"
        force_basic_auth: yes
      register: cluster_health
      until: cluster_health.json.status in ["green", "yellow"]
      retries: 30
      delay: 10
      run_once: true
```

### 2. Elasticsearch with Kibana and Nginx

```yaml
---
- name: Deploy ELK Stack
  hosts: all
  become: true
  
- name: Install Elasticsearch
  hosts: elasticsearch
  become: true
  roles:
    - elasticsearch

- name: Install Kibana
  hosts: kibana
  become: true
  vars:
    kibana_https_enabled: yes
    elasticsearch_https_enabled: yes
    elasticsearch_security_enabled: yes
  roles:
    - kibana

- name: Install Nginx
  hosts: nginx
  become: true
  vars:
    nginx_https_enabled: yes
    nginx_kibana_enabled: yes
  roles:
    - nginx
```

### 3. Development Environment (No Security)

```yaml
---
- name: Deploy Elasticsearch for Development
  hosts: elasticsearch
  become: true
  vars:
    elasticsearch_major_version: 9
    elasticsearch_cluster_name: "dev-cluster"
    elasticsearch_security_enabled: no
    elasticsearch_https_enabled: no
    elasticsearch_use_vault: no
    elasticsearch_disable_firewall: yes
    
  roles:
    - elasticsearch
```

## 🚀 Installation Guide

### Step 1: Clone the Repository

```bash
git clone https://github.com/smraze96/elasticsearch.git
cd elasticsearch
```

### Step 2: Prepare Inventory

```bash
# Copy your inventory file
cp your-inventory.yml inventory/hosts.yml

# Create group variables
mkdir -p inventory/group_vars/elasticsearch
# Edit the group variables as shown above
```

### Step 3: Configure Vault (Optional)

```bash
# Set Vault environment variables
export VAULT_ADDR="https://vault.example.com:8200"
export VAULT_TOKEN="your-vault-token"
export VAULT_PATH="secret/elasticsearch"

# Store secrets in Vault
vault kv put secret/elasticsearch \
    CI_ELASTICSEARCH_ADMIN="your_admin_password" \
    CI_ELASTICSEARCH_DEVELOPER="your_dev_password" \
    CI_KIBANA_SYSTEM="your_kibana_password"
```

### Step 4: Run the Playbook

```bash
# Syntax check
ansible-playbook -i inventory/hosts.yml main.yml --syntax-check

# Dry run
ansible-playbook -i inventory/hosts.yml main.yml --check

# Deploy
ansible-playbook -i inventory/hosts.yml main.yml

# With specific tags
ansible-playbook -i inventory/hosts.yml main.yml --tags="elasticsearch,install"
```

### Step 5: Verify Installation

```bash
# Check cluster health
curl -k -u admin:SecurePassword123! https://192.168.1.93:9200/_cluster/health?pretty

# Check nodes
curl -k -u admin:SecurePassword123! https://192.168.1.93:9200/_nodes?pretty

# Check indices
curl -k -u admin:SecurePassword123! https://192.168.1.93:9200/_cat/indices?v
```

## 🏗 Node Architecture

Based on your inventory, this role will configure:

### Master Nodes (3 nodes)
- **elastic-master01.example.local** (192.168.1.93) - Also runs Kibana & Nginx
- **elastic-master02.example.local** (192.168.1.94)
- **elastic-master03.example.local** (192.168.1.95)

**Role**: Cluster management, metadata operations, node discovery

### Data Nodes (2 nodes)
- **elastic-data01.example.local** (192.168.1.96)
- **elastic-data02.example.local** (192.168.1.97)

**Role**: Store data, perform data-related operations, indexing, searching

### Coordinating Nodes (2 nodes)
- **elastic-coordinating01.example.local** (192.168.1.91)
- **elastic-coordinating02.example.local** (192.168.1.92)

**Role**: Route requests, aggregate results, handle client connections

## 🔐 Security Configuration

### User Management

The role automatically creates these users when security is enabled:

| Username | Role | Purpose |
|----------|------|---------|
| `admin` | superuser | Full cluster access |
| `developer` | editor | Development access |
| `kibana_system` | kibana_system | Kibana integration |

### Certificate Management

- **Transport certificates**: Automatically generated for inter-node communication
- **HTTP certificates**: Generated per node for HTTPS API access
- **Vault integration**: Optional for enterprise certificate management

## 🛠 Troubleshooting

### Common Issues

#### 1. Java Version Problems
```bash
# Check Java version
java -version

# Set JAVA_HOME
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
```

#### 2. Memory Issues
```bash
# Check JVM settings
grep -E "^-Xm[sx]" /etc/elasticsearch/jvm.options

# Check system memory
free -h
```

#### 3. Cluster Formation Issues
```bash
# Check cluster health
curl -k -u admin:password https://localhost:9200/_cluster/health?pretty

# Check node discovery
curl -k -u admin:password https://localhost:9200/_cat/nodes?v
```

### Log Files

```bash
# Service logs
journalctl -u elasticsearch -f

# Application logs
tail -f /var/log/elasticsearch/production-elastic-cluster.log

# Check cluster logs on all nodes
for node in 192.168.1.{91..97}; do
  echo "=== Node $node ==="
  ssh root@$node "tail -5 /var/log/elasticsearch/*.log"
done
```

## 📊 Monitoring

### Health Check Script

```bash
#!/bin/bash
# cluster-health.sh
ELASTICSEARCH_URL="https://192.168.1.93:9200"
USERNAME="admin"
PASSWORD="SecurePassword123!"

HEALTH=$(curl -s -k -u $USERNAME:$PASSWORD "$ELASTICSEARCH_URL/_cluster/health" | jq -r '.status')

case $HEALTH in
    "green")
        echo "✅ Cluster is healthy"
        exit 0
        ;;
    "yellow")
        echo "⚠️ Cluster has some issues but is operational"
        exit 1
        ;;
    "red")
        echo "❌ Cluster is in critical state"
        exit 2
        ;;
    *)
        echo "❓ Unable to determine cluster health"
        exit 3
        ;;
esac
```

### Performance Monitoring

```bash
# Node stats
curl -k -u admin:password https://192.168.1.93:9200/_nodes/stats?pretty

# Cluster stats
curl -k -u admin:password https://192.168.1.93:9200/_cluster/stats?pretty

# Index stats
curl -k -u admin:password https://192.168.1.93:9200/_stats?pretty
```

## 🔄 Maintenance

### Rolling Restart

```bash
# Disable shard allocation
curl -k -u admin:password -X PUT "https://192.168.1.93:9200/_cluster/settings" \
  -H 'Content-Type: application/json' -d '{
    "persistent": {
      "cluster.routing.allocation.enable": "primaries"
    }
  }'

# Restart each node (start with data nodes)
systemctl restart elasticsearch

# Re-enable shard allocation
curl -k -u admin:password -X PUT "https://192.168.1.93:9200/_cluster/settings" \
  -H 'Content-Type: application/json' -d '{
    "persistent": {
      "cluster.routing.allocation.enable": null
    }
  }'
```

### Index Management

```bash
# Create index template
curl -k -u admin:password -X PUT "https://192.168.1.93:9200/_index_template/production-logs" \
  -H 'Content-Type: application/json' -d '{
    "index_patterns": ["logs-*"],
    "template": {
      "settings": {
        "number_of_shards": 2,
        "number_of_replicas": 1,
        "index.lifecycle.name": "delete_old_logs"
      }
    }
  }'
```

## 📚 Advanced Configuration

### Custom JVM Settings per Node Type

```yaml
# In inventory host variables
elasticsearch_master:
  hosts:
    elastic-master01.example.local:
      elasticsearch_jvm_xms: 4g
      elasticsearch_jvm_xmx: 4g
      # Master nodes need less memory

elasticsearch_data:
  hosts:
    elastic-data01.example.local:
      elasticsearch_jvm_xms: 8g
      elasticsearch_jvm_xmx: 8g
      # Data nodes need more memory

elasticsearch_coordinating:
  hosts:
    elastic-coordinating01.example.local:
      elasticsearch_jvm_xms: 6g
      elasticsearch_jvm_xmx: 6g
      # Coordinating nodes need moderate memory
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🆘 Support

- **GitHub Issues**: [https://github.com/smraze96/elasticsearch/issues](https://github.com/smraze96/elasticsearch/issues)
- **Documentation**: [https://github.com/smraze96/elasticsearch](https://github.com/smraze96/elasticsearch)

## 🏷 Tags

`ansible` `elasticsearch` `elk-stack` `devops` `automation` `infrastructure` `monitoring` `logging`
