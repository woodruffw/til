---
title: pkgdiff makes archive diffing easy
date: 2024-08-20
tags: [shell, security]
---

I need to diff two archives (i.e. ZIPs, tarballs, etc.) occasionally,
often because the two are nearly identical (two point releases of the
same package) or because they *purport* to be identical but don't
have the same digest (mysterious changes to hosted releases, [GitHub archives],
etc.).

When this comes up, I've historically whipped together a quick Python script
to extract the two archives and compare both their layouts and contents.
I've written this script half a dozen times, because I tend to save it in
`/tmp` and forget about it until I need it again.

But today I learned about [`pkgdiff`]:

```bash
brew install pkgdiff # or any other package manager
pkgdiff old.tar.xz new.tar.xz
```

yields:

```
reading packages ...
comparing packages ...
creating report ...
result: UNCHANGED
report: pkgdiff_reports/password-store/X_to_Y/changes_report.html
```

The report mentioned in the output is a nicely HTML-formatted summary of any
differences, broken down by filetype and individual file (with visual diffs
available). Very convenient!

I used it today to [diagnose a digest change on `pass` in `homebrew-core`].

[GitHub archives]: https://github.blog/open-source/git/update-on-the-future-stability-of-source-code-archives-and-hashes/

[`pkgdiff`]: https://lvc.github.io/pkgdiff/

[diagnose a digest change on `pass` in `homebrew-core`]: https://github.com/Homebrew/homebrew-core/pull/181795
