---
title: GitHub Actions can call Docker images as workflow steps
date: 2024-08-19
tags: [github, github-actions]
---

Whenever I think I've reached a level of immense familiarity with GitHub
Actions, it surprises me.

Today, I've learned that you can use the `uses:` clause of a step (within
a workflow's job) to refer not just to a normal action
(e.g. `actions/checkout`) or a local action (e.g. `./somewhere/in/the/repo`)
but *also* to a full-blown Docker image!

[GitHub's docs] provides examples:

```yaml
jobs:
my_first_job:
  steps:
    - name: My first step
      uses: docker://alpine:3.8
```

This of course works with other registries besides Docker Hub too, e.g.
GitHub's own GHCR:

```yaml
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: docker://ghcr.io/OWNER/IMAGE_NAME
```

To make matters confusing, this feature is **completely different** from the one
you get when you search for Docker in GitHub Actions, which is an unrelated
thing called ["Docker container actions"]. *That* feature involves a standard
`action.yml` which then dispatches to a Docker image via some dedicated
metadata, while *this* feature seemingly invokes a Docker image, *any* image,
directly with no `action.yml` needed.

The docs are otherwise pretty thin on how this behaves, so I tried it out:

```yaml
on: push

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Docker
        uses: docker://alpine:3.8
```

...which did nothing, probably because `alpine:3.8` has no `ENTRYPOINT`
and a default `CMD` of `/bin/sh`. But it did confirm that GitHub Actions
invokes the image via `docker run`, with a great deal of useful state:

```bash
/usr/bin/docker run --name alpine38_63efcf --label a790e7 \
    --workdir /github/workspace --rm -e "HOME" -e "GITHUB_JOB" \
    -e "GITHUB_REF" -e "GITHUB_SHA" -e "GITHUB_REPOSITORY" \
    -e "GITHUB_REPOSITORY_OWNER" -e "GITHUB_REPOSITORY_OWNER_ID" \
    -e "GITHUB_RUN_ID" -e "GITHUB_RUN_NUMBER" -e "GITHUB_RETENTION_DAYS" \
    -e "GITHUB_RUN_ATTEMPT" -e "GITHUB_REPOSITORY_ID" -e "GITHUB_ACTOR_ID" \
    -e "GITHUB_ACTOR" -e "GITHUB_TRIGGERING_ACTOR" -e "GITHUB_WORKFLOW" \
    -e "GITHUB_HEAD_REF" -e "GITHUB_BASE_REF" -e "GITHUB_EVENT_NAME" \
    -e "GITHUB_SERVER_URL" -e "GITHUB_API_URL" -e "GITHUB_GRAPHQL_URL" \
    -e "GITHUB_REF_NAME" -e "GITHUB_REF_PROTECTED" -e "GITHUB_REF_TYPE" \
    -e "GITHUB_WORKFLOW_REF" -e "GITHUB_WORKFLOW_SHA" -e "GITHUB_WORKSPACE" \
    -e "GITHUB_ACTION" -e "GITHUB_EVENT_PATH" -e "GITHUB_ACTION_REPOSITORY" \
    -e "GITHUB_ACTION_REF" -e "GITHUB_PATH" -e "GITHUB_ENV" \
    -e "GITHUB_STEP_SUMMARY" -e "GITHUB_STATE" -e "GITHUB_OUTPUT" \
    -e "RUNNER_OS" -e "RUNNER_ARCH" -e "RUNNER_NAME" -e "RUNNER_ENVIRONMENT" \
    -e "RUNNER_TOOL_CACHE" -e "RUNNER_TEMP" -e "RUNNER_WORKSPACE" \
    -e "ACTIONS_RUNTIME_URL" -e "ACTIONS_RUNTIME_TOKEN" -e "ACTIONS_CACHE_URL" \
    -e "ACTIONS_RESULTS_URL" -e GITHUB_ACTIONS=true -e CI=true \
    -v "/var/run/docker.sock":"/var/run/docker.sock" \
    -v "/home/runner/work/_temp/_github_home":"/github/home" \
    -v "/home/runner/work/_temp/_github_workflow":"/github/workflow" \
    -v "/home/runner/work/_temp/_runner_file_commands":"/github/file_commands" \
    -v "/home/runner/work/actions-experiments/actions-experiments":"/github/workspace" \
    alpine:3.8
```

Of particular interest:

* The container receives the Docker socket (`/var/run/docker.sock`), making
  escapes from the container trivial. GitHub probably considers this intended
  behavior, since there isn't much purpose in hardening a container on an
  ephemeral VM that the user has complete control over.
* The special GitHub files (e.g. `_runner_file_commands`) get mounted in as well,
  but at different paths.
* The container doesn't receive the default runner credential
  (i.e. `secrets.GITHUB_TOKEN`) through the
  environment. However, it can almost certainly extract it from system memory
  through a container escape and attaching to GitHub's runner agent.

Overall, this feature seems to behave almost identically to GitHub Actions's
other technique for spawning Docker containers as action steps, but with
one less piece of indirection.

[GitHub's docs]: https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions#example-using-a-docker-hub-action

["Docker container actions"]: https://docs.github.com/en/actions/sharing-automations/creating-actions/creating-a-docker-container-action#testing-out-your-action-in-a-workflow
