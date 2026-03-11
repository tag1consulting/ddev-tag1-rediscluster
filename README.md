## DDEV Redis Cluster with Envoy Proxy

## Overview

[Redis](https://redis.io/) is an in-memory key–value database, used as a distributed cache and message broker, with optional durability.

This add-on integrates a **3-node Redis Cluster** into your [DDEV](https://ddev.com/) project with an **Envoy proxy** frontend for load balancing and connection pooling. The cluster provides high availability and horizontal scaling capabilities, with Redis persistence enabled by default.

### Architecture

- **3 Redis nodes** running in cluster mode (no replicas)
  - `redis-node-1:7001`
  - `redis-node-2:7002`
  - `redis-node-3:7003`
- **Envoy proxy** frontend for connection pooling and load balancing
  - Proxy endpoint: `envoy:10000`
  - Admin UI: `envoy:9901`

## Installation

```bash
ddev add-on get tag1consulting/ddev-tag1-rediscluster
ddev restart
```

After installation, make sure to commit the `.ddev` directory to version control.

## Usage

| Command | Description |
| ------- | ----------- |
| `ddev redis-cli` | Run `redis-cli` inside the Redis cluster (connects to node-1 with cluster mode) |
| `ddev redis` | Alias for `ddev redis-cli` |
| `ddev redis-flush` | Flush all cache inside the Redis cluster (flushes all 3 nodes) |
| `ddev describe` | View service status and used ports for Redis cluster and Envoy |
| `ddev logs -s redis-node-1` | Check Redis node 1 logs |
| `ddev logs -s redis-node-2` | Check Redis node 2 logs |
| `ddev logs -s redis-node-3` | Check Redis node 3 logs |
| `ddev logs -s envoy` | Check Envoy proxy logs |

### Connection Information

Applications should connect to Redis via the **Envoy proxy** for load balancing:

**Recommended (via Envoy proxy):**
- Host: `envoy`
- Port: `10000`

**Direct node access** (for cluster management or debugging):
- Node 1: `redis-node-1:7001`
- Node 2: `redis-node-2:7002`
- Node 3: `redis-node-3:7003`

**Envoy admin interface:**
- URL: `http://envoy:9901`

## Redis Cluster Management

Check cluster status:

```bash
ddev redis-cli CLUSTER INFO
```

View cluster nodes:

```bash
ddev redis-cli CLUSTER NODES
```

Get cluster slots distribution:

```bash
ddev redis-cli CLUSTER SLOTS
```

## Redis Credentials

By default, there is no authentication. The cluster runs with `protected-mode no` to allow inter-node communication.

> [!WARNING]
> For production deployments, you should enable Redis authentication by configuring ACLs in the cluster configuration.

For more information about ACLs, see the [Redis documentation](https://redis.io/docs/latest/operate/oss_and_stack/management/security/acl/).

## Configuration Files

- `redis/redis.conf` - Standard Redis configuration (for reference, not used by cluster)
- `redis/redis-cluster.conf` - Cluster-specific configuration used by all nodes
- `docker-compose.redis.yaml` - Redis cluster service definitions
- `docker-compose.envoy.yaml` - Envoy proxy service definition
- `envoy/envoy.yaml` - Envoy proxy configuration (to be created)

## Customization

To change the Redis Docker image used for all cluster nodes:

```bash
ddev dotenv set .ddev/.env.redis --redis-docker-image=redis:7-alpine
ddev restart
```

To change the Envoy Docker image:

```bash
ddev dotenv set .ddev/.env.redis --envoy-docker-image=envoyproxy/envoy:v1.37-latest
ddev restart
```

To change the Envoy hostname:

```bash
ddev dotenv set .ddev/.env.redis --envoy-hostname=envoy
ddev restart
```

Make sure to commit the `.ddev/.env.redis` file to version control.

### Available Environment Variables

| Variable | Flag | Default |
| -------- | ---- | ------- |
| `REDIS_DOCKER_IMAGE` | `--redis-docker-image` | `redis:7` |
| `ENVOY_DOCKER_IMAGE` | `--envoy-docker-image` | `envoyproxy/envoy:v1.37-latest` |
| `ENVOY_HOSTNAME` | `--envoy-hostname` | `envoy` |

## Persistence and Data

Each Redis node maintains its own data volume:
- `redis-node-1` volume
- `redis-node-2` volume
- `redis-node-3` volume

The cluster uses append-only file (AOF) persistence by default.

To clear all cluster data:

```bash
ddev stop
docker volume rm ddev-$(ddev status -j | docker run -i --rm ddev/ddev-utilities jq -r '.raw.name')_redis-node-1
docker volume rm ddev-$(ddev status -j | docker run -i --rm ddev/ddev-utilities jq -r '.raw.name')_redis-node-2
docker volume rm ddev-$(ddev status -j | docker run -i --rm ddev/ddev-utilities jq -r '.raw.name')_redis-node-3
ddev restart
```

## Credits

**Based on the original [ddev-contrib recipe](https://github.com/ddev/ddev-contrib/tree/master/docker-compose-services/redis) by [@gormus](https://github.com/gormus)**

**Redis cluster implementation by Tag1 Consulting**

**Original DDEV Redis add-on contributed by [@hussainweb](https://github.com/hussainweb)**
