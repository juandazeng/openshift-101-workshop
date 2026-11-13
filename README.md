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



# Volume mount and owner mapping
podman run --rm -d --name c1 --volume ./src/:/app/:Z myfirstcontainer:v1.0
podman exec -it c1 /bin/bash

# supertiny
podman build -t supertinyapp:v1.0 .
podman run --rm -it supertinyapp:v1.0


# Distroless
podman run --rm --volume ./src/:/app/:Z --workdir /app/ registry.access.redhat.com/hi/python:latest python test.py
podman run --rm --volume ./src/:/app/:Z --workdir /app/ registry.access.redhat.com/hi/python:latest python -c "import os; print(os.listdir('/bin'))"
podman run --rm --volume ./src/:/app/:Z --workdir /app/ registry.access.redhat.com/hi/python:3.15-builder python -c "import os; print(os.listdir('/bin'))"

podman run --rm -it --volume ./src/:/app/:Z --workdir /app/ registry.access.redhat.com/hi/python:3.15-builder /bin/sh



# RHACS

## roxctl
wget -O roxctl --header="Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6Imp3dGswIiwidHlwIjoiSldUIn0.eyJhdWQiOiI0ZGYxYjk4Yy0yNGVkLTQwNzMtYTlhZC0zNTZhZWM2YmI2MmQiLCJleHAiOjE3OTQ1MTcwNTgsImV4dGVybmFsX3VzZXIiOnsiQXR0cmlidXRlcyI6eyJyb2xlIjpbIkFkbWluIl0sInVzZXJuYW1lIjpbImFkbWluIl19LCJ1c2VyX2lkIjoiYWRtaW4ifSwiaWF0IjoxNzk0NDczODU4LCJpc3MiOiJodHRwczovL3N0YWNrcm94LmlvL2p3dCIsImp0aSI6IjkwMjc2ZDNmLTM5YzAtNDJhMS1hNTJhLTJjZDM3NzQyNzIxYSJ9.db_83_V06hqTPWcUYWvOBwNoT1InCN9aI4zeeiJUvBvRGPakX5BP2tVjzSJPc17dPD6Pbd30awkfy03EDv_Ix5NooIINDjjkEr2rnAc9TPnB_DqnNo-Mc8-T0gq_nX5bLrapCDzsF-EpC3MCk6vS7uD4qvN1aiTzrVbwbyoENl6Y_3QokmEusdACZtOsatwjXMWZHmyb339eIwxhlthGxAyZ5MRcvaOe7OU5JD8MVjZTDszZ5qc0bQZe4T-Qhz7E_DkTzckg8BT0A20Q02Z59LNJo43bx9QFteaeZhGkxPraK7y9Ap6zfF3cEJpiQq33rovXFw4bcAm_6-RUWlk-hzS8RlIng0NnTIvI5-BH75rbVShiXxtPFTG4_an7YEkhd29_OqIz82kCl93eYrBiA3DjjVy0WiISglNLYMtjUAcLXf7XHKmxBpVVcMdu5mS4AGS80P3blBDbxWrDvDLm5jA9ft2A_CvlsX_Y111f4oYnWsIZY9i5TGSOpqBq9JFCBpRDIfywVr8AZhEvPYDxYaY7ygUsTX0lkorJeCFoxZg1RJ_MDyZSDp8qa0ip4Ekup-3NjNxOlbjTNAqA039tDEmEqlcqrSxHtgoV_ZjrO1Z0mskX489Zcgp4vKkouzS28J9brxhT-Va-Rj_ugRpKPPNB2GMNWV476HtuFUuXAcA" https://central-stackrox.apps.cluster-bz5rg.dyn.redhatworkshops.io/api/cli/download/roxctl-linux-amd64

chmod +x roxctl

export ROX_API_TOKEN=...
export ROX_ENDPOINT=central-stackrox.apps.cluster-bz5rg.dyn.redhatworkshops.io:443

## podman to quay.io and scan
podman login quay.io

podman tag localhost/myfirstcontainer:v1.0 quay.io/bankingdemo/myfirstcontainer:v1.0
podman push quay.io/bankingdemo/myfirstcontainer:v1.0

podman tag localhost/supertinyapp:v1.0 quay.io/bankingdemo/supertinyapp:v1.0
podman push quay.io/bankingdemo/supertinyapp:v1.0

roxctl --insecure-skip-tls-verify image scan --image=quay.io/bankingdemo/myfirstcontainer:v1.0 --severity CRITICAL,IMPORTANT -o table
roxctl --insecure-skip-tls-verify image scan --image=quay.io/bankingdemo/supertinyapp:v1.0 --severity CRITICAL,IMPORTANT -o table


## OpenShift deployment
oc new-project uobdemo
oc delete all -l app=myfirstcontainer -n uobdemo
oc apply -f myfirstcontainer.yaml -n uobdemo
roxctl deployment check --file myfirstcontainer.yaml
