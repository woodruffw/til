---
title: Configuring GitHub releases with release.yml
date: 2024-08-13
tags: [github]
origin: personal correspondence
---

I use [GitHub's releases] to manage releases for (most of) my
projects.

Combined with [`gh release create`], they allow for very smooth locally-triggered,
CI-finished release processes!

One of the cool things that GitHub can do is automatically
[generate release notes]. These are often good enough for smaller projects,
supplanting the need to manually curate and maintain a `CHANGELOG` or similar.

One downside to these automatically generated notes is that they
can be very noisy: if your repository contains a lot of automation
(such as [Dependabot] for dependency updates), the release notes will
include summaries of each automatic PR, by default. Automated
release notes also don't contain semantic groupings by default, e.g.
telling users which changes are considered breaking vs. non-breaking.

However, both of these can be configured! When creating a release,
GitHub will check the `.github/release.yml` file and use its configuration
to customize the changelog's content and categorization.

For example, here's a `.github/release.yml` that filters out all noise
from Dependabot, and also performs some notes grouping (e.g. placing
bugfixes into their own category):

```yaml
changelog:
  exclude:
    authors:
      - dependabot
  categories:
    - title: Bugfixes
      labels:
        - bugfix
    - title: Features
      labels:
        - enhancement
```

[GitHub's releases]: https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases

[`gh release create`]: https://cli.github.com/manual/gh_release_create

[generate release notes]: https://docs.github.com/en/repositories/releasing-projects-on-github/automatically-generated-release-notes

[Dependabot]: https://github.com/dependabot
