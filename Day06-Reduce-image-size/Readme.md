# Day 06: Reducing Docker Image Size

## Objectives

By the end of this session, you should be able to:

- explain why smaller images are important
- apply practical techniques to reduce image size
- use multi-stage builds effectively
- understand Docker build cache behavior

## 1. Why Smaller Images Matter

Smaller Docker images are faster to build, faster to pull, and cheaper to store and ship.

Benefits include:

- quicker deployments
- lower storage and bandwidth usage
- reduced attack surface
- faster startup time

## 2. Best Practices to Reduce Image Size

### Use a minimal base image

Prefer official lightweight images such as:

- `python:3.11-slim`
- `python:3.11-alpine`
- `node:20-alpine`

### Combine related instructions

Instead of writing many separate `RUN` steps, combine them.

```dockerfile
RUN apk update && apk add --no-cache git && rm -rf /var/cache/apk/*
```

This reduces the number of layers and avoids unnecessary files in the final image.

### Use `.dockerignore`

Avoid copying unnecessary files into the build context.

```gitignore
__pycache__
*.pyc
*.pyo
*.pyd
venv/
```

### Keep changing files late in the Dockerfile

Place files that change often toward the end so Docker can reuse cached layers.

## 3. Multi-Stage Builds

Multi-stage builds separate the build environment from the final runtime image.

### Single-stage example

```dockerfile
FROM python:3.9-alpine
WORKDIR /app
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["python", "app.py"]
```

This often creates a larger image because build tools and intermediate files remain in the final output.

### Multi-stage example

```dockerfile
# Stage 1: Build
FROM python:3.9-alpine AS builder
WORKDIR /app
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt
COPY . .

# Stage 2: Runtime
FROM python:3.9-alpine
WORKDIR /app
COPY --from=builder /app /app
EXPOSE 5000
CMD ["python", "app.py"]
```

This keeps the runtime image smaller and cleaner.

## 4. Distroless Images

Distroless images contain only your application and its runtime dependencies.

They are useful when you want:

- a very small image
- a smaller attack surface
- fewer package manager tools inside the container

Example:

```dockerfile
FROM gcr.io/distroless/python3
COPY app.py /app.py
CMD ["/app.py"]
```

## 5. Docker Build Cache

Docker reuses cached layers when earlier instructions and inputs have not changed.

A better pattern is:

```dockerfile
FROM node:20
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

Why this works well:

- dependency files are copied first
- `npm install` is cached until dependencies change
- application source changes do not always force a full reinstall

## 6. Practical Commands

```bash
docker build -t myapp:latest .
docker images | sort -k7
docker run --rm -p 5000:5000 myapp:latest
```

## 7. Security Checklist

- use official and trusted base images
- run containers as non-root users
- limit exposed ports
- avoid hardcoding secrets in images
- scan images for vulnerabilities regularly

## 8. Key Takeaway

Reducing image size means faster builds, faster deployments, and a leaner production environment.
