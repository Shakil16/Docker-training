To test the Java multistage build application, clone the below repo
git clone https://github.com/spring-projects/spring-petclinic.git


Use Case
How to Reduce Docker Image Size
Minimizing Docker image sizes accelerates container deployment, and for large-scale operations, this can lead to substantial savings in storage space.

1. Use Official Minimal Base Images:
When building Docker images, always start with an official base image. Instead of using a full-sized OS image, opt for lightweight versions like python:3.9-slim or python:3.9-alpine. These minimal images contain only the essentials, significantly reducing the image size.

Taking an example for a Python image, here are the image sizes for python:3.9 vs python:3.9-alpine:


Python 3.9-alpine is a whomping 95.2% smaller than Python 3.9.

2. Minimize Layers:
Every command in your Dockerfile (like RUN, COPY, etc.) generates a separate layer in the final image. Grouping similar commands together into one step decreases the total number of layers, leading to a smaller overall image size.

Instead of doing this:

RUN apk update
RUN apk add --no-cache git
RUN rm -rf /var/cache/apk/*
Do this:

RUN apk update && apk add --no-cache git && rm -rf /var/cache/apk/*
3. Use .dockerignore File:
When creating Docker images, Docker transfers all the files from your project directory into the image by default. To avoid including unneeded files, use a .dockerignore file to exclude them.

Sample .dockerignore

__pycache__
*.pyc
*.pyo
*.pyd
venv/
4. Multi-Stage Builds (Mandatory atleast for me 😀):
Multi-stage builds enable you to divide the build process from the final runtime environment. This approach is particularly beneficial when your application needs certain tools for compiling that are not necessary in the final image.

Single Stage Vs Multi-Stage Builds Comparison:

Take an example of a Flask app built using the python:3.9-alpine image with a single-stage Dockerfile like:

# Use an official Python runtime as a parent image
FROM python:3.9-alpine

# Install necessary build dependencies
RUN apk add --no-cache build-base \
    && apk add --no-cache gfortran musl-dev lapack-dev

# Set the working directory
WORKDIR /app

# Copy the requirements file and install dependencies
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application code to the working directory
COPY . .

# Expose the port the app will run on
EXPOSE 5000

# Run the Flask app
CMD ["python", "app.py"]

Docker - Single Stage Build Output

The image built was of size: 588 MB

Redesigned Multi Stage Dockerfile looks like:


# Dockerfile.multi-stage

# Stage 1: Build
FROM python:3.9-alpine AS builder

# Install necessary build dependencies
RUN apk add --no-cache build-base \
    && apk add --no-cache gfortran musl-dev lapack-dev

# Set the working directory
WORKDIR /app

# Copy the requirements file and install dependencies
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application code to the working directory
COPY . .

# Uninstall unnecessary dependencies
RUN pip uninstall -y pandas && apk del build-base gfortran musl-dev lapack-dev

# Stage 2: Production
FROM python:3.9-alpine

# Set the working directory
WORKDIR /app

# Copy only the necessary files from the build stage
COPY --from=builder /app /app

# Expose the port the app will run on
EXPOSE 5000

# Run the Flask app
CMD ["python", "app.py"]

Docker - Multi Stage Build Output

The new image size was: Only 47.7 MB

The application works exactly the same, but it spins up much faster in this version.

This is an illustration of the drastic effect of Multi-Stage Builds.



5. Use Static Binaries and the 'scratch' Base Image:
If your application can be compiled into a static binary, you can use the scratch base image, which is essentially an empty image. This leads to extremely small final images.

Example:

FROM scratch
COPY myapp /
CMD ["/myapp"]
Works well for applications that don’t need operating system-level dependencies.

Security Considerations
Use Trusted and Official Base Images

Run Containers as Non-Root Users

Regularly scan your Docker images for known vulnerabilities

Limit the network exposure of your container by restricting the ports and IP addresses

docker run -p 127.0.0.1:8080:8080 myimage
Avoid hardcoding sensitive information like API keys or passwords directly into your Dockerfile or environment variables.

Final reminder,

Less the image size = Faster deployments + Quicker scaling + Lean infrastructure

