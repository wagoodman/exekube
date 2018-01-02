⚠️ This is a work in progress. Don't attempt to use it for anything except developing Exekube (or inspiration).

# Exekube

*Exekube* is a declarative framework for administering Kubernetes clusters and deploying containerized software onto them.

## Introduction

You only need [Docker CE](/) and [Docker Compose](/) on your local machine to begin using Exekube. The framework is a thin layer on top of several open-source DevOps tools:

- Docker Compose (for our local development environment)
- Terraform and HCL (HashiCorp Language)
- Kubernetes
- Helm

Exekube allows you to manage both cloud infrastructure resources and Kubernetes resources using a git-based workflow with a continuous integration (CI) pipeline.

📘 Read the companion guide: <https://github.com/ilyasotkov/learning-kubernetes/>

- [Introduction](#introduction)
- [Design principles](#design-principles)
- [Setup and usage](#setup-and-usage)
	- [Requirements starting from zero](#requirements-starting-from-zero)
		- [Linux](#linux)
		- [macOS](#macos)
		- [Windows](#windows)
	- [Usage step-by-step](#usage-step-by-step)
		- [Cloud provider setup: do it once](#cloud-provider-setup-do-it-once)
		- [Cluster setup: do it as often as you need](#cluster-setup-do-it-as-often-as-you-need)
	- [Workflows](#workflows)
		- [Legacy imperative workflow (CLI)](#legacy-imperative-workflow-cli)
		- [Declarative workflow (HCL .tf files)](#declarative-workflow-hcl-tf-files)
- [Feature tracker](#feature-tracker)
	- [Cloud provider and local environment setup](#cloud-provider-and-local-environment-setup)
	- [Cloud provider config](#cloud-provider-config)
	- [Cluster creation](#cluster-creation)
	- [Cluster access control](#cluster-access-control)
	- [Supporting tools](#supporting-tools)
	- [User apps and services](#user-apps-and-services)

## Design principles

- Everything on client side runs in a Docker container
- Infrastructure (cloud provider) objects and Kubernetes API objects are expressed as declarative code, using HCL (HashiCorp Language) and Helm packages (YAML + Go templates)
- Modular design
- Git-based workflow with a CI pipeline [TBD]
- No vendor lock-in, choose any cloud provider you want [only GCP for now]
- Test-driven (TDD) or behavior-driven (BDD) model of development [TBD]

## Setup and usage

### Requirements starting from zero

The only requirements, depending on your local OS:

#### Linux

- [Docker](/)
- [Docker Compose](/)

#### macOS

- [Docker for Mac](/)

#### Windows

- [Docker for Windows](/)

### Usage step-by-step

#### Cloud provider setup: do it once

1. Create `xk` (stands for "exekube") alias for your shell session (or save to ~/.bashrc):
    ```bash
    alias xk="docker-compose run --rm exekube"
    ```
2. [Set up a Google Account](https://console.cloud.google.com/) for GCP (Google Cloud Platform), create a project named "ethereal-argon-186217", and enable billing.
3. [Create a service account](/) in GCP Console GUI, give it project owner permissions.
4. [Download JSON credentials](/) ("key") to repo root directory and rename the file to `credentials.json`.
5. Use JSON credentials to activate service account:
    ```sh
    xk gcloud auth activate-service-account --key-file credentials.json
    ```
6. Create Google Cloud Storage bucket (with versioning) for our Terraform remote state:
    ```sh
    xk gsutil mb -p ethereal-argon-186217 gs://ethereal-argon-terraform-state \
        && xk gsutil versioning set on gs://ethereal-argon-terraform-state
    ```

#### Cluster setup: do it as often as you need

7. Edit code in `live` and `modules` directories

    [Guide to HCL, Terraform, and Exekube directory structure](/) [TODO]
8. Initialize terraform and create the cluster:
    ```sh
    xk init live/infra/gcp-ethereal-argon/
    xk apply live/infra/gcp-ethereal-argon/

    # Make the cluster dashboard available at localhost:8001/ui
    docker-compose up -d
    # Disable local dashboard: docker-compose down
    ```
9. Deploy *core tools* (nginx-ingress-controller, kube-lego):
    ```sh
    xk init live/kube/core/
    xk apply live/kube/core/
    ```
10. Deploy *continuous integration tools*:
    ```sh
    xk init live/kube/ci/
    xk apply live/kube/ci/
    ```

#### Cleanup

```sh
xk destroy live/kube/ci \
&& xk destroy live/kube/core \
&& xk destroy live/infra/gcp-ethereal-argon
```

### Workflows

#### Legacy imperative workflow (CLI)

⚠️ These tools are relatively mature and work well, but are considered *legacy* here since this framework aims to be [declarative](/)

Command line tools `kubectl` and `helm` are known to those who are familiar with Kubernetes. `gcloud` (part of Google Cloud SDK) is used for managing the Google Cloud Platform.

- `xk gcloud`
- `xk kubectl`
- `xk helm`

```sh
xk gcloud auth list

xk kubectl get nodes

xk helm install --name cluster-proxy \
        -f live/kube/core/values/ingress-controller.yaml \
        stable/nginx-ingress
```

#### Declarative workflow (HCL .tf files)

- `xk apply` / `xk destroy` (Terraform wrapper)

Declarative tools are exact equivalents of the legacy imperative (CLI) toolset, except everything is implemented as a [Terraform provider plugin](/) and expressed as declarative HCL (HashiCorp Language) code. Instead of writing CLI commands like `xk helm install --name <release-name> -f <values> <chart>` for each individual Helm release, we install all releases simultaneously by running `xk apply`.

## Feature tracker

Features are marked with ✔️ when they enter the *alpha stage*, meaning a minimum viable solution has been implemented

### Cloud provider and local environment setup

- [x] Create GCP account, enable billing in GCP Console (web GUI)
- [x] Get credentials for GCP (`credentials.json`)
- [x] Authenticate to GCP using `credentials.json` (for `gcloud` and `terraform` use)
- [x] Enable terraform remote state in a Cloud Storage bucket

### Cloud provider config

- [ ] Create GCP Folders and Projects and associated policies
- [x] Create GCP IAM Service Accounts and IAM Policies for the Project

### Cluster creation

- [x] Create the GKE cluster
- [x] Get cluster credentials (`/root/.kube/config` file)
- [x] Initialize Helm

### Cluster access control

- [x] Add cluster namespaces (virtual clusters)
- [ ] Add cluster roles and role bindings
- [ ] Add cluster network policies

### Supporting tools

- [x] Install cluster ingress controller (cloud load balancer)
- [x] Install TLS certificates controller (kube-lego)
- [ ] Install Continuous Delivery tools
    - [x] Continuous Delivery service (Drone / Jenkins)
    - [ ] Git service (Gitlab / Gogs)
- [ ] Monitoring and alerting tools (Prometheus / Grafana)

### User apps and services

- [ ] Install "hello-world" apps like static sites, Ruby on Rails apps, etc.
