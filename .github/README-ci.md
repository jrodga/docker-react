# CI/CD with GitHub Actions – Sparta App

This document describes the **CI (Continuous Integration)** setup for the Sparta Node.js application using **GitHub Actions**, and explains how workflow updates are applied.

##

### Purpose of This CI Pipeline

The GitHub Actions pipeline is used to:

- Automatically run when code is pushed to main

- Build a Docker image for the application

- Run tests inside a Docker container

- Validate changes before deployment

This ensures that every change pushed to the repository is tested in a clean, reproducible environment.

##
### Workflow Trigger
The pipeline is triggered on every push to the `main` branch:

```yml
on:
  push:
    branches:
      - main
```
This follows standard CI practice where the main branch always represents a stable state.
##
### Docker Hub Authentication
The pipeline logs into Docker Hub using GitHub Secrets.

Secrets used:

`DOCKERHUB_USERNAME`

`DOCKERHUB_TOKEN`

Authentication is handled using the official Docker GitHub Action:
```yml
- name: Login to DockerHub
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```

This avoids interactive login, which is not supported in CI environments.
##
### Build and Test Stages
**Build Docker Image**

The application image is built using the development Dockerfile:
```yml
- name: Build Docker image
  run: docker build -t jrodga/docker-react -f Dockerfile.dev .
```
This ensures consistency between local development and CI.
##
### run tests

tests are executed inside the built docker container.
```yml
- name: Run tests
  run: docker run -e CI=true jrodga/docker-react npm test
```

Running tests inside the container guarantees that tests run in the same environment as the application.

##

### Important Note: Updating a Failing Workflow

When a workflow fails and the YAML file is updated, GitHub Actions does not automatically re-run the job using the new configuration.

This is because:

- Each workflow run uses the workflow file from the commit that triggered it

- Re-running a failed job may still use the old configuration
##

### How to Apply Workflow Changes Correctly

To ensure GitHub Actions uses the **latest workflow version**, a new commit must be pushed.

**Recommended approach**

Make any small change (for example updating documentation), then:

```bash
git add .
git commit -m "Update CI workflow"
git push
```
This triggers a new workflow run using the updated **YAML** file.

