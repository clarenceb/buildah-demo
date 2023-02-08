buildah demo
============

This demo will show how to build a Linux container image within a Kubernetes pod on AKS.
The build happens completely within the pod - no extra pods are spawned to run the build like with `docker buildx`.
Currently, buildah can only build Linux container images, not Windows container images.

It does require using a privileged container, though non-root user (docker buildx does not require using privleged containers) - see link in references below on how to configure buildah to run as a non-privileged, non-root user.

Prerequisites
-------------

* Azure Kubernetes Service (AKS) cluster with a [Mariner OS](https://learn.microsoft.com/EN-us/azure/aks/cluster-configuration#mariner-os) nodepool (for a new Linux kernel version)
* [Azure Container Registry](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-get-started-azure-cli) to store the base buildah container used to execute builds

Demo steps
----------

```sh
az acr build -r <acr_reg> -t docker-buildah -f Dockerfile.buildah .

# Run on Mariner OS for newer kernel
kubectl apply -f buildah-demo.deploy.yaml

# Exec into pod for an interactive demo
kubectl exec -ti buildah-demo -- bash

# Within the buildah container, type the following
cat << EOF > Dockerfile
FROM docker.io/nginx:latest

RUN echo "Installing nginx"; apt update -y; apt install -y nginx

# Expose the default nginx port 80
EXPOSE 80

# Run nginx
CMD ["nginx", "-g", "daemon off;"]
EOF

buildah bud -t ubuntu-nginx

buildah images

#REPOSITORY                TAG      IMAGE ID       CREATED         SIZE
#localhost/ubuntu-nginx    latest   8a95fa283b11   6 minutes ago   164 MB
#docker.io/library/nginx   latest   9eee96112def   4 days ago      146 MB
```

Optional - push image to ACR
----------------------------

```sh
docker login <acr_login_server> -u <acr_username> -p <acr_password>
buildah tag localhost/ubuntu-nginx <acr_login_server>/ubuntu-linux
buildah push <acr_login_server>/ubuntu-linux
```

References
----------

* [How to use Podman inside of Kubernetes](https://www.redhat.com/sysadmin/podman-inside-kubernetes) -- read this to see how to run buildah without a privileged container
