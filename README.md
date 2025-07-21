# openshift-101-workshop

# Build
podman build -t myfirstcontainer:v1.0 .

# Run
podman run -d --rm --name c1 --publish 8080:80 myfirstcontainer:v1.0
podman run -d --rm --name c2 --publish 8081:80 myfirstcontainer:v1.0
podman run -d --rm --name c3 --publish 8082:80 myfirstcontainer:v1.0

# See running processes in container
podman exec -it c1 sh
ps -ef

# SSH into podman machine
podman machine ssh
ps -ef --forest
