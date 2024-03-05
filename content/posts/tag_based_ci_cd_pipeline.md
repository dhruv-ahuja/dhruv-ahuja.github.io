+++
title = "Tag-Based Python CI/CD Pipeline"
description = "A guide to setting up a CI/CD pipeline for Python using GitHub Actions, that runs on Git tag pushes. Also includes a step to handle CI pipeline failures through a step that allows SSHing into the workflow runner instance."
date = "2024-03-05"
+++

## Introduction

I recently setup a CI/CD pipeline using [GitHub Actions](https://docs.github.com/en/actions/quickstart "https://docs.github.com/en/actions/quickstart"), to automate code quality management, testing and Docker image deployment for [my Python webapp](https://github.com/dhruv-ahuja/backend_burger "https://github.com/dhruv-ahuja/backend_burger"). The CI workflow triggers on every commit to the `main` branch and formats, lints and tests the code. It uses a Redis [service container](https://docs.github.com/en/actions/using-containerized-services/about-service-containers "https://docs.github.com/en/actions/using-containerized-services/about-service-containers") since the integration tests call the API endpoints, which use a caching layer before accessing the database. It also uses an action step to debug failures. The CD workflow runs on new tag pushes to the repository. Both workflows can also be run manually.

Let’s get started with the setup.

## Initial Setup

Create a Docker Hub repository to push the images to, and generate a `read-write` scope Access Token for use with the workflows. Copy the token for use in the next step.

Next, setup environment secrets so that our application can access these values during the testing step. Go to the `Settings` → `Secrets and variables` → `Actions` panel in the GitHub repository and define the any repository secrets required during the workflow’s execution. Also define the Docker username and password secrets here. Use the access token generated above for `DOCKER_PASSWORD`.

![Manage Repository Secrets](/images/repository_secrets.png)

## Creating the CI Workflow

Create a `.github/workflows` folder in your local codebase and a `ci.yml`  file, adding the following code at the top of the file:

```yaml
name: CI

on:
    push:
        branches: main
    
    workflow_dispatch: {}

concurrency:
    group: ${{ github.workflow }}-${{ github.ref }}
    cancel-in-progress: true
```

This defines that the `CI` workflow runs only when code is pushed to the `main` branch. `workflow_dispatch: {}`  allows us to run the workflow manually from the `Actions` page. Our `concurrency` configuration ensures that the workflow’s runs are grouped together under one Git reference value and that only one run happens at a time. If a workflow is underway and another is triggered, the current run is cancelled in favour of the newer run.

Next, define the list of environment variables required by the application like so:

```yaml
env:
    DB_URL: ${{secrets.DB_URL}}
    JWT_SECRET_KEY: ${{secrets.JWT_SECRET_KEY}}
    AWS_ACCESS_KEY_ID: ${{secrets.AWS_ACCESS_KEY_ID}}
    AWS_SECRET_ACCESS_KEY: ${{secrets.AWS_SECRET_ACCESS_KEY}}
    AWS_REGION_NAME: ${{secrets.AWS_REGION_NAME}}
    SQS_QUEUE_NAME: ${{secrets.SQS_QUEUE_NAME}}
    S3_BUCKET_URL: ${{secrets.S3_BUCKET_URL}}
    S3_BUCKET_NAME: ${{secrets.S3_BUCKET_NAME}}
    S3_LOGS_FOLDER: ${{secrets.S3_LOGS_FOLDER}}
    REDIS_HOST: localhost
```

We define environment variables by reading the repository secrets we set in the initial setup section, with the exception of `REDIS_HOST`, which is set to `localhost` to enable our application access to the Redis service.

Now comes the main part for the CI logic, the job itself:

```yaml
jobs:
    build:
        runs-on: ubuntu-latest
        
        services:
            # Label used to access the service container
            redis:
                image: redis
                # Set health checks to wait until redis has started
                options: >-
                    --health-cmd "redis-cli ping"
                    --health-interval 10s
                    --health-timeout 5s
                    --health-retries 5

                ports:
                    - 6379:6379

        steps:
            - name: Checkout
              uses: actions/checkout@v4

            - name: Setup Python
              uses: actions/setup-python@v4
              with:
                python-version: "3.11"
                cache: "pip"
            
            - name: Install PyCurl Dependencies
              run: 
                sudo apt-get update && sudo apt-get install -y curl libcurl4-openssl-dev build-essential libssl-dev

            - name: Install Dependencies
              run:
                python -m pip install --upgrade pip
                pip install -r requirements.txt
            
            - name: Test code
              run: 
                pytest . -s -v -W ignore
            
            - name: Check Code Formatting
              run:
                ruff format --line-length=120 --check . 
            
            - name: Check Code Linting
              run: 
                ruff check .
            
            - name: Setup Tmate Session
              if: ${{/* failure() */}}
              uses: mxschmitt/action-tmate@v3
```

Let’s walk through the job’s specifics, step-by-step.

The Redis service logic sets up a Redis container with health check options to ensure that the workflow waits for it to boot up, and exposes port `6379` to make it accessible to the application when we run the tests.

`Checkout` makes the repository’s code available to the workflow, and `Setup Python` setups the specific Python version — `3.11` in our case — and caches the dependencies installed by `pip` to make future workflow runs faster. `Install Pycurl Dependencies` installs the dependencies required by the `pycurl` Python library on Ubuntu. The following step installs the Python dependencies used by our application.

The code testing step runs the `pytest` test suite gathers and runs all tests in the current directory. My project has a few unit tests and integration tests for each API endpoint. The `-s` flag outputs any Python print statements to the stdout stream, and `-v` runs the tests in verbose mode, giving us a detailed overview of the ongoing tests. I have added `-W ignore` to ignore the warnings emitted during the execution of the tests, primarily to help avoid the `Pydantic v1 deprecation warnings` issued for third party libraries.

I am using `Ruff` as the formatter and linter of choice, it is very fast and I feel that it has good linting rules without being overly restrictive. It is easy to setup formatting, lint and type checks in editors and is a one-time setup and I feel that it really helps keep the codebase maintainable in the long run.

The next two steps check for formatting and lint errors in the code and stop the workflow in case of any errors. This ensures that contributing developers adhere to Ruff’s code quality standards.

The last step is optional, and only runs if any of the previous steps fails. It allows us to ssh into the currently ongoing workflow session to check the environment and debug issues. Be careful though, it kept running for quite a while since I forgot to cancel the workflow run manually.  I am not sure if it has a time limit or it keeps running indefinitely.

## Creating the CD workflow

 The CD pipeline is quite straightforward:

```yaml
name: CD

on:
    push:
        tags: "v*"

    # allow manually triggering this workflow
    workflow_dispatch: {}

concurrency:
    group: ${{ github.workflow }}-${{ github.ref }}
    cancel-in-progress: true

jobs:
    deploy:
        runs-on: ubuntu-latest

        steps:
            - name: Checkout
              uses: actions/checkout@v4

            - name: Log into Docker Hub
              uses: docker/login-action@v3
              with:
                username: ${{secrets.DOCKER_USERNAME}}
                password: ${{secrets.DOCKER_PASSWORD}}

            - name: Get Latest Tag
              id: latest-tag
              uses: "WyriHaximus/github-action-get-previous-tag@v1"
              with:
                fallback: latest

            - name: Build and push Docker image
              uses: docker/build-push-action@v5
              with:
                push: true
                tags: dhruvahuja/backend_burger:${{ steps.latest-tag.outputs.tag }}
                labels: ${{steps.latest-tag.outputs.tag}}
```

We define the workflow to either run manually or when a new tag prefixed by `v` is pushed to the repository, example. `v0.0.1`. `Checkout` is required to allow getting git tag in the third step. The next step reads the Docker username and password from repository secrets and logs us into Docker Hub. `Get Latest Tag` reads the tag which was just pushed, if the workflow was triggered by a tag push, otherwise defaulting to `latest`.

The final step builds the Docker image with the version tag and label, and pushes it to the Docker repository URL, defined in the `tags` directive. In this case, `dhruvahuja/backend_burger`.

## Conclusion

That’s it, our CI/CD pipeline is ready! There are Actions available for all sort of use-cases, and you can remove or add steps according to your needs. For example, you may choose to ssh into a server after the `Build and push Docker image` step to pull and run the new image. I did not add this particular step since I did not have a need for it at the moment.

I chose the tag-based approach for the deployment process since I wanted to deploy new images only on specific milestones, which I can manage with version tags.
