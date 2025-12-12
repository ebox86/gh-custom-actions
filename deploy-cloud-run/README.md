# deploy-cloud-run

Composite action to update an existing **Cloud Run** service to a new container image.

> This action assumes you have already authenticated and installed `gcloud`, for example via [`gcloud-wif-auth`](../gcloud-wif-auth/README.md).

## Inputs

| Name          | Required | Description |
| ------------- | -------- | ----------- |
| `project_id`  | ✅       | GCP project ID, e.g. `pittsburgh-in-progress`. |
| `region`      | ✅       | Cloud Run region, e.g. `us-east4`. |
| `service_name`| ✅       | Cloud Run service name, e.g. `pgh-ingest-api`. |
| `image`       | ✅       | Full image URL, e.g. `us-east4-docker.pkg.dev/pittsburgh-in-progress/ghcr/ebox86/pip-ingest-api:main`. |
| `platform`    | ❌       | Cloud Run platform (`managed` or `gke`). Default: `managed`. |
| `extra_args`  | ❌       | Extra flags appended to `gcloud run services update` (e.g. `--cpu=1 --memory=512Mi`). |

## Usage

Typical pattern with `gcloud-wif-auth`:

```yaml
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

