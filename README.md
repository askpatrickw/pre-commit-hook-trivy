# pre-commit-hook-trivy

This repository publishes a set of reusable [pre‑commit](https://pre-commit.com/) hooks based on
[Aqua Security Trivy](https://trivy.dev/). Trivy is an open‑source security scanner that can find
package vulnerabilities, infrastructure‑as‑code misconfigurations, secrets and license problems in
container images, filesystems and code repositories. By using these hooks you can enforce
security checks locally and in CI/CD pipelines without maintaining custom scripts.

## Available hooks

The hooks defined in `.pre-commit-hooks.yaml` mirror Trivy’s targets and allow you to choose the
appropriate scanner for each workflow:

| ID | Description |
| --- | --- |
| **trivy-fs** | Scans the repository’s filesystem for vulnerabilities, secrets and license issues. |
| **trivy-config** | Scans IaC files (Terraform, Kubernetes, Dockerfile, etc.) for misconfigurations. |
| **trivy-repo** | Performs a full repository scan (vulnerabilities, misconfigurations, secrets, licenses). |
| **trivy-image-registry** | Scans a container image pulled from a registry (requires image reference via args). |
| **trivy-image-daemon** | Scans a locally built image by mounting the host’s Docker daemon into the Trivy container. |

All hooks pin Trivy to version `0.67.2` for reproducibility. The container image is downloaded
automatically when the hook runs; the vulnerability database is cached in `.pre-commit-trivy-cache`
inside your repository.

## Usage

1. Ensure you have [pre‑commit](https://pre-commit.com/) installed in your development environment.

2. Add this repository to your `.pre-commit-config.yaml` and select the hooks you want to run. For
   example:

   ```yaml
   repos:
     - repo: https://github.com/askpatrickw/pre-commit-hook-trivy
       rev: v0.1.0  # replace with the git tag or commit you wish to pin
       hooks:
         - id: trivy-fs
           args: [--scanners, vuln,secret,license, --severity, HIGH,CRITICAL, --ignore-unfixed]
         - id: trivy-config
           args: [--severity, HIGH,CRITICAL]
         - id: trivy-repo
           stages: [push]
           args: [--scanners, vuln,misconfig,secret,license]
         - id: trivy-image-registry
           stages: [manual]
           args: [--scanners, vuln,license, alpine:3.20]
   ```

3. Optional: create a `trivy.yaml` file in the root of your project to centralise
   Trivy configuration such as severity thresholds, ignore lists or additional scanners. See
   the [Trivy documentation](https://trivy.dev/docs/latest/configuration/how-to-use-trivy-as-a-linter/) for
   details.

4. Run the hooks locally with `pre-commit run --all-files` or configure them to run on every
   commit/push in CI.

## Notes

* The `trivy-image-daemon` hook requires that Docker is available on the host. It mounts
  `/var/run/docker.sock` inside the Trivy container to inspect local images. This hook should
  typically be run manually or in CI environments where a Docker daemon is available.

* To reduce scanning time, the vulnerability database is cached in `.pre-commit-trivy-cache` in your
  project. If you need a fresh database (for example to pick up newly published CVEs), remove
  that directory and re-run the hook.

* For additional scanners (e.g. **secret** or **license**) or to adjust severity thresholds,
  pass the corresponding flags via `args:` in your `.pre-commit-config.yaml`.
