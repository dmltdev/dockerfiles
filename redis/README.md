# Redis Docker Configuration

This directory contains a production-ready Redis setup with comprehensive security, persistence, and performance configurations.

## Files Overview

- `Dockerfile` - Production-ready Redis container with security best practices
- `redis.conf` - Comprehensive Redis configuration file
- `docker-compose.yml` - Docker Compose setup with environment variables
- `.env.example` - Environment variables template

## Redis Configuration Explained

### Security Configuration

```conf
protected-mode yes
port 6379
bind 0.0.0.0
```

- **protected-mode**: Enables Redis protected mode for security
- **port**: Standard Redis port (6379)
- **bind**: Allows connections from any IP (use with caution in production)

### Authentication

```conf
# Authentication via environment variables
# requirepass and user directives are set dynamically via REDIS_PASSWORD
```

- Passwords are set via environment variables (not hardcoded)
- Uses Redis 6+ ACL system for user management

### Disabled Commands (Security)

```conf
rename-command FLUSHDB ""
rename-command FLUSHALL ""
rename-command DEBUG ""
rename-command CONFIG ""
rename-command SHUTDOWN SHUTDOWN_REDIS
rename-command EVAL ""
```

- **FLUSHDB/FLUSHALL**: Prevents accidental data deletion
- **DEBUG/CONFIG**: Prevents configuration tampering
- **EVAL**: Prevents Lua script execution (security risk)
- **SHUTDOWN**: Renamed to prevent unauthorized shutdowns

### Memory Management

```conf
maxmemory 256mb
maxmemory-policy allkeys-lru
maxmemory-samples 5
```

- **maxmemory**: Limits Redis memory usage to 256MB (configurable via env var)
- **maxmemory-policy**: Uses LRU (Least Recently Used) eviction
- **maxmemory-samples**: Number of keys to sample for LRU algorithm

### Persistence Configuration

#### RDB Snapshots

```conf
save 900 1     # Save if ≥1 key changed in 15 minutes
save 300 10    # Save if ≥10 keys changed in 5 minutes
save 60 10000  # Save if ≥10,000 keys changed in 1 minute
```

- Creates point-in-time snapshots based on activity level
- Saves to `/data/dump.rdb` file
- **rdbcompression**: Compresses RDB files to save space
- **rdbchecksum**: Adds checksums for data integrity

#### AOF (Append Only File)

```conf
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
```

- **appendonly**: Enables AOF for better durability
- **appendfsync everysec**: Syncs to disk every second (balance of performance/safety)
- **auto-aof-rewrite**: Automatically rewrites AOF when it grows too large

### Logging

```conf
loglevel notice
logfile /var/log/redis/redis-server.log
syslog-enabled yes
```

- **loglevel notice**: Standard logging level for production
- **logfile**: Writes logs to persistent volume
- **syslog**: Also sends logs to system logger

### Network Settings

```conf
timeout 300
tcp-keepalive 300
tcp-backlog 511
```

- **timeout**: Client idle timeout (5 minutes)
- **tcp-keepalive**: TCP keepalive interval
- **tcp-backlog**: TCP listen backlog size

### Performance Tuning

```conf
databases 16
hash-max-ziplist-entries 512
list-max-ziplist-size -2
set-max-intset-entries 512
```

- **databases**: Number of Redis databases (0-15)
- **ziplist settings**: Memory optimization for small collections
- **intset settings**: Memory optimization for integer sets

### Monitoring

```conf
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 100
```

- **slowlog**: Logs queries slower than 10ms
- **latency-monitor**: Tracks operations with latency >100μs

### Client Buffer Limits

```conf
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
```

- Prevents clients from consuming too much memory
- Different limits for normal clients, replicas, and pub/sub

## Docker Setup

### Environment Variables

Create a `.env` file from `.env.example`:

```bash
cp .env.example .env
```

Configure these variables:

- **REDIS_PASSWORD**: Strong password for Redis authentication
- **REDIS_MAXMEMORY**: Memory limit (default: 256mb)

### Building and Running

```bash
# Build and start
docker-compose up -d

# View logs
docker-compose logs redis

# Connect to Redis CLI
docker-compose exec redis redis-cli -a your_password

# Stop
docker-compose down
```

### Data Persistence

Data is persisted in Docker volumes:

- **redis_data**: Contains RDB snapshots and AOF files (`/data`)
- **redis_logs**: Contains Redis log files (`/var/log/redis`)

### Security Features

- Runs as non-root `redis` user
- Uses `su-exec` for secure privilege dropping
- File permissions set to 640 (read-write for owner, read for group)
- Health checks with `redis-cli ping`
- Resource limits in docker-compose

## Usage Examples

### Basic Connection

```bash
# Connect with password
redis-cli -h localhost -p 6379 -a your_password

# Test connection
redis-cli -h localhost -p 6379 -a your_password ping
```

### Monitoring

```bash
# View slow queries
redis-cli -a your_password slowlog get 10

# Monitor commands in real-time
redis-cli -a your_password monitor

# Check memory usage
redis-cli -a your_password info memory
```

### Backup and Restore

```bash
# Manual snapshot
redis-cli -a your_password bgsave

# Copy RDB file for backup
docker cp redis-server:/data/dump.rdb ./backup-$(date +%Y%m%d).rdb
```

## Security Recommendations

1. **Change default password**: Use a strong, unique password
2. **Network security**: Use firewall rules to restrict access
3. **TLS encryption**: Consider enabling TLS for encrypted connections
4. **Regular backups**: Implement automated backup strategy
5. **Monitor logs**: Set up log monitoring and alerting
6. **Update regularly**: Keep Redis version updated

## Troubleshooting

### Common Issues

1. **Permission denied**: Check file ownership and permissions
2. **Out of memory**: Adjust `REDIS_MAXMEMORY` or `maxmemory-policy`
3. **Connection refused**: Verify port mapping and firewall settings
4. **Slow performance**: Check slow log and adjust configuration

### Useful Commands

```bash
# Check Redis info
docker-compose exec redis redis-cli -a password info

# View configuration
docker-compose exec redis redis-cli -a password config get "*"

# Monitor memory usage
docker-compose exec redis redis-cli -a password info memory
```

This configuration provides a robust, secure, and performant Redis setup suitable for production environments.
