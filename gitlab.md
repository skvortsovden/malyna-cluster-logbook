# gitlab

## add k3s cluster to gitlab as runner

install Helm on the master node

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

get your GitLab Runner Token

- go to your project (or group) on GitLab.com.
- navigate to Settings > CI/CD > Runners.
- click New project runner, add any tags you want, and click Create.
- gitLab will provide a Runner Authentication Token (it usually starts with glrt-). Copy this.


create a `values.yaml` file with the following content, replacing `<YOUR_GITLAB_RUNNER_TOKEN>` with the token you copied from GitLab:

```yaml
gitlabUrl: "https://gitlab.com/"
runnerRegistrationToken: "" # Leave empty, we use runnerToken now
runnerToken: "<YOUR_TOKEN>"
concurrent: 4 # How many jobs can run at once

rbac:
  create: true

runners:
  config: |
    [[runners]]
      [runners.kubernetes]
        namespace = "{{.Release.Namespace}}"
        image = "ubuntu:22.04"
```

deploy the runner using Helm:

```bash
# Add the GitLab chart repository
helm repo add gitlab https://charts.gitlab.io
helm repo update

# Create a namespace for the runner
kubectl create namespace gitlab-runner

# Install the runner using your values.yaml
helm install --namespace gitlab-runner gitlab-runner -f values.yaml gitlab/gitlab-runner
```

## explicitly specify the helper image for the runner

by default, the GitLab Runner spin ups helper for AMD64 (x86_64)

raspberry pi 4B has ARM64 architecture, so the helper image for AMD64 will not work and the runner will fail with the following error:

```
WARNING: Failed to pull image "registry.gitlab.com/gitlab-org/gitlab-runner/gitlab-runner-helper:x86_64-v18.10.0" for container "init-permissions" with policy "": image pull failed: rpc error: code = NotFound desc = failed to pull and unpack image "registry.gitlab.com/gitlab-org/gitlab-runner/gitlab-runner-helper:x86_64-v18.10.0": no match for platform in manifest: not found
ERROR: Job failed: prepare environment: waiting for pod running: pulling image "registry.gitlab.com/gitlab-org/gitlab-runner/gitlab-runner-helper:x86_64-v18.10.0" for container init-permissions: image pull failed: rpc error: code = NotFound desc = failed to pull and unpack image "registry.gitlab.com/gitlab-org/gitlab-runner/gitlab-runner-helper:x86_64-v18.10.0": no match for platform in manifest: not found. Check https://docs.gitlab.com/runner/shells/#shell-profile-loading for more information
```

to fix this issue I need to explicitly specify the helper image for ARM64 architecture in the helm `values.yaml` file:

```
runners:
  config: |
    [[runners]]
      [runners.kubernetes]
...
        # Force the runner to use the ARM64 helper image
        helper_image = "registry.gitlab.com/gitlab-org/gitlab-runner/gitlab-runner-helper:arm64-v18.10.0"
```

and re-deploy the runner using Helm:

```bash
helm upgrade --namespace gitlab-runner gitlab-runner -f values.yaml gitlab/gitlab-runner
```