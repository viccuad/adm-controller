# Release automation

You can trigger the pipeline and open the PR from your local machine
as needed.

## Configuration Structure

The updatecli configuration uses a modular structure with shared values:

- **`values/scms.yaml`** - GitHub authentication and repository settings
- **`values/artifacts.yaml`** - Shared Kubewarden images and Helm charts definitions (used by all pipelines)
- **`values/update-deps-pr-values.yaml`** - PR configuration for weekly dependency updates
- **`values/release-pr-values.yaml`** - PR configuration for release PRs

This structure ensures a single source of truth for artifact definitions while allowing different PR configurations per pipeline.

## Running Release PR Locally

Change `updatecli/values/scms.yaml` as needed to target forks.

```console
$ cd adm-controller/
$ export UPDATECLI_GITHUB_TOKEN=<your token>
$ export UPDATECLI_GITHUB_OWNER=kubewarden
$ updatecli compose apply --file updatecli/open-release-pr.yaml

(...)

Run Summary
===========
Pipeline(s) run:
  * Changed:    1
  * Failed:     0
  * Skipped:    0
  * Succeeded:  0
  * Total:      1

One action to follow up:
  * https://github.com/kubewarden/adm-controller/pull/XXX
```

## Dependency Updates

### Renovate

Renovate updates:
- Policy image tags in the Helm chart values
- kuberlr-kubectl image
- openreports chart

Renovate puts each update into one PR, which changes the Helm chart values,
`charts/admission-controller/Chart.yaml`, `Chart.lock`, the openreports archive
in `charts/admission-controller/charts/`, and the Hauler manifest together.

### Weekly Updatecli Dependency Updates

via `.github/workflows/update-dependencies.yaml`, Updatecli updates:
- The policy-reporter chart version
- Hauler manifest with the Kubewarden and policy-reporter versions
- The `artifacthub.io/images` annotation in `charts/admission-controller/Chart.yaml`

Renovate cannot do these updates. The policy-reporter and policy-reporter-ui
image tags come from the policy-reporter chart, and Renovate cannot read them
from the new chart version.

### Running Updates Manually
For manual runs of all dependency updates:
```console
$ export UPDATECLI_GITHUB_TOKEN=<your token>
$ export UPDATECLI_GITHUB_OWNER=kubewarden
$ updatecli compose diff --file updatecli/update-deps.yaml
```

### Adding New Dependencies

To add a new container image or Helm chart, add a custom manager or a
package rule to `renovate.json`. Use updatecli only for a dependency that
Renovate cannot update.
