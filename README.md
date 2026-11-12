# openshift-101-workshop

# Build
podman build -t myfirstcontainer:v1.0 .

# Run
podman run --platform linux/amd64 -d --rm --name c1 --publish 8080:80 myfirstcontainer:v1.0
podman run -d --rm --name c2 --publish 8081:80 myfirstcontainer:v1.0
podman run -d --rm --name c3 --publish 8082:80 myfirstcontainer:v1.0

# See running processes in container
podman exec -it c1 sh
ps -ef

# SSH into podman machine
podman machine ssh
ps -ef --forest



# Volume mount
podman run --rm -it --volume ./src/:/app/:Z --name c1 --publish 8081:80 myfirstcontainer:v1.0 /bin/sh


# supertiny
podman build -t supertinyapp:v1.0 .


# Distroless
podman run --rm --volume ./src/:/app/:Z --workdir /app/ registry.access.redhat.com/hi/python:latest python test.py
podman run --rm --volume ./src/:/app/:Z --workdir /app/ registry.access.redhat.com/hi/python:latest python -c "import os; print(os.listdir('/bin'))"
podman run --rm --volume ./src/:/app/:Z --workdir /app/ registry.access.redhat.com/hi/python:3.15-builder python -c "import os; print(os.listdir('/bin'))"

podman run --rm -it --volume ./src/:/app/:Z --workdir /app/ registry.access.redhat.com/hi/python:3.15-builder /bin/sh

