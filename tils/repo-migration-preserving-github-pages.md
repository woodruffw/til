---
title: Preserving old GitHub Pages URLs while migrating a repo
date: 2025-05-09
tags: [github, web]
origin: personal correspondence
---

Today I migrated [`zizmor`] from [@woodruffw] (my personal GitHub account)
to [@zizmorcore] (a new GitHub organization that I created to host it).

One of the nice things about GitHub is that is does a _pretty_ good job
of handling these kinds of migrations: the old repository's URLs
(and `git` remotes) automatically (and permanently) redirect to the new
repository's URLs, so long as I don't create a _brand new_ repository with the
same name as the old one (i.e. `woodruffw/zizmor`).

However, not all is perfect: while _repository_ URLs are redirected,
[GitHub Pages] URLs are not. That meant that anything under
`https://woodruffw.github.io/zizmor/` would no longer work, and would
not redirect to `https://zizmorcore.github.io/zizmor/`.

GitHub's documentation implies that this is insurmountable, but it isn't!

My workaround involved two steps: one optional, and one required.

## Optional step: acquire a separate domain

This step is strictly optional: you **do not need** a separate domain to
gracefully migrate a GitHub Pages site.

However, having one solves the problem more generally, by turning
it into a one-time process (instead of something that you'll need to
do should you ever need to migrate again).

