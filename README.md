# MySQL Database Service

**English** | [中文](README_zh.md)

Enterprise-grade MySQL database service for Kubernetes with high availability, read-write splitting, and automatic failover.

## Overview

MySQL is one of the world's most popular open-source relational database management systems, known for its high performance, reliability, and ease of use. This package delivers a production-ready MySQL cluster on Kubernetes, featuring primary-replica replication, intelligent read-write splitting via ProxySQL, automatic failover, and integrated monitoring.

## Features

### Core Capabilities
- **High availability**: Primary-replica replication with automatic failover and recovery
- **Read-write splitting**: Intelligent query routing through ProxySQL
- **Durable storage**: Kubernetes persistent volumes with backup and restore support
- **Monitoring and alerting**: Integrated Prometheus metrics and alert rules
- **Auto scaling**: Dynamically adjust the instance count
- **Backup and restore**: Physical backups (XtraBackup) and logical backups

### Advanced Features
- **Semi-synchronous replication**: Ensures data consistency and high availability
- **Connection pool management**: Optimized database connections via ProxySQL
- **Slow query analysis**: Built-in slow query logging and performance profiling
- **Audit logging**: Database operation auditing for compliance requirements
- **Resource management**: Flexible CPU and memory sizing
- **Network isolation**: Supports host-network and Pod-network modes
- **Time zone control**: Defaults to the Asia/Shanghai time zone

### Enterprise Features
- **Thread pool**: Optimized concurrent connection handling
- **Buffer pool control**: Dynamic InnoDB buffer pool sizing
- **Recycle bin**: Protection against accidental data deletion
- **Performance tuning**: Built-in performance optimization parameters

## Supported Versions

### MySQL Releases
- **8.4.3** (latest, recommended)
- **8.0.39** (stable)
- **8.0.45**

### Component Releases
- **MySQL Operator**: v1.13.0
- **MySQL Init**: v80-2.0.0 / v57-2.0.0
- **XtraBackup**: v8.0-4 / v1.2.0-hc.8
- **MySQL Exporter**: v0.12.1-1.0.0
- **ProxySQL**: 2.5.2-1.0.0
- **Logrotate**: 3.21.0-1.0.0

## Architecture

### Deployment Modes

#### 1. Primary-Replica (masterslave)
- **Use cases**: Production workloads, applications that need read-write separation
- **Traits**: Primary-replica replication with automatic failover
- **Topology**: 1 primary + N replicas, configurable count
- **Sync mode**: Semi-synchronous replication for data consistency

#### 2. Proxy (proxy)
- **Use cases**: High-concurrency applications, scenarios requiring intelligent routing
- **Traits**: Integrated ProxySQL for read-write splitting and connection pooling
- **Topology**: MySQL cluster + ProxySQL proxy layer
- **Advantages**: Automatic failover, query routing, connection multiplexing

#### 3. Highly Available (operator-highly-available)
- **Use cases**: Mission-critical systems with strict uptime targets
- **Traits**: Multi-instance deployment, automatic failover, strong data consistency
- **Topology**: 3+ instances with synchronous replication and auto-recovery

#### 4. Standard (operator-standard)
- **Use cases**: Development, testing, and quick deployment
- **Traits**: Minimal resources, simple single-instance setup

### Technical Architecture

```
+---------------------------------------------------------+
|                    MySQL Cluster                        |
+---------------------------------------------------------+
|  +-----------+  +-----------+  +-----------+            |
|  |  Master   |  |  Slave    |  |  Slave    |            |
|  | (Primary) |  | (Replica) |  | (Replica) |            |
|  +-----------+  +-----------+  +-----------+            |
+---------------------------------------------------------+
|                    ProxySQL Layer                        |
|  +-----------+  +-----------+  +-----------+            |
|  | ProxySQL  |  | ProxySQL  |  | ProxySQL  |            |
|  | Instance  |  | Instance  |  | Instance  |            |
|  +-----------+  +-----------+  +-----------+            |
+---------------------------------------------------------+
|  +-----------+  +-----------+  +-----------+            |
|  |  Service  |  | ConfigMap |  |  Secret   |            |
|  |(Endpoints)|  |  (Config) |  |(Passwords)|            |
|  +-----------+  +-----------+  +-----------+            |
+---------------------------------------------------------+
|                 Kubernetes Storage (PVC)                 |
+---------------------------------------------------------+
```

### Data Flow

```
Application -> ProxySQL -> MySQL Master (Write)
                      |
                MySQL Slaves (Read)
                      |
                Backup & Monitoring
```

### Component Overview

- **MySQL Server**: Core database engine
- **MySQL Operator**: Cluster management controller
- **ProxySQL**: High-performance proxy providing connection pooling and query routing
- **XtraBackup**: Physical backup tool
- **MySQL Exporter**: Prometheus metrics collector
- **Logrotate**: Log rotation management

