# PhatNDH CI/CD Template

Demo monorepo for testing CI/CD before the real backend repository is ready.

## Structure

```text
backend/
  api-gateway/
  auth-service/
  user-service/
  booking-service/
  payment-service/
.github/workflows/
  ci.yml
  cd.yml
deploy/
  docker-compose.yml
  .env.example
  nginx/nginx.conf
scripts/
  deploy.sh
```

Each backend service is a small Spring Boot app with:

- `GET /health`
- its own `pom.xml`
- its own `Dockerfile`

## Local Test

```bash
./mvnw clean verify
```

Build one Docker image after packaging:

```bash
./mvnw -pl backend/auth-service -am package -DskipTests
docker build -t auth-service:local ./backend/auth-service
```

## CI/CD Flow

`ci.yml` runs on pull requests to `main` and pushes to `dev`:

- Maven build
- tests
- Docker build for every service

`cd.yml` runs on pushes to `main` and manual dispatch:

- Maven build
- tests
- build and push service images to GHCR
- SSH to Azure VM and run `/opt/capstone/deploy.sh`

Required GitHub secrets for Azure deploy:

- `AZURE_VM_HOST`
- `AZURE_VM_USER`
- `AZURE_VM_SSH_KEY`

## Azure VM Files

Copy `deploy/` and `scripts/deploy.sh` to `/opt/capstone`, then create the real env file:

```bash
cp deploy/.env.example deploy/.env
```

Update:

```text
IMAGE_OWNER=your-github-username
IMAGE_PROJECT=phatndh-cicd-template
IMAGE_TAG=latest
```

Run:

```bash
cd /opt/capstone
./deploy.sh
```
