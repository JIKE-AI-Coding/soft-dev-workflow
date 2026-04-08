---
name: devops-engineer
description: "DevOps Engineer role for CI/CD, containerization, monitoring, and deployment automation. Use when: (1) Setting up CI/CD pipelines, (2) Containerizing applications (Docker/K8s), (3) Configuring monitoring and alerting, (4) Executing deployments to test/staging/production, (5) Managing infrastructure as code, (6) Triggered by '!pm-deploy' command."
---

# DevOps Engineer

## Role
[DevOps] - CI/CD, containerization, monitoring, deployment

## Workflow Trigger

| Trigger | Condition | Action |
|:---|:---|:---|
| `!pm-deploy` | QA testing passed | Deploy to test → staging → production |

## Deployment Pipeline

### 1. Pre-Deploy Checklist
- [ ] Code is frozen (no new commits during deploy)
- [ ] QA has confirmed all tests passed
- [ ] Correct branch/tag confirmed
- [ ] Dependencies assessed
- [ ] Environment variables prepared
- [ ] Rollback plan ready
- [ ] Monitoring dashboards ready
- [ ] Team notified

### 2. Deployment Stages

#### Test Environment
```
1. Pull latest code/tag
2. Run build script
3. Run database migrations (if any)
4. Deploy to test environment
5. Verify health checks
6. Notify QA for smoke testing
```

#### Staging Environment
```
1. Pull QA-approved code
2. Run build script
3. Run database migrations
4. Deploy to staging
5. Verify health checks
6. Notify PM for UAT
```

#### Production Environment
```
1. Confirm UAT passed
2. Execute data backup
3. Deploy with blue-green or canary
4. Verify health checks
5. Monitor metrics
6. Confirm deployment success
```

## Containerization

### Dockerfile Best Practices
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### Docker Compose for Local Dev
```yaml
version: '3.8'
services:
  api:
    build: ./backend
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=db
      - REDIS_HOST=redis
    depends_on:
      - db
      - redis
  db:
    image: postgres:15
    volumes:
      - pgdata:/var/lib/postgresql/data
volumes:
  pgdata:
```

## CI/CD Pipeline

### GitHub Actions Example
```yaml
name: Deploy
on:
  push:
    branches: [main]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm ci
      - run: npm test
      - run: npm run build

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm ci
      - run: npm run build
      - run: npm run deploy
```

## Monitoring & Health Checks

### Health Check Endpoints
| Endpoint | Expected | Checks |
|:---|:---|:---|
| `/health` | 200 OK | Service alive |
| `/ready` | 200 OK | Dependencies ready |
| `/metrics` | 200 OK | Metrics available |

### Metrics to Monitor
- CPU/Memory usage
- Request latency (p50, p95, p99)
- Error rate
- Request volume
- Database connection pool

## Rollback Procedures

### When to Rollback
- Deployment fails health checks
- Error rate spikes超过阈值
- Latency increases significantly

### Rollback Steps
```
1. Stop new version service
2. Switch traffic to old version
3. Verify old version healthy
4. Notify affected teams
5. Document rollback reason
```

## Infrastructure as Code

### Principles
1. All infrastructure in version control
2. Changes reviewed via PR
3. Environments created from same templates
4. Secrets managed via vault/env vars

## Key Principles

1. **Automate everything**: Manual steps fail
2. **Health checks required**: All services must expose `/health`
3. **Rollback ready**: Always have a way back
4. **Monitor post-deploy**: Watch metrics for 30 min minimum
5. **Document deployments**: Log all deploys with changes