## Prerequisites

- Kubernetes 1.26+
- [OpenSaola Operator](https://github.com/harmonycloud/opensaola) deployed
- [saola-cli](https://github.com/harmonycloud/saola-cli) installed

## Quick Start

```bash
# Publish the package
saola publish mysql/

# Install the operator
saola operator create mysql-operator --type MySQL --version 8.4.3

# Create an instance
saola middleware create my-mysql --type MySQL --version 8.4.3

# Check status
saola middleware get my-mysql
```

## Available Actions

| Action | Description |
|--------|-------------|
| restart | Restart the middleware instance |
| failover | Trigger manual failover |
| migrate | Migrate nodes to different Kubernetes nodes |
| passive-switch | Perform a passive primary-replica switchover |
| datasecurity | Manage data security settings |
| setParameters | Modify runtime configuration parameters |

## Configuration

Key parameters can be customized via the baseline configuration. See `manifests/*parameters.yaml` for the full parameter reference.

### Resource Planning

```yaml
# Recommended production settings
resources:
  mysql:
    limits:
      cpu: "4"        # 4 cores
      memory: "8Gi"   # 8 GB
    requests:
      cpu: "2"        # 2 cores
      memory: "4Gi"   # 4 GB
  proxy:
    limits:
      cpu: "2"        # 2 cores
      memory: "4Gi"   # 4 GB
    requests:
      cpu: "1"        # 1 core
      memory: "2Gi"   # 2 GB

# Storage profile
volume:
  size: 200           # 200 GB
  storageClass: "fast-ssd"  # SSD-backed class
```

### High Availability Settings

```yaml
# Primary-replica configuration
syncMode: semi-sync   # Semi-synchronous replication
replicas: 3          # 3 instances

# ProxySQL configuration
proxy:
  enable: true
  replicaCount: 3    # 3 proxy instances
  mysql-max_connections: 2048
  mysql-threads: 2
```

### Monitoring Profile

```yaml
# Monitoring and alerting
monitor:
  enableAlert: true         # Turn on alerts
  enableExporter: true      # Enable metrics exporter
  slowqueryLogging: true    # Enable slow query logging
  auditLogging: true        # Enable audit logging
```

## Usage Guidance

### Environment Selection

#### Development and Test
- **Recommended topology**: Standard (operator-standard)
- **Resources**: CPU 2 cores, memory 4 Gi, storage 50 Gi
- **Suggested version**: MySQL 8.0.39
- **Instances**: 2 (1 primary + 1 replica)

#### Production
- **Recommended topology**: Proxy or highly available mode
- **Resources**: CPU 4+ cores, memory 8+ Gi, storage 200+ Gi
- **Suggested version**: MySQL 8.4.3 or 8.0.39
- **Instances**: 3+ for high availability

### Best Practices

#### Security
- Enforce strong passwords with mixed character classes
- Enable SSL connection encryption
- Rotate database credentials periodically
- Configure appropriate access control rules
- Enable audit logging for sensitive operations

#### Performance Tuning
- Adjust `innodb_buffer_pool_size` based on workload (recommend 70% of total memory)
- Configure appropriate `max_connections` and timeout parameters
- Enable slow query logging for performance analysis
- Use ProxySQL for connection pool management
- Optimize InnoDB-related parameters

#### Backup Strategy
- Enable XtraBackup for physical backups
- Schedule recurring logical backups
- Regularly test restore procedures
- Store backups in secure remote locations
- Define backup retention policies

#### Monitoring and Alerting
- Track connection count, query performance, and disk usage
- Define alert thresholds for critical metrics
- Review slow query logs routinely
- Monitor primary-replica replication lag
- Watch ProxySQL connection pool status

## Related Projects

| Project | Description |
|---------|-------------|
| [OpenSaola Operator](https://github.com/harmonycloud/opensaola) | Core Kubernetes operator for middleware lifecycle management |
| [saola-cli](https://github.com/harmonycloud/saola-cli) | Command-line tool for middleware management |
| [PostgreSQL](https://github.com/harmonycloud/postgresql) | PostgreSQL database package |
| [Kafka](https://github.com/harmonycloud/kafka) | Apache Kafka streaming platform package |
| [Redis](https://github.com/harmonycloud/redis) | Redis in-memory data store package |
| [Elasticsearch](https://github.com/harmonycloud/elasticsearch) | Elasticsearch search engine package |
| [ZooKeeper](https://github.com/harmonycloud/zookeeper) | Apache ZooKeeper coordination service package |
| [RabbitMQ](https://github.com/harmonycloud/rabbitmq) | RabbitMQ message broker package |

## License

This project is licensed under the [Apache License 2.0](LICENSE).
