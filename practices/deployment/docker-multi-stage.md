---
concern: deployment
tech: [docker, node, typescript]
priority: recommended
source-repo: supportforge-platform
applies-to: [node, typescript, docker]
---
# Docker Multi-Stage Builds for Node.js

## PATTERN
Use a two-stage Dockerfile: a `builder` stage that installs all dependencies and compiles TypeScript, and a `production` stage that copies only the build output and production dependencies. Run as a non-root user with `dumb-init` as the entrypoint for proper signal handling.

## WHY
Single-stage Docker images include dev dependencies, source code, and build tools -- inflating image size by 2-5x and increasing attack surface. Multi-stage builds produce minimal production images. Running as non-root and using `dumb-init` prevents container-level security and zombie process issues.

## EXAMPLE
From supportforge-platform (`Dockerfile`):
```dockerfile
FROM node:24-alpine AS builder
WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci && npm cache clean --force

# Copy source and build
COPY tsconfig.json ./
COPY src ./src
RUN npm run build:simple

# ---- Production image ----
FROM node:24-alpine AS production

RUN apk update && apk upgrade && \
    apk add --no-cache dumb-init curl bash && \
    addgroup -g 1001 -S wellforce && \
    adduser -S wellforce -u 1001

WORKDIR /app

# Production deps only
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Copy build output only
COPY --from=builder /app/dist ./dist

USER wellforce
EXPOSE 3002
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "dist/server.js"]
```

## CHECK
How to verify if a repo already follows this:
- [ ] Dockerfile exists with at least two `FROM` stages
- [ ] Production stage uses `--only=production` for npm/pnpm install
- [ ] Non-root user is created and switched to via `USER`
- [ ] `dumb-init` or equivalent signal handler is the entrypoint
- [ ] `npm cache clean --force` follows install steps

## IMPLEMENT
1. Create a `Dockerfile` with builder and production stages
2. In builder: `COPY package*.json`, `RUN npm ci`, copy source, `RUN npm run build`
3. In production: install system deps + dumb-init, create non-root user, `npm ci --only=production`, `COPY --from=builder` the dist output
4. Set `USER`, `EXPOSE`, `ENTRYPOINT ["dumb-init", "--"]`, `CMD`
5. Add `.dockerignore` excluding `node_modules`, `.git`, `.env`, `*.md`

## NOTES
- Use `npm ci` (not `npm install`) for reproducible builds from lockfile
- Copy `package*.json` before source code to leverage Docker layer caching for dependencies
- Alpine images are preferred for smaller size but may need extra system deps
- For monorepos, consider a root-level Dockerfile with selective COPY for the target package
