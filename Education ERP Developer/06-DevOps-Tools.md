---
title: DevOps Tools
tags: [git, docker, cicd, devops]
created: 2026-02-03
---

# DevOps Tools

## 🎯 Git & GitHub

### Essential Commands

```bash
# Branching
git checkout -b feature/fee-module    # Create & switch
git branch -d feature/fee-module      # Delete local
git push origin --delete feature/fee  # Delete remote

# Stashing
git stash                             # Save changes
git stash list                        # List stashes
git stash pop                         # Apply & remove
git stash apply stash@{0}             # Apply specific

# Rebasing (clean history)
git rebase main                       # Rebase on main
git rebase -i HEAD~3                  # Interactive (squash commits)

# Cherry-pick
git cherry-pick abc123                # Apply specific commit

# Reset
git reset --soft HEAD~1               # Undo commit, keep changes staged
git reset --mixed HEAD~1              # Undo commit, keep changes unstaged
git reset --hard HEAD~1               # Undo commit, discard changes

# Reflog (recover lost commits)
git reflog                            # View history
git checkout HEAD@{2}                 # Recover state
```

### Git Flow for ERP Projects

```
main ─────●─────────────●─────────────●───────
          │             │             │
          │   release/  │             │
          │   v1.2.0    │             │
          │      │      │             │
develop ──●──────●──────●─────────────●───────
          │      │      │             │
          │      │      │             │
feature/  │      │      │             │
fee-module●──────┘      │             │
                        │             │
hotfix/                 │             │
payment-fix ────────────●─────────────┘
```

```bash
# Feature development
git checkout develop
git checkout -b feature/exam-module
# ... work ...
git push origin feature/exam-module
# Create PR to develop

# Release
git checkout develop
git checkout -b release/v1.2.0
# ... testing, bug fixes ...
git checkout main
git merge release/v1.2.0
git tag v1.2.0

# Hotfix
git checkout main
git checkout -b hotfix/payment-fix
# ... fix ...
git checkout main
git merge hotfix/payment-fix
git checkout develop
git merge hotfix/payment-fix
```

### Commit Message Convention

```
<type>(<scope>): <subject>

<body>

<footer>

# Types
feat:     New feature
fix:      Bug fix
docs:     Documentation
style:    Formatting (no code change)
refactor: Code restructuring
test:     Adding tests
chore:    Maintenance

# Examples
feat(fees): add bulk fee generation API

- Added POST /api/fees/bulk endpoint
- Supports class-wise fee creation
- Queues email notifications

Closes #123
```

## 🐳 Docker

### Dockerfile for Node.js ERP

```dockerfile
# Multi-stage build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:20-alpine
WORKDIR /app

# Security: non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

COPY --from=builder /app/node_modules ./node_modules
COPY . .

USER nodejs
EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

CMD ["node", "src/index.js"]
```

### Dockerfile for Laravel

```dockerfile
FROM php:8.2-fpm-alpine

# Install extensions
RUN docker-php-ext-install pdo pdo_mysql opcache

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/html
COPY . .

RUN composer install --no-dev --optimize-autoloader
RUN php artisan config:cache && \
    php artisan route:cache && \
    php artisan view:cache

RUN chown -R www-data:www-data storage bootstrap/cache

EXPOSE 9000
CMD ["php-fpm"]
```

### Docker Compose for ERP Stack

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DB_HOST=mysql
      - MONGO_URI=mongodb://mongo:27017/erp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - mysql
      - mongo
      - redis
    networks:
      - erp-network

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
      MYSQL_DATABASE: erp
      MYSQL_USER: erp_user
      MYSQL_PASSWORD: ${DB_PASSWORD}
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - erp-network

  mongo:
    image: mongo:6.0
    volumes:
      - mongo-data:/data/db
    networks:
      - erp-network

  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
    networks:
      - erp-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - app
    networks:
      - erp-network

volumes:
  mysql-data:
  mongo-data:
  redis-data:

networks:
  erp-network:
    driver: bridge
```

### Docker Commands

```bash
# Build & run
docker build -t erp-app:v1.0 .
docker run -d -p 3000:3000 --name erp erp-app:v1.0

# Compose
docker-compose up -d              # Start all
docker-compose down               # Stop all
docker-compose logs -f app        # Follow logs
docker-compose exec app sh        # Shell into container

# Debugging
docker logs erp --tail 100        # Last 100 lines
docker exec -it erp sh            # Interactive shell
docker stats                      # Resource usage

