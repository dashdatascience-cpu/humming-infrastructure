# Humming Traefik Configuration

## Installation

Helm release: humming-traefik
Namespace: jenkins
Helm chart: traefik
Chart version: 42.2.0
Traefik application version: v3.7.16

## Install on a new Kubernetes cluster

Prerequisites:
- Kubernetes cluster
- Helm installed
- jenkins and humming-dev namespaces created

Add the Traefik Helm repository:

helm repo add traefik https://traefik.github.io/charts
helm repo update

Install Traefik:

helm upgrade --install humming-traefik traefik/traefik \
  --namespace jenkins \
  --version 42.2.0 \
  -f kubernetes/traefik/helm-custom-values.yaml

## Routing

- public-routes.json: Jenkins and Auth
- auth-middleware.json: Auth prefix removal
- catalog-route.json: Catalog
- catalog-middleware.json: Catalog prefix removal

Apply routes after installing Traefik:

kubectl apply -f kubernetes/traefik/

Note: This directory also contains Helm values and documentation.
For a controlled installation, apply the four JSON route and
middleware files individually instead of applying the entire directory.

## Public Access

Traefik is exposed through the separate jenkins-public
LoadBalancer Service, not through its Helm-managed Service.

The public LoadBalancer must be recreated separately when
migrating to another Kubernetes cluster.
