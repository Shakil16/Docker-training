# Day 05: Dockerfile Instructions, Users, and Networking

## Objectives

By the end of this session, you should be able to:

- understand the purpose of common Dockerfile instructions
- build images using `CMD` and `ENTRYPOINT`
- run containers as a non-root user
- understand basic Docker networking concepts

## 1. Dockerfile Basics

A Dockerfile is a recipe for building a container image. Each instruction creates a layer in the final image.

```dockerfile
# Use an official Python runtime as a parent image
FROM python:3.11-slim

# Set the working directory in the container
WORKDIR /app

# Copy the application files
COPY . .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Expose the application port
EXPOSE 8080

# Start the application
CMD ["python", "app.py"]
```

### Common instructions

- `FROM`: defines the base image
- `WORKDIR`: sets the working directory inside the container
- `COPY` / `ADD`: copy files into the image
- `RUN`: executes commands during the build
- `USER`: switches to a non-root user
- `EXPOSE`: documents the port used by the app
- `CMD`: defines the default command
- `ENTRYPOINT`: defines the main executable

## 2. Running as a Non-Root User

Running containers as non-root improves security.

```dockerfile
FROM python:3.11-slim

RUN groupadd -r appuser && useradd -r -g appuser -d /app -s /sbin/nologin appuser

WORKDIR /app
COPY . .
RUN chown -R appuser:appuser /app

USER appuser
EXPOSE 8080
CMD ["python", "app.py"]
```

### Hands-on

```bash
docker build -t test1-user -f dockerfile.user .
docker run --rm -d --name app3 -p 8080:8080 test1-user:latest
docker exec -it app3 sh
id
cd /root
```

You should see that the container user does not have permission to access `/root` as a privileged user.

## 3. `CMD` vs `ENTRYPOINT`

### `CMD`

- provides a default command or default arguments
- can be overridden easily by the user

```dockerfile
FROM ubuntu
CMD ["echo", "Default message"]
```

```bash
docker build -t cmd-image1 -f dockerfile.cmd .
docker run -it cmd-image1:latest
```

### `ENTRYPOINT`

- defines the main executable for the container
- additional arguments are appended to it

```dockerfile
FROM ubuntu
ENTRYPOINT ["echo", "Default message"]
```

```bash
docker build -t cmd-image2 -f dockerfile.entrypoint .
docker run -it cmd-image2:latest
```

### Using both together

```dockerfile
FROM ubuntu
ENTRYPOINT ["echo"]
CMD ["Default message"]
```

```bash
docker build -t cmd-image3 --no-cache -f dockerfile.both .
docker run -it cmd-image3:latest
docker run -it cmd-image3:latest hello world
```

### Quick summary

| Instruction | Purpose | Typical behavior |
| --- | --- | --- |
| `CMD` | Default command or arguments | Often replaced by `docker run` arguments |
| `ENTRYPOINT` | Main executable | Appends extra arguments to the entrypoint |



## 5. Key Takeaways

- Dockerfiles define how images are built.
- Use `USER` to avoid running as root.
- `CMD` is flexible; `ENTRYPOINT` is more fixed.

