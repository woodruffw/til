---
title: OSV.dev provides data dumps
date: 2025-01-19
tags: [security, data]
origin: https://google.github.io/osv.dev/data/#data-dumps
---

[OSV.dev](https://osv.dev/) is an OSS vulnerability database maintained by Google.

It provides a [web interface](https://osv.dev/) as well as an unmetered
[REST API](https://google.github.io/osv.dev/api/) that serves
responses in the [OSV schema format](https://ossf.github.io/osv-schema/).

However, sometimes you need to retrieve the *full* vulnerability
history for an *entire* ecosystem, regardless of individual package identifiers.
OSV.dev provides this as well!
From their [data dumps page](https://google.github.io/osv.dev/data/#data-dumps):

```
gs://osv-vulnerabilities/<ECOSYSTEM>/all.zip
```

...where `<ECOSYSTEM>` is the ecosystem name as it appears on OSV.dev, e.g.
`PyPI` for Python or `GitHub Actions` for GitHub Actions. Note the space;
that appears in the URL as well.

For those of us who prefer plain old HTTP:

```bash
curl -o github-actions.zip https://osv-vulnerabilities.storage.googleapis.com/GitHub%20Actions/all.zip
unzip -d github-actions github-actions.zip
```

yields:

```
Archive:  github-actions.zip
  inflating: github-actions/GHSA-2c6m-6gqh-6qg3.json
  inflating: github-actions/GHSA-4mgv-m5cm-f9h7.json
  inflating: github-actions/GHSA-4xqx-pqpj-9fqw.json
  inflating: github-actions/GHSA-5xr6-xhww-33m4.json
  inflating: github-actions/GHSA-634p-93h9-92vh.json
  inflating: github-actions/GHSA-6q4m-7476-932w.json
  inflating: github-actions/GHSA-7f32-hm4h-w77q.json
  inflating: github-actions/GHSA-7x29-qqmq-v6qc.json
  inflating: github-actions/GHSA-8v8w-v8xg-79rf.json
  inflating: github-actions/GHSA-99jg-r3f4-rpxj.json
  inflating: github-actions/GHSA-cxww-7g56-2vh6.json
  inflating: github-actions/GHSA-f9qj-7gh3-mhj4.json
  inflating: github-actions/GHSA-g85v-wf27-67xc.json
  inflating: github-actions/GHSA-g86g-chm8-7r2p.json
  inflating: github-actions/GHSA-ghm2-rq8q-wrhc.json
  inflating: github-actions/GHSA-h3qr-39j9-4r5v.json
  inflating: github-actions/GHSA-hw6r-g8gj-2987.json
  inflating: github-actions/GHSA-mcph-m25j-8j63.json
  inflating: github-actions/GHSA-p756-rfxh-x63h.json
  inflating: github-actions/GHSA-rg3q-prf8-qxmp.json
  inflating: github-actions/GHSA-xj87-mqvh-88w2.json
```

Each of the files in the dump is JSON, in OSV's schema format.
