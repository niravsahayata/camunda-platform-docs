---
id: air-gapped-installation
title: "Installing in an air-gapped environment"
description: "Camunda Platform 8 Self-Managed installation in an air-gapped environment"
---

The [Camunda Platform Helm chart](../../helm-kubernetes/deploy.md) may assist in an air-gapped environment. By default, the Docker images are fetched via Docker Hub (except for [Web Modeler](../../docker.md#web-modeler)).
With the dependencies in third-party Docker images and Helm charts, additional steps are required to make all charts available as outlined in this resource.

## Required Docker images

The following images must be available in your air-gapped environment:

- [camunda/zeebe](https://hub.docker.com/r/camunda/zeebe)
- [camunda/operate](https://hub.docker.com/r/camunda/operate)
- [camunda/tasklist](https://hub.docker.com/r/camunda/tasklist)
- [camunda/optimize](https://hub.docker.com/r/camunda/optimize)
- [camunda/identity](https://hub.docker.com/r/camunda/identity)
- [postgres](https://hub.docker.com/_/postgres)
- [bitnami/keycloak](https://hub.docker.com/r/bitnami/keycloak)
- [elasticsearch](https://hub.docker.com/_/elasticsearch)
- Web Modeler images (only available from [Camunda's private registry](../../docker.md#web-modeler)):
  - `web-modeler-ee/modeler-restapi`
  - `web-modeler-ee/modeler-webapp`
  - `web-modeler-ee/modeler-websockets`

## Required Helm charts

The following charts must be available in your air-gapped environment:

- [Camunda Platform Helm chart](https://github.com/camunda/camunda-platform-helm)
- [Elasticsearch Helm chart](https://github.com/elastic/helm-charts/tree/main/elasticsearch)
- [Keycloak Helm chart](https://github.com/bitnami/charts/tree/main/bitnami/keycloak)
- [Postgres Helm chart](https://github.com/bitnami/charts/tree/main/bitnami/postgresql)
- [Bitnami Common Helm chart](https://github.com/bitnami/charts/tree/main/bitnami/common)

## Dependencies explained

Identity utilizes Keycloak and allows you to manage users, roles, and permissions for Camunda Platform 8 components. This third-party dependency is reflected in the Helm chart as follows:

```
camunda-platform
    |_ elasticsearch
    |_ identity
        |_ keycloak
            |_ postgresql
    |_ zeebe
    |_ optimize
    |_ operate
    |_ tasklist
    |_ postgresql
```

- Keycloak is a dependency for Camunda Identity and PostgreSQL is a dependency for Keycloak.
- PostgreSQL is a dependency for Web Modeler.
  - This dependency is optional as you can either install PostgreSQL with Helm or use an existing [external database](../deploy.md#optional-configure-external-database).
- Elasticsearch is a dependency for Zeebe, Operate, Tasklist, and Optimize.

The values for the dependencies Keycloak and PostgreSQL can be set in the same hierarchy:

```yaml
identity:
  [identity values]
  keycloak:
    [keycloak values]
    postgresql:
      [postgresql values]
postgresql:
  [postgresql values]
```

## Push Docker images to your repository

All the [required Docker images](#required-docker-images) need to be pushed to your repository using the following steps:

1. Tag your image using the following command (replace `<IMAGE ID>`, `<DOCKER REPOSITORY>`, and `<DOCKER TAG>` with the corresponding values.)

```
docker tag <IMAGE_ID> example.jfrog.io/camunda/<DOCKER_IMAGE>:<DOCKER_TAG>
```

2. Push your image using the following command:

```
docker push example.jfrog.io/camunda/<DOCKER_IMAGE>:<DOCKER_TAG>
```

## Deploy Helm charts to your repository

You must deploy the [required Helm charts](#required-helm-charts) to your repository.
For details about hosting options, visit the [chart repository guide](https://helm.sh/docs/topics/chart_repository).

### Add your Helm repositories

You must add your Helm chart repositories to use the charts:

```
helm repo add camunda https://example.jfrog.io/artifactory/api/helm/camunda-platform
helm repo add elastic https://example.jfrog.io/artifactory/api/helm/elastic
helm repo add bitnami https://example.jfrog.io/artifactory/api/helm/bitnami
helm repo update
```

### Helm chart values

In a custom values file, it is possible to override the image repository and the image tag.

```yaml
zeebe:
  image:
    repository: example.jfrog.io/camunda/zeebe
    # e.g. work with the latest versions in development
    tag: latest
zeebe-gateway:
  image:
    repository: example.jfrog.io/camunda/zeebe
    tag: latest
elasticsearch:
  image: example.jfrog.io/elastic/elasticsearch
  imageTag: 7.16.3
identity:
  image:
    repository: example.jfrog.io/camunda/identity
    ...
  keycloak:
    image:
      repository: example.jfrog.io/bitnami/keycloak
      ...
    postgresql:
      image:
        repository: example.jfrog.io/bitnami/postgres
        ...
operate:
  image:
    repository: example.jfrog.io/camunda/operate
    ...
tasklist:
  image:
    repository: example.jfrog.io/camunda/tasklist
    ...
optimize:
  image:
    repository: example.jfrog.io/camunda/optimize
    ...
webModeler:
  image:
    # registry and tag will be used for all three Web Modeler images
    registry: example.jfrog.io
    tag: latest
  restapi:
    image:
      repository: camunda/modeler-restapi
  webapp:
    image:
      repository: camunda/modeler-webapp
  websockets:
    image:
      repository: camunda/modeler-websockets
  ...
# only necessary if the PostgreSQL chart dependency is used for Web Modeler
postgresql:
  image:
    repository: example.jfrog.io/bitnami/postgres
```

Afterwards, you can deploy Camunda Platform using Helm and the custom values file.

```
helm install my-camunda-platform camunda/camunda-platform -f values.yaml
```