So, I acquired [zizmor.sh] and followed [GitHub's documentation]
to configure it (via a subdomain) for [`zizmor`]'s GitHub Pages site.

End result: <https://woodruffw.github.io/zizmor> now redirects to
<https://docs.zizmor.sh>. But <https://woodruffw.github.io/zizmor>
_itself_ still needs to be migrated/redirected.

## Required step: use `owner/owner.github.io` to "trampoline" redirects

This step is required.

First, it's important to understand that GitHub Pages comes in
[two "flavors"]:

1. **Project pages** within a repository (e.g. `org/repo`).
   These appear at `https://org.github.io/repo/`.
2. **Account pages** within a user or organization (e.g. `org`).
   These appear at `https://org.github.io/`, and are configured
   via a _special repository_ named `org/org.github.io`.

The former is what most people use and are familiar with, while the latter
is a bit more obscure.

However, the latter is also a *superset* of the former:
if you create an `org/org.github.io` repository, it can contain
_arbitrary subdirectories_ that are served at `https://org.github.io/`.

Because these subdirectories are arbitrary, they can overlap with
repository names. This is the heart of our trick: we can create a
subdirectory[^nested] named `zizmor` within `woodruffw/woodruffw.github.io` that
will serve content _as if_ it were a GitHub Pages site within `woodruffw/zizmor`,
but without needing to create a new repository (and break all of GitHub's
nice redirects).

This even works nicely in a live setting: GitHub allows project pages to shadow
account pages, so the `https://woodruffw.github.io/zizmor/` URL will
gracefully stay tied to `woodruffw/zizmor` until the very moment of the
migration itself, at which point it will fall back to
`woodruffw/woodruffw.github.io`, which in turn "trampolines" the request
back to `https://docs.zizmor.sh`.

## Putting it all together

To make this all work, the account-level GitHub Pages site needs to serve
HTML content that redirects to the final destination.

Here's what I settled on in my case, using the `https://woodruffw.github.io/zizmor/`
to `https://docs.zizmor.sh` redirect as an example:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="refresh" content="7; url=https://docs.zizmor.sh/">
    <title>Redirecting to docs.zizmor.sh</title>
    <link rel="canonical" href="https://docs.zizmor.sh/">
</head>
<body>
    <h1>Redirecting to docs.zizmor.sh 🌈</h1>

    <p>
        You're seeing this because you have an old link to zizmor's docs.
        Update your bookmarks to the redirected URL!
    </p>
    <p>
        If you are not redirected automatically, follow the
        <a id="dest" href="https://docs.zizmor.sh/">link</a>.
    </p>

    <script>
        let fragment = window.location.hash;
        let redirectUrl = `https://docs.zizmor.sh/${fragment}`;
        setTimeout(function() {
            window.location=redirectUrl
        }, 5000);
        document.getElementById("dest").setAttribute("href", redirectUrl);
    </script>
</html>
```

This is a little bit more verbose than the "basic" `http-equiv=refresh`
meta tag trick, but in return it allows the redirect to forward any
URL fragments that might be present. This is particularly useful in
`zizmor`'s case, where the documentation makes heavy use of fragments to
enable deep linking.

It also degrades gracefully in the absence of JavaScript: the `setTimeout`
will never fire, but the `http-equiv=refresh` meta tag will still
redirect the user after a slightly longer delay.

Finally, to make this work with _all_ of my old links, I needed a bit
of code generation to create each appropriate redirect:

```python
#!/usr/bin/env -S uv run --script

# docs.zizmor.sh.py
# generate redirect pages from woodruffw.github.io to docs.zizmor.sh.

import string
import sys
from pathlib import Path

_TEMPLATE = string.Template(
    """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="refresh" content="7; url=${real_dest}">
    <title>Redirecting to docs.zizmor.sh</title>
    <link rel="canonical" href="${real_dest}">
</head>
<body>
    <h1>Redirecting to docs.zizmor.sh 🌈</h1>

    <p>
        You're seeing this because you have an old link to zizmor's docs.
        Update your bookmarks to the redirected URL!
    </p>
    <p>
        If you are not redirected automatically, follow the
        <a id="dest" href="${real_dest}">link</a>.
    </p>

    <script>
        let fragment = window.location.hash;
        let redirectUrl = `${real_dest}$${fragment}`;
        setTimeout(function() {
            window.location=redirectUrl
        }, 5000);
        document.getElementById("dest").setAttribute("href", redirectUrl);
    </script>
</html>
""".strip()
)


_PAGES = [
    "",  # web root (can't use / because of path joining)
    "installation/",
    "quickstart/",
    "usage/",
    "release-notes/",
    "configuration/",
    "audits/",
    "development/",
    "trophy-case/",
]

_OUT = Path(__file__).parent.parent / "docs" / "zizmor"
assert _OUT.is_dir(), f"Output directory {_OUT} does not exist"

for page in _PAGES:
    dir_ = _OUT / page
    dir_.mkdir(exist_ok=True)

    index = dir_ / "index.html"
    with index.open("w") as io:
        real_dest = f"https://docs.zizmor.sh/{page}"
        io.write(_TEMPLATE.substitute(real_dest=real_dest))

    print(f"[+] wrote {index}", file=sys.stderr)
```

The end result of this is a `docs/zizmor` directory within my account-level
GitHub Pages site that gracefully redirects all of my old
`woodruffw.github.io/zizmor/` links to the new `docs.zizmor.sh` domain.
All without compromising the repository migration itself or GitHub's own
redirects!

You can see the end result of this by clicking on one of `zizmor`'s old
links, like <https://woodruffw.github.io/zizmor/installation/>. You
can also check out the [woodruffw/woodruffw.github.io] repository to see the
redirects in action.

Finally, I _think_ this could probably be simplified a bit by using a single
`404.html` instead and handling all of the redirects there. But I didn't
try that.

[`zizmor`]: https://github.com/woodruffw/zizmor
[@woodruffw]: https://github.com/woodruffw
[@zizmorcore]: https://github.com/zizmorcore
[GitHub Pages]: https://pages.github.com/
[zizmor.sh]: https://zizmor.sh
[GitHub's documentation]: https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site
[two "flavors"]: https://docs.github.com/en/pages/getting-started-with-github-pages/what-is-github-pages
[woodruffw/woodruffw.github.io]: https://github.com/woodruffw/woodruffw.github.io

[^nested]: This subdirectory can actually be arbitrarily nested. I chose to put
           my entire account-level GitHub Pages site in a subdirectory named
           `docs`, so the actual path is `docs/zizmor` while the URL is still
           `https://woodruffw.github.io/zizmor/`.
