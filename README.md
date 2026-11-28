# Java App CI/CD

This repo is part of my final project for the DevOps course.

The app is a small Spring Boot API that I took from Hassan Eid's training repo.
My part is the pipeline that builds it and deploys it.

## The app

It runs on port 8090 and has two endpoints:

- GET /persons/all
- GET /persons/{id}

To run it locally:

    mvn clean package
    java -jar target/demo1-0.0.1-SNAPSHOT.jar

## The pipeline

The workflow is in .github/workflows/ci-cd.yml. I run it by hand from the Actions tab.

The first job builds the JAR with Java 11 and uploads it as an artifact.
Then it builds the Docker image and pushes it to Docker Hub as shadydex1/java-app.

The second job checks out my java-cd repo and changes the image tag in
shady-magdy/deployment.yaml. It commits and pushes, and ArgoCD deploys the new version.

I added concurrency so two runs don't push at the same time.
If I start it twice, the second one waits.

## Secrets

Both jobs use the GitHub environment shady-magdy. It has three secrets:

- DOCKER_USERNAME is my Docker Hub username
- DOCKER_PASSWORD is a Docker Hub access token
- CD_REPO_SSH_KEY is a deploy key that can push to java-cd

I used a deploy key because the default GITHUB_TOKEN can't push to another repo.
