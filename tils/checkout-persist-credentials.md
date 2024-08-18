---
title: actions/checkout can leak GitHub credentials
tags: [github-actions, security]
date: 2024-08-17
origin: personal correspondence
---

[`actions/checkout`] is one of GitHub Actions' official, blessed actions:
it's used by millions of CI workflows to perform `git` repository checkouts.

`actions/checkout` also contains a **significant** security footgun:
by default, it'll persist a copy of the workflow's credential
(either the default `secrets.GITHUB_TOKEN` one or a custom-supplied one)
in the checked-out repository's `.git/config`.

This wouldn't be _too_ bad, except that it's somewhat common for CI workflows
to then archive and upload the repository as a workflow artifact for use in
subsequent steps. A team at Palo Alto Networks [found] dozens of projects
that inadvertently leak their credentials through this and similar artifact
vectors.

GitHub knows about this and considers it intentional behavior, which seems
not great&trade; to me. If you don't want credentials added to your
`.git/config`, they suggest running steps like `actions/checkout` like so:

```yaml
uses: actions/checkout@v4
with:
  persist-credentials: false
```

TL;DR: By default, `actions/checkout` will **silently introduce** secrets
into your repository, even if you maintain proper `git` hygiene and never
check any secrets in. The only way to avoid this is to pass
`persist-credentials: false` whenever using `actions/checkout`.

[`actions/checkout`]: https://github.com/actions/checkout

[found]: https://unit42.paloaltonetworks.com/github-repo-artifacts-leak-tokens/
