# Java App - CI/CD with GitHub Actions and ArgoCD

This is part of my final project for the DevOps course at Route.

The app itself is a small Spring Boot API (taken from Hassan Eid's training repo).
My work here is the pipeline around it: building it, packaging it as a Docker image,
pushing it to Docker Hub, and updating the CD repo so ArgoCD deploys the new version.

## App

- Java 11, Spring Boot, Maven
- Runs on port 8090
- Endpoints:
  - `GET /persons/all`
  - `GET /persons/{id}`

Run it locally:

    mvn clean package
    java -jar target/demo1-0.0.1-SNAPSHOT.jar

## Pipeline

Workflow file: `.github/workflows/ci-cd.yml`

It runs manually from the Actions tab (`workflow_dispatch`) and has two jobs.

**1. Build and push image**
- checkout the repo
- set up Java 11 (temurin)
- build the JAR with `mvn clean package`
- upload the JAR as an artifact (`demo1-jar`)
- build the Docker image for amd64 and arm64 (I deploy on minikube on an M1 Mac, so arm64 was needed)
- push it to Docker Hub as `shadydex1/java-app:<run-number>-<short-sha>` and `latest`

**2. Update CD repo**
- checkout [java-cd](https://github.com/shadymagdy-devops/java-cd)
- replace the image tag in `shady-magdy/deployment.yaml`
- commit and push, then ArgoCD picks up the change

`concurrency` is set so only one run happens at a time. If I trigger it twice,
the second run waits instead of both pushing at once.

## Secrets

Everything is stored in the GitHub environment `shady-magdy`:

| Secret | What it is |
|---|---|
| `DOCKER_USERNAME` | Docker Hub username |
| `DOCKER_PASSWORD` | Docker Hub access token (not my real password) |
| `CD_REPO_SSH_KEY` | deploy key with write access to java-cd only |

I used a deploy key for the CD repo because the default `GITHUB_TOKEN`
can only push to the repo the workflow runs in.

## Dockerfile

I changed the base image from `openjdk:11.0.11-slim` to `eclipse-temurin:11-jre`
because the openjdk images are deprecated, and temurin has an arm64 build.

## Deployment

The Kubernetes files are in the [java-cd](https://github.com/shadymagdy-devops/java-cd) repo
under `shady-magdy/` (Deployment with 2 replicas + NodePort service on 30090).
ArgoCD watches that folder with auto sync and self heal enabled.

## Author

Shady Magdy - [GitHub](https://github.com/shadymagdy-devops)
