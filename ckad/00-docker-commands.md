## Docker commands

```bash
# Show available images on the system
docker images

# Build docker image from Dockerfile in the same folder
docker build -t <app-name> .
# Example: docker build -t http-echo

# Tag docker image with complete name to upload it to the image registry (dockerhub)
docker tag <app-name>:<tag> <registry-account>/<app-name>:<tag>
# Example: docker tag http-echo:latest antoniomarfer/http-echo:latest

# Push image to the image registry
docker push <registry-account>/<app-name>:<tag>
# Example: docker push antoniomarfer/http-echo:latest

# Run image mapping port
docker run -p <host-port>:<container-port> <app-name>
# example: docker run 9000:8080 http-echo

# List running containers
docker ps
```
