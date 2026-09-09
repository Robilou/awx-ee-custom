# awx-ee-custom

Custom AWX Execution Environment (EE) image, extending the official `awx-ee` base image with extra Python
dependencies and Ansible collections needed by playbooks that don't ship with the stock image (e.g. `ovh`
for `kube_cloud.ovh` modules).

## What's inside

The actual list of dependencies lives in
[`execution-environment/execution-environment.yml`](execution-environment/execution-environment.yml) — that file
is the source of truth, not this README. Open it to see the current base image, Python packages and Galaxy
collections baked into the image.

## How the image is built and published

The build is fully automated via [`.github/workflows/build-ee.yml`](.github/workflows/build-ee.yml) and only
triggers when `execution-environment/**` changes.

| Trigger                                                | Image tags pushed                       |
|---------------------------------------------------------|------------------------------------------|
| Push to `develop`                                        | `dev`, `dev-<short-sha>`                  |
| Push to `main` (i.e. after a PR merge)                    | `latest`, `<short-sha>`                   |
| Manual run (`workflow_dispatch`) with a `tag` input        | that exact tag                            |
| Pull request into `main`/`develop`                         | nothing pushed — just verifies the image still builds (`validate` job, used as a required status check) |

Image is published to:

```
ghcr.io/robilou/awx-ee-custom
```

`main` is protected — changes only land there via pull request, and the `validate` build check must pass first.

## Building locally

```bash
pip install ansible-builder

cd execution-environment
ansible-builder build \
  -f execution-environment.yml \
  -t awx-ee-custom:local \
  --container-runtime=docker
```

Verify before relying on it:

```bash
docker run --rm awx-ee-custom:local python3 -c "import ovh; print(ovh.__version__)"
docker run --rm awx-ee-custom:local ansible-doc kube_cloud.ovh.dns_record
```

## Using this image in AWX

1. **Administration → Execution Environments → Add**
   - Name: e.g. `awx-ee-custom`
   - Image: `ghcr.io/robilou/awx-ee-custom:latest` (or pin to a specific `<short-sha>`/tag for reproducibility)
   - Pull: `Always` if tracking `latest`, otherwise `Missing` is fine for pinned tags
   - If the GHCR package is private, add a **Container Registry** credential (GHCR token with `read:packages`)
2. Set it as default on the relevant **Organization**, or assign it directly on a **Project**/**Job Template**.

## Adding or updating a dependency

1. Edit `execution-environment/execution-environment.yml` on a branch off `develop`.
2. Push to `develop` first if you want to test against a `dev`-tagged image before promoting it.
3. Open a pull request into `main`.
4. Wait for the `validate` check to pass, then merge — `latest` gets rebuilt automatically.
