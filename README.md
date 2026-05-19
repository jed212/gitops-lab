# GitOps CD
This repository demonstrates a GitOps-based continuous delivery system integrated with a secure CI pipeline for a Go microservice.

It forms part of a broader DevSecOps workflow where application changes are automatically built, scanned, containerized, and deployed into a Kubernetes cluster using GitOps principles.

## Architecture Overview

This project is split into two connected repositories:

1. Secure CI Pipeline (Application Repo)
* Builds and tests a Go application
* Performs security scanning (SAST, SCA, secrets detection, container scanning)
* Generates SBOM (Software Bill of Materials)
* Builds and pushes Docker images to GitHub Container Registry (GHCR)
* Updates this GitOps repository with new image versions
2. GitOps Repository (This Repo)
* Stores Kubernetes manifests declaratively
* Managed by FluxCD
* Automatically reconciles desired state into a Kind Kubernetes cluster
* Receives updates from the CI pipeline via automated commits

## End-to-End Flow

The system works as follows:

1. Code Commit

    Developer makes a change/pushes code to the application repository [secure-ci-pipeline](https://github.com/jed212/secure-ci-pipeline).

2. CI Pipeline Execution

   The pipeline:
      
      * Runs unit tests (go test)
      * Performs linting (golangci-lint)
      * Runs static analysis (go vet, formatting checks)
      * Executes security scans:
        * Gitleaks (secret detection)
        * Trivy (filesystem + container vulnerability scanning)
      * Builds Docker image
      * Tags image using unique build identifiers (SHA + timestamp)
      * Pushes image to GHCR
      * Generates SBOM for supply chain transparency
  
3. GitOps Update
   
   After a successful build: 
      * CI clones this GitOps repository
      * Updates Kubernetes deployment manifest with the new image tag
      * Commits and pushes changes back to GitHub
  
4. FluxCD Reconciliation

   FluxCD continuously watches this repository:

    * Detects manifest changes
    * Applies updates to the Kubernetes cluster
    * Triggers rolling updates of the application


## Kubernetes Deployment

The application is deployed using:

* Kubernetes Deployments
* Services

A local Kubernetes cluster is used for testing via kind.


## Repository Relationship
  
| Component | Repository |
| ------------------------ | ------------------- |
| Application + CI pipeline| secure-ci-pipeline  |
|  GitOps manifests (this repo) | gitops-lab     |

The CI pipeline automatically updates this repository after every successful build.

## Current Status

* CI pipeline fully functional
*  Security scanning integrated
* SBOM generation working
* CR image publishing working
* GitOps repository auto-updated from CI
* FluxCD connected to cluster
* Automated deployments working
