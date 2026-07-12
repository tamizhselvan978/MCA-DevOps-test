# MCA DevOps Test — Solution

We have a classic N-tier application:

* a [backend](./backend/): a Java/SpringBoot 3/Maven application exposing endpoints to list and add users
* a [frontend](./frontend/): an Angular application to interact with the backend
* a PostgreSQL database, deployed inside the cluster (see **Database** note below)

## What was used

* **Cluster**: minikube (Docker driver)
* **Deployment method**: plain `kubectl` YAML manifests, no Helm — simplest option for two services
* **Ingress**: nginx (minikube's `ingress` addon) — routes `/api` to the backend and `/` to the frontend, under one address
* **Images**: built with custom multistage Dockerfiles, pushed to Docker Hub, pulled by Kubernetes like any real deployment
* **DB credentials**: exposed as plain environment variables — `SPRING_POSTGRES_URL`, `SPRING_POSTGRES_USERNAME`, `SPRING_POSTGRES_PASSWORD` — no Secret, since the exercise says there's no need to hide them

## Why

* **minikube + plain YAML** — enough for two services; Helm would add templating complexity with no real benefit here
* **Ingress instead of just Services** — the frontend calls `/api/...` as a relative path, so frontend and backend need to appear under one origin; the Ingress is what provides that
* **Multistage Dockerfiles** — keeps the final images small (no Maven/JDK or Node in the runtime image, just the compiled jar / built static files)
* **Docker Hub instead of `minikube image load`** — works the same way a real cluster would pull images, not just a local shortcut

## Database

The exercise gives `my-db-test.io` as an example address — it isn't a real, reachable host. Pointing the app at it as-is causes the backend to correctly read the env vars and attempt a connection, but fail with `UnknownHostException`, since nothing exists there.

Instead of leaving that as a permanent crash loop, this repo runs a small Postgres Deployment inside the cluster (`k8s/postgres-deployment.yaml`) and points `SPRING_POSTGRES_URL` at that instead:

```
jdbc:postgresql://postgres:5432/myapplication
```

Swap that one value for a real external DB and nothing else needs to change.

## Repo layout

```
backend/     Spring Boot app (source unchanged) + Dockerfile
frontend/     Angular app (source unchanged) + Dockerfile + nginx.conf
k8s/          All Kubernetes manifests
```

## Bonus tasks

* **Bonus 1** (reverse proxy) — done, via the nginx Ingress
* **Bonus 2** (own Dockerfiles) — done, for both backend and frontend
* **Bonus 2.1** (multistage build) — done, for both

