# gh-custom-actions

<img src="./icon.svg" alt="gh-custom-actions icon" width="96" height="96" />

A small collection of reusable **GitHub composite actions** I use across projects — mostly for GCP and deployment workflows.

Each action lives in its own directory and can be used via:
```
    uses: ebox86/gh-custom-actions/<action-name>@v1
```
---

## Actions

| Category     | Action name       | Path                 | Description |
| ------------ | ----------------- | -------------------- | ----------- |
| GCP / Auth   | `gcloud-wif-auth` | `./gcloud-wif-auth`  | Authenticate to Google Cloud using **Workload Identity Federation** and install `gcloud`. |
| GCP / Deploy | `deploy-cloud-run`| `./deploy-cloud-run` | Update an existing Cloud Run service to a new container image (assumes `gcloud` is already authenticated). |

---

## `gcloud-wif-auth` (GCP / Auth)

Authenticate to Google Cloud using **Workload Identity Federation** and set up the `gcloud` CLI.

Typical usage in a workflow:
```
    jobs:
      deploy:
        runs-on: ubuntu-latest
        permissions:
          contents: read
          id-token: write   # required for Workload Identity

        steps:
          - uses: actions/checkout@v4

          - name: GCP auth (Workload Identity)
            uses: ebox86/gh-custom-actions/gcloud-wif-auth@v1
            with:
              project_id: pittsburgh-in-progress
              workload_identity_provider: ${{ secrets.GCP_PGH_WIP }}
              service_account: ci-runner@pittsburgh-in-progress.iam.gserviceaccount.com
```
After that step, `gcloud` is installed and authenticated for the given project.

---

## `deploy-cloud-run` (GCP / Deploy)

Update an existing Cloud Run service to a new container image.

This action assumes you already ran `gcloud-wif-auth` (or otherwise authenticated and installed `gcloud`).

Example:
```
    jobs:
      deploy:
        runs-on: ubuntu-latest
        permissions:
          contents: read
          id-token: write

        steps:
          - uses: actions/checkout@v4

          - name: GCP auth (Workload Identity)
            uses: ebox86/gh-custom-actions/gcloud-wif-auth@v1
            with:
              project_id: pittsburgh-in-progress
              workload_identity_provider: ${{ secrets.GCP_PGH_WIP }}
              service_account: ci-runner@pittsburgh-in-progress.iam.gserviceaccount.com

          - name: Deploy pgh-ingest-api
            uses: ebox86/gh-custom-actions/deploy-cloud-run@v1
            with:
              project_id: pittsburgh-in-progress
              region: us-east4
              service_name: pgh-ingest-api
              image: us-east4-docker.pkg.dev/pittsburgh-in-progress/ghcr/ebox86/pip-ingest-api:main
```
---

## Example end-to-end flow

A full build + deploy workflow might look like this in a backend repo:
```
    name: Build and deploy ingest-api

    on:
      push:
        branches: [ main ]
        paths:
          - 'ingest-api/**'
          - '.github/workflows/ingest-api-build.yml'

    jobs:
      build-and-deploy:
        runs-on: ubuntu-latest
        permissions:
          contents: read
          id-token: write

        steps:
          - uses: actions/checkout@v4

          - name: Build JAR
            working-directory: ingest-api
            run: mvn -B -DskipTests package

          - name: Build & push Docker image
            run: |
              echo "${{ secrets.GITHUB_TOKEN }}" \
                | docker login ghcr.io -u "${{ github.actor }}" --password-stdin
              docker build -t ghcr.io/ebox86/pip-ingest-api:main ingest-api
              docker push ghcr.io/ebox86/pip-ingest-api:main

          - name: GCP auth (Workload Identity)
            uses: ebox86/gh-custom-actions/gcloud-wif-auth@v1
            with:
              project_id: pittsburgh-in-progress
              workload_identity_provider: ${{ secrets.GCP_PGH_WIP }}
              service_account: ci-runner@pittsburgh-in-progress.iam.gserviceaccount.com

          - name: Deploy Cloud Run
            uses: ebox86/gh-custom-actions/deploy-cloud-run@v1
            with:
              project_id: pittsburgh-in-progress
              region: us-east4
              service_name: pgh-ingest-api
              image: us-east4-docker.pkg.dev/pittsburgh-in-progress/ghcr/ebox86/pip-ingest-api:main
```
---

## Versioning

When you’re happy with a change to an action:

1. Tag it, e.g. `v1.0.0`.
2. Optionally create a floating `v1` tag that points to the latest `v1.x`.

Then your consumers can use:
```
    uses: ebox86/gh-custom-actions/gcloud-wif-auth@v1
```
and get new bugfixes without changing their workflow.