# Cleanup
docker system prune -a            # Remove unused
docker volume prune               # Remove unused volumes
```

## 🔄 CI/CD Pipelines

### GitHub Actions

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: root
          MYSQL_DATABASE: test_db
        ports:
          - 3306:3306
        options: --health-cmd="mysqladmin ping" --health-interval=10s
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linter
        run: npm run lint
      
      - name: Run tests
        run: npm test
        env:
          DB_HOST: localhost
          DB_USER: root
          DB_PASSWORD: root
          DB_NAME: test_db

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: |
            myorg/erp-app:latest
            myorg/erp-app:${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - name: Deploy to server
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /opt/erp
            docker-compose pull
            docker-compose up -d
            docker system prune -f
```

### GitLab CI

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

variables:
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

test:
  stage: test
  image: node:20
  services:
    - mysql:8.0
  variables:
    MYSQL_ROOT_PASSWORD: root
    MYSQL_DATABASE: test_db
  script:
    - npm ci
    - npm run lint
    - npm test
  only:
    - merge_requests
    - main
    - develop

build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $DOCKER_IMAGE .
    - docker push $DOCKER_IMAGE
  only:
    - main

deploy:
  stage: deploy
  image: alpine
  before_script:
    - apk add --no-cache openssh-client
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | ssh-add -
  script:
    - ssh -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_HOST "
        cd /opt/erp &&
        docker-compose pull &&
        docker-compose up -d"
  only:
    - main
  environment:
    name: production
```

## ❓ Interview Q&A

> [!question]- What is the difference between `git merge` and `git rebase`?
> ```
> Merge (preserves history):
> main:    A───B───────M
>               \     /
> feature:       C───D
> 
> Rebase (linear history):
> main:    A───B
>               \
> feature:       C'──D' (rebased)
> ```
> - **Merge**: Creates merge commit, preserves branch history
> - **Rebase**: Rewrites history, creates linear timeline
> - Use merge for shared branches, rebase for local cleanup

> [!question]- How do you resolve merge conflicts?
> ```bash
> # 1. Identify conflicts
> git status
> 
> # 2. Open conflicted files
> <<<<<<< HEAD
> your changes
> =======
> their changes
> >>>>>>> feature-branch
> 
> # 3. Edit to resolve, remove markers
> 
> # 4. Stage and commit
> git add .
> git commit -m "Resolve merge conflicts"
> ```

> [!question]- Explain Docker layers and caching
> ```dockerfile
> # Each instruction creates a layer
> FROM node:20          # Layer 1 (cached)
> WORKDIR /app          # Layer 2 (cached)
> COPY package*.json ./ # Layer 3 (invalidated if package.json changes)
> RUN npm install       # Layer 4 (rebuilds if Layer 3 changes)
> COPY . .              # Layer 5 (invalidated on any code change)
> ```
> Order instructions from least to most frequently changing for better caching.

> [!question]- What is the difference between Docker CMD and ENTRYPOINT?
> ```dockerfile
> # ENTRYPOINT: Main command (hard to override)
> ENTRYPOINT ["node"]
> CMD ["app.js"]
> # Runs: node app.js
> # docker run myapp server.js → node server.js
> 
> # CMD alone: Default command (easily overridden)
> CMD ["node", "app.js"]
> # docker run myapp bash → bash (replaces CMD)
> ```

> [!question]- How do you handle secrets in CI/CD?
> 1. **Environment variables**: Set in CI/CD platform
> 2. **Secret managers**: AWS Secrets Manager, HashiCorp Vault
> 3. **Encrypted files**: git-crypt, SOPS
> 4. **Never**: Commit secrets to repo
> ```yaml
> # GitHub Actions
> env:
>   DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
> 
> # Docker
> docker run -e DB_PASSWORD=$DB_PASSWORD myapp
> ```

> [!question]- What is blue-green deployment?
> ```
> ┌─────────────┐     ┌─────────────┐
> │   Blue      │     │   Green     │
> │  (v1.0)     │     │  (v1.1)     │
> │  [Active]   │     │  [Standby]  │
> └──────┬──────┘     └──────┬──────┘
>        │                   │
>        └─────────┬─────────┘
>                  │
>           ┌──────┴──────┐
>           │ Load Balancer│
>           └─────────────┘
> ```
> - Two identical environments
> - Switch traffic instantly
> - Easy rollback (switch back)
> - Zero downtime deployments

> [!question]- What is container orchestration?
> Managing multiple containers across hosts:
> - **Kubernetes**: Industry standard, complex
> - **Docker Swarm**: Simpler, built into Docker
> - **ECS/Fargate**: AWS managed
> 
> Features: Auto-scaling, load balancing, self-healing, rolling updates
