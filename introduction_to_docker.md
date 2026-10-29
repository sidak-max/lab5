# Introduction to Docker

## Overview

Docker is a platform that helps developers package an application and its dependencies into a **container**. Containers make it easier to run software consistently across development, testing, and production environments.

## Key Concepts

- **Image:** A template used to create containers.
- **Container:** A running instance of an image.
- **Dockerfile:** A text file containing instructions to build an image.
- **Docker Hub:** An online registry for sharing container images.
- **Port mapping:** Connects a port on your computer to a port inside a container.

## Common Commands

```bash
# Check the installed Docker version
docker --version

# Show Docker system information
docker info

# Download an image
docker pull nginx

# Run Nginx and map host port 8080 to container port 80
docker run -d --name my-nginx -p 8080:80 nginx

# List running containers
docker ps

# View container logs
docker logs my-nginx

# Stop the container
docker stop my-nginx

# Remove the container
docker rm my-nginx
```

After starting the Nginx container, open <http://localhost:8080> in a browser to see the default Nginx page, provided Docker is running and the port is available.

## Example Dockerfile

Create a file named `Dockerfile` in a project folder:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

This example assumes the project contains `requirements.txt` and `app.py`. Build the image and run it with:

```bash
docker build -t my-python-app .
docker run --rm -p 5000:5000 my-python-app
```

The Python application must listen on `0.0.0.0` inside the container for port publishing to work correctly.

## Benefits of Docker

1. Consistent environments across machines.
2. Faster and repeatable application deployment.
3. Isolation between applications and their dependencies.
4. Easy sharing and distribution of application images.
5. Convenient scaling and automation in development workflows.

## Troubleshooting Tips

- If `docker` is not recognized, check that Docker is installed and available on your system's `PATH`.
- If `docker info` hangs or fails, make sure Docker Desktop or the Docker daemon is running.
- If a port is already in use, choose another host port, such as `-p 8081:80`.
- Use `docker logs <container-name>` to investigate application startup errors.

## Conclusion

Docker simplifies application packaging and deployment by using portable containers. Learning images, containers, Dockerfiles, and basic CLI commands provides a strong foundation for containerized development.
