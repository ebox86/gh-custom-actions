# gcloud-wif-auth

Reusable composite action to authenticate to Google Cloud using **Workload Identity Federation** and set up the `gcloud` CLI.

This wraps the boilerplate:

- `google-github-actions/auth@v2`
- `google-github-actions/setup-gcloud@v2`

so your workflows can stay focused on the actual deploy/build steps.

## Inputs

| Name                     | Required | Description |
| ------------------------ | -------- | ----------- |
| `project_id`             | ✅       | GCP project ID, e.g. `pittsburgh-in-progress`. |
| `workload_identity_provider` | ✅   | Full Workload Identity Provider resource name, e.g. `projects/123456789012/locations/global/workloadIdentityPools/gh-pool/providers/gh-provider`. |
| `service_account`        | ✅       | Service account email to impersonate, e.g. `ci-runner@pittsburgh-in-progress.iam.gserviceaccount.com`. |
| `gcloud_version`         | ❌       | Version of `gcloud` to install (default: `latest`). |

## Usage

In a workflow in another repo (e.g. `pittsburgh-in-progress-backend`):

```yaml
name: Build and deploy

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write   # required for Workload Identity

    steps:
      - uses: actions/checkout@v4

      - name: Authenticate to Google Cloud (WIF)
        uses: ebox86/gh-custom-actions/gcloud-wif-auth@v1
        with:
          project_id: pittsburgh-in-progress
          workload_identity_provider: ${{ secrets.GCP_PGH_WIP }} # e.g. projects/123.../locations/global/workloadIdentityPools/gh-pool/providers/gh-provider
          service_account: ci-runner@pittsburgh-in-progress.iam.gserviceaccount.com

      - name: Deploy Cloud Run service
        run: |
          gcloud run services update pgh-ingest-api \
            --platform=managed \
            --region=us-east4 \
            --image=us-east4-docker.pkg.dev/pittsburgh-in-progress/ghcr/ebox86/pip-ingest-api:main \
            --quiet
```
Recommended secrets:

`GCP_PGH_WIP` – the full Workload Identity Provider resource string.

`GCP_PGH_PROJECT_NUMBER` – project number, if you need it for IAM bindings elsewhere.

---


