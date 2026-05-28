# EKEX Intelligence — Deployment Guide

**Version:** 1.0  
**Date:** 2026-05-28  
**Author:** Kimuntu Group  
**Status:** Reference Implementation

---

## Table of Contents

1. [Deployment Overview](#deployment-overview)
2. [Local Development Setup](#local-development-setup)
3. [Staging Environment](#staging-environment)
4. [Production Environment](#production-environment)
5. [Environment Configuration](#environment-configuration)
6. [Database Migrations](#database-migrations)
7. [Health Checks & Monitoring](#health-checks--monitoring)

---

## Deployment Overview

EKEX Intelligence supports three deployment environments, each with specific infrastructure requirements:

| Environment | Purpose | Scale | Data | SSL/TLS |
|-------------|---------|-------|------|---------|
| **Local (Docker Compose)** | Development & testing | Single machine | Sample data | No |
| **Staging** | Pre-production testing | 2-3 app servers | Real-like data | Yes |
| **Production** | Live platform | 3+ app servers + workers | Production data | Yes (mandatory) |

---

## Local Development Setup

### Quick Start (Docker Compose)

For developers who want to run EKEX locally with minimal setup:

```bash
# Clone repository
git clone https://github.com/kimuntugroup-commits/EKEXINTELLIGENCE.git
cd EKEXINTELLIGENCE

# Copy environment template
cp .env.example .env.local

# Edit environment file (optional — defaults work for local dev)
# nano .env.local

# Start all services (app, database, cache, bot)
docker-compose -f docker-compose.example.yml up -d

# Install dependencies
npm install

# Run database migrations
npm run db:migrate

# Seed sample data
npm run db:seed

# Start development server
npm run dev
```

### Services Started

```
┌────────────────────────────────────────────┐
│      LOCAL DEVELOPMENT ENVIRONMENT         │
├────────────────────────────────────────────┤
│                                            │
│  🟢 Node.js API Server                     │
│     Port: 3000                             │
│     URL: http://localhost:3000             │
│     Auto-reload on code changes            │
│                                            │
│  🗄️ PostgreSQL Database                    │
│     Port: 5432                             │
│     Database: ekex_intelligence            │
│     Credentials: root/password             │
│                                            │
│  📦 Redis Cache                            │
│     Port: 6379                             │
│     Pub/Sub enabled                        │
│                                            │
│  🤖 Telegram Bot (NEZA)                    │
│     Port: 3001                             │
│     Token: Set via TELEGRAM_BOT_TOKEN      │
│                                            │
└────────────────────────────────────────────┘
```

### Access Points

| Service | URL | Purpose |
|---------|-----|---------|
| Consumer Portal | `http://localhost:3000` | Web interface |
| Admin Dashboard | `http://localhost:3000/admin` | Management UI |
| GraphQL Playground | `http://localhost:3000/graphql` | Query testing |
| PostgreSQL | `localhost:5432` | Database |
| Redis CLI | `redis-cli -p 6379` | Cache inspection |

### Stopping Services

```bash
# Stop all services (keeps volumes/data)
docker-compose -f docker-compose.example.yml stop

# Remove containers (keeps volumes/data)
docker-compose -f docker-compose.example.yml down

# Full reset (removes all data)
docker-compose -f docker-compose.example.yml down -v
```

---

## Staging Environment

### Infrastructure Setup

Staging runs on AWS-like infrastructure with persistent storage:

```
┌─────────────────────────────────────────────────────┐
│              STAGING CLUSTER (AWS)                  │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Application Tier                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────┐  │
│  │ App Server 1 │  │ App Server 2 │  │  Worker  │  │
│  │ (t3.small)   │  │ (t3.small)   │  │ (t3.sm)  │  │
│  └──────────────┘  └──────────────┘  └──────────┘  │
│         │                 │                 │       │
│         └─────────────────┼─────────────────┘       │
│                           │                         │
│         ┌─────────────────┴─────────────────┐       │
│         │   Target Group (ALB)              │       │
│         │   Health check: /health           │       │
│         └─────────────────┬─────────────────┘       │
│                           │                         │
│  Data Tier                                          │
│  ┌──────────────────────────────────────────┐       │
│  │ RDS PostgreSQL (db.t3.micro)             │       │
│  │ • Multi-AZ (automated failover)          │       │
│  │ • Automated backups (7-day retention)    │       │
│  │ • Enhanced monitoring enabled            │       │
│  └──────────────────────────────────────────┘       │
│                                                     │
│  Cache Tier                                         │
│  ┌──────────────────────────────────────────┐       │
│  │ ElastiCache Redis (cache.t3.micro)       │       │
│  │ • Parameter group: default.redis7        │       │
│  │ • Replication: single-AZ                 │       │
│  │ • Automatic failover: disabled           │       │
│  └──────────────────────────────────────────┘       │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Deployment Steps

#### 1. Create RDS Database

```bash
# Create RDS instance
aws rds create-db-instance \
  --db-instance-identifier ekex-staging \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --engine-version 14.7 \
  --master-username ekexadmin \
  --master-user-password $(openssl rand -base64 32) \
  --allocated-storage 100 \
  --storage-type gp3 \
  --multi-az \
  --backup-retention-period 7 \
  --publicly-accessible false \
  --vpc-security-group-ids sg-xxxxx
```

#### 2. Create ElastiCache Redis

```bash
aws elasticache create-cache-cluster \
  --cache-cluster-id ekex-staging-redis \
  --cache-node-type cache.t3.micro \
  --engine redis \
  --engine-version 7.0 \
  --num-cache-nodes 1 \
  --parameter-group-name default.redis7 \
  --security-group-ids sg-xxxxx
```

#### 3. Deploy Application

```bash
# Build Docker image
docker build -t ekex-intelligence:staging .

# Push to ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $(aws sts get-caller-identity --query Account --output text).dkr.ecr.us-east-1.amazonaws.com

docker tag ekex-intelligence:staging $(aws sts get-caller-identity --query Account --output text).dkr.ecr.us-east-1.amazonaws.com/ekex-intelligence:staging

docker push $(aws sts get-caller-identity --query Account --output text).dkr.ecr.us-east-1.amazonaws.com/ekex-intelligence:staging

# Deploy via ECS
aws ecs update-service \
  --cluster ekex-staging \
  --service ekex-app \
  --force-new-deployment
```

#### 4. Run Database Migrations

```bash
# SSH into app server
ssh ec2-user@app-server-ip

# Run migrations
npm run db:migrate -- --environment staging
```

### Environment Variables (Staging)

Create `.env.staging`:

```env
NODE_ENV=staging
APP_PORT=3000
LOG_LEVEL=info

# Database
DATABASE_URL=postgresql://ekexadmin:PASSWORD@ekex-staging.rds.amazonaws.com:5432/ekex_intelligence
DATABASE_POOL_SIZE=20

# Cache
REDIS_URL=redis://ekex-staging-redis.xxxxx.ng.0001.use1.cache.amazonaws.com:6379

# Auth & Security
JWT_SECRET=<generate-with-openssl-rand-base64-32>
JWT_EXPIRY=15m
JWT_REFRESH_EXPIRY=7d

# Telegram Bot
TELEGRAM_BOT_TOKEN=<bot-token>
TELEGRAM_WEBHOOK_URL=https://staging.ekexintelligence.com/webhook/telegram

# Monitoring
SENTRY_DSN=https://xxxxx@sentry.io/yyyyy
LOG_SERVICE=datadog

# Feature Flags
FEATURE_MATCHING_ENGINE=false  # Enable after Layer 3 fixes
FEATURE_GAP_DETECTION=false    # Enable after Layer 4 implementation
```

---

## Production Environment

### Architecture

Production uses a highly available, distributed architecture:

```
┌──────────────────────────────────────────────────────────────┐
│                   PRODUCTION ARCHITECTURE                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  CDN / Caching Layer                                         │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ CloudFlare / AWS CloudFront                            │  │
│  │ • DDoS protection (default)                            │  │
│  │ • Static asset caching (1hr TTL)                       │  │
│  │ • Global edge locations                                │  │
│  └────────────────────────────────────────────────────────┘  │
│           │                                                   │
│  WAF & Rate Limiting                                         │
│  ┌────────┴─────────────────────────────────────────────────┐ │
│  │ AWS WAF / ModSecurity                                    │ │
│  │ • Bot detection                                          │ │
│  │ • Rate limiting: 100 req/min per IP                      │ │
│  │ • SQL injection & XSS prevention                         │ │
│  └────────────────────────────────────────────────────────┘  │
│           │                                                   │
│  Load Balancer                                               │
│  ┌────────┴─────────────────────────────────────────────────┐ │
│  │ AWS ALB / NLB                                            │ │
│  │ • SSL/TLS termination (TLS 1.3)                          │ │
│  │ • Path-based routing                                     │ │
│  │ • Health check interval: 30s                             │ │
│  └────────────────────────────────────────────────────────┘  │
│           │                                                   │
│  Application Tier (Auto-Scaling)                             │
│  ┌────────┴──────────────────────────────────────────────────┐│
│  │ ECS / Kubernetes Cluster                                  ││
│  │                                                           ││
│  │ ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ││
│  │ │ App 1    │  │ App 2    │  │ App 3    │  │ App N    │  ││
│  │ │ (c5.xl)  │  │ (c5.xl)  │  │ (c5.xl)  │  │ (scaled) │  ││
│  │ └──────────┘  └──────────┘  └──────────┘  └──────────┘  ││
│  │                                                           ││
│  │ ┌──────────────────────────────────────────────────┐    ││
│  │ │ Worker Pool (Background Jobs)                    │    ││
│  │ │ • Signal processing                              │    ││
│  │ │ • Matching engine                                │    ││
│  │ │ • Gap detection scheduler                        │    ││
│  │ │ • Email/notification queue                       │    ││
│  │ └──────────────────────────────────────────────────┘    ││
│  └────────────────────────────────────────────────────────┘ │
│           │                                                   │
│  Data Tier                                                   │
│  ┌────────┴────────────────────────────────────────────────┐ │
│  │ RDS PostgreSQL Multi-AZ (db.r5.xlarge)                 │ │
│  │ ┌──────────────────────────────────────────────────┐   │ │
│  │ │ Primary (us-east-1a)                             │   │ │
│  │ │ • Read/Write                                     │   │ │
│  │ │ • Automated daily backups (30-day retention)     │   │ │
│  │ │ • Encryption at rest (AES-256)                   │   │ │
│  │ └──────────────────────────────────────────────────┘   │ │
│  │           │                                             │ │
│  │ ┌─────────┴──────────┐   ┌────────────────────────────┐ │ │
│  │ │ Replica 1 (us-a-b) │   │ Replica 2 (us-east-1c)     │ │ │
│  │ │ • Read-only        │   │ • Read-only                │ │ │
│  │ │ • Automatic failover   │ • Analytics queries        │ │ │
│  │ └────────────────────┘   └────────────────────────────┘ │ │
│  └─────────────────────────────────────────────────────────┘ │
│           │                                                   │
│  Cache Tier                                                  │
│  ┌────────┴────────────────────────────────────────────────┐ │
│  │ ElastiCache Redis Cluster (cache.r5.large × 3 nodes)  │ │
│  │ • Automatic failover enabled                            │ │
│  │ • Multi-AZ deployment                                   │ │
│  │ • Encryption in transit (TLS)                           │ │
│  │ • Encryption at rest enabled                            │ │
│  │ • Automated backups to S3                               │ │
│  └────────────────────────────────────────────────────────┘ │
│           │                                                   │
│  Storage & Logging                                           │
│  ┌────────┴────────────────────────────────────────────────┐ │
│  │ S3 (File uploads, exports, backups)                    │ │
│  │ CloudWatch Logs (Application logging)                  │ │
│  │ Datadog / New Relic (APM & monitoring)                 │ │
│  │ PagerDuty (Incident alerting)                          │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Production Checklist

Before deploying to production:

- [ ] Database is multi-AZ with automated backups
- [ ] Redis cluster has 3+ nodes with failover enabled
- [ ] SSL/TLS certificate installed (Let's Encrypt or AWS ACM)
- [ ] Application auto-scaling policy configured (2-10 instances)
- [ ] Health checks enabled on load balancer
- [ ] CloudWatch alarms configured for:
  - [ ] Database CPU > 80%
  - [ ] API error rate > 1%
  - [ ] Signal processing latency > 5s
  - [ ] Matching engine failures
- [ ] Monitoring & alerting (Sentry, Datadog, PagerDuty)
- [ ] Backup strategy tested (RDS, Redis, S3)
- [ ] Disaster recovery runbook prepared
- [ ] Security scan completed (OWASP, penetration test)

### Environment Variables (Production)

Create `.env.production` (stored in AWS Secrets Manager):

```env
NODE_ENV=production
APP_PORT=3000
LOG_LEVEL=warn

# Database
DATABASE_URL=postgresql://ekexadmin:STRONG_PASSWORD@ekex-prod.rds.amazonaws.com:5432/ekex_intelligence
DATABASE_POOL_SIZE=50
DATABASE_POOL_TIMEOUT=30000
DATABASE_SSL=require
DATABASE_REPLICA_URLS=postgresql://reader:PASSWORD@replica1.rds.amazonaws.com:5432/ekex_intelligence,postgresql://reader:PASSWORD@replica2.rds.amazonaws.com:5432/ekex_intelligence

# Cache
REDIS_URL=redis://ekex-prod-cluster.xxxxx.ng.0001.use1.cache.amazonaws.com:6379
REDIS_SSL=true
REDIS_AUTH_TOKEN=<strong-auth-token>

# Auth & Security
JWT_SECRET=<generate-with-openssl-rand-base64-64>
JWT_EXPIRY=15m
JWT_REFRESH_EXPIRY=7d
BCRYPT_ROUNDS=12

# Telegram Bot
TELEGRAM_BOT_TOKEN=<production-bot-token>
TELEGRAM_WEBHOOK_URL=https://api.ekexintelligence.com/webhook/telegram

# Monitoring & Observability
SENTRY_DSN=https://xxxxx@sentry.io/yyyyy
SENTRY_ENVIRONMENT=production
DATADOG_API_KEY=<datadog-key>
PAGERDUTY_INTEGRATION_KEY=<pagerduty-key>

# Feature Flags
FEATURE_MATCHING_ENGINE=true
FEATURE_GAP_DETECTION=true
FEATURE_FEEDBACK_LOOP=true

# Rate Limiting
RATE_LIMIT_WINDOW_MS=60000
RATE_LIMIT_MAX_REQUESTS=100
RATE_LIMIT_MAX_API_KEY=1000
```

---

## Environment Configuration

### Configuration Hierarchy

```
1. Command-line arguments (highest priority)
2. Environment variables (.env file)
3. Environment-specific config (config/production.js)
4. Default config (config/default.js)
5. Hardcoded defaults (lowest priority)
```

### .env File Example

```bash
# Server
NODE_ENV=development
APP_PORT=3000
APP_HOST=localhost

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/ekex_intelligence
DATABASE_POOL_SIZE=20
DATABASE_SSL=false

# Cache
REDIS_URL=redis://localhost:6379
REDIS_PASSWORD=

# Auth
JWT_SECRET=dev-secret-change-in-production
JWT_EXPIRY=15m
JWT_REFRESH_EXPIRY=7d

# External Services
TELEGRAM_BOT_TOKEN=xxxx:yyyy
SENTRY_DSN=

# Feature Flags
FEATURE_MATCHING_ENGINE=false
FEATURE_GAP_DETECTION=false
```

### Configuration Validation

On startup, the application validates required variables:

```bash
$ npm start

✓ Checking environment configuration...
✓ DATABASE_URL is set
✓ JWT_SECRET is set (min 32 chars)
✓ REDIS_URL is set
✓ TELEGRAM_BOT_TOKEN is set
✓ All required environment variables present
```

---

## Database Migrations

### Running Migrations

```bash
# Run pending migrations
npm run db:migrate

# Run with environment override
npm run db:migrate -- --environment production

# Show migration status
npm run db:migrate:status

# Rollback last migration
npm run db:migrate:rollback

# Rollback specific migration
npm run db:migrate:rollback -- --step 2
```

### Migration Files

Located in `src/database/migrations/`:

```
migrations/
├── 20260528_120000_create_consumer.sql
├── 20260528_120100_create_supplier.sql
├── 20260528_120200_create_consumer_signal.sql
├── 20260528_120300_create_product_match.sql
└── ...
```

### Production Migration Safety

```bash
# Test migration on staging first
npm run db:migrate -- --environment staging

# Create snapshot before migration
aws rds create-db-snapshot \
  --db-instance-identifier ekex-prod \
  --db-snapshot-identifier ekex-prod-backup-$(date +%Y%m%d-%H%M%S)

# Run migration with monitoring
npm run db:migrate -- --environment production --verbose
```

---

## Health Checks & Monitoring

### Application Health Endpoint

```bash
GET /health
```

Response (healthy):
```json
{
  "status": "ok",
  "uptime": 3600,
  "timestamp": "2026-05-28T12:00:00Z",
  "services": {
    "database": "connected",
    "redis": "connected",
    "telegram_bot": "connected"
  }
}
```

### Readiness Probe (Kubernetes)

```bash
GET /ready
```

```json
{
  "ready": true,
  "database": "ok",
  "cache": "ok"
}
```

### Monitoring Metrics

| Metric | Target | Alert Threshold |
|--------|--------|-----------------|
| API Response Time (p95) | < 200ms | > 500ms |
| Error Rate | < 0.1% | > 1% |
| Database CPU | < 70% | > 80% |
| Database Connections | < 30 | > 40 |
| Redis Memory | < 70% | > 85% |
| Signal Processing | < 5s | > 10s |
| Matching Engine | < 2s | > 5s |

### Log Aggregation

Logs are centralized in:
- **Development**: Console (colorized)
- **Staging**: CloudWatch Logs
- **Production**: CloudWatch + Datadog

Log format:
```json
{
  "timestamp": "2026-05-28T12:00:00.123Z",
  "level": "info",
  "service": "signal-processor",
  "message": "Signal processed",
  "correlation_id": "xxxx-yyyy",
  "metadata": {}
}
```

---

## Troubleshooting Deployments

### Common Issues

**Issue**: Database migration fails

```bash
# Check migration status
npm run db:migrate:status

# View migration logs
docker-compose logs db

# Rollback and re-check
npm run db:migrate:rollback
```

**Issue**: Redis connection timeout

```bash
# Check Redis health
redis-cli ping

# View Redis logs
docker-compose logs redis

# Restart Redis
docker-compose restart redis
```

**Issue**: Telegram bot not receiving webhooks

```bash
# Verify webhook URL
curl https://api.telegram.org/bot<TOKEN>/getWebhookInfo

# Re-register webhook
npm run telegram:register-webhook
```

---

<div align="center">

**[← Back to README](../README.md)** · **[Architecture Documentation](./ARCHITECTURE.md)** · **[Database Schema](./DATABASE.md)**

*© 2026 Kimuntu Group · EKEX Intelligence*

</div>
