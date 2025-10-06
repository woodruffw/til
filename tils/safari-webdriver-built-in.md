---
title: Safari has built-in WebDriver support
date: 2025-10-05
tags: [web, html, testing, scraping, python]
---

I recently needed to do a small amount of web scraping. This scraping
involved pages with an indeterminate amount of JavaScript, so I needed
*some* kind of browser automation (instead of something like [Beautiful Soup]).

Normally, I'd do this by setting up Selenium with [ChromeDriver]
or [geckodriver]. However, today I learned that macOS's Safari has built-in
WebDriver support!

```bash
$ which safaridriver
/System/Cryptexes/App/usr/bin/safaridriver
```

To use it, you need to run `safaridriver --enable` once. That'll prompt you
for your password and will ensure that the `safaridriver` binary is allowed
to control Safari.

From there, I was able to use Selenium (from Python) as normal, with
`webdriver.Safari` as the driver instance:

```python
options = webdriver.SafariOptions()
driver = webdriver.Safari(options=options)

driver.get("https://example.com")
```

This dovetails really nicely with `uv`'s support for [inline script metadata],
making it *incredibly* easy to write self-contained scrapers that
Just Work&trade; on macOS:

```python
# /// script
# requires-python = ">=3.13"
# dependencies = [
#     "selenium~=4.0",
# ]
# ///

from selenium import webdriver

def main():
    options = webdriver.SafariOptions()
    driver = webdriver.Safari(options=options)

    # for demo purposes, run with a maximized window
    driver.maximize_window()

    driver.get("https://example.com")

    # for demo purposes, block before exiting
    input()

    driver.quit()

if __name__ == "__main__":
    main()
```

And to run it:

```bash
uv run --script demo.py
```

Then, as a cherry on top, it turns out that the macOS runners on
GitHub Actions *also* have `safaridriver` installed *and* already
enabled by default!

In other words, this Just Works&trade; on GitHub Actions:

```yaml
name: Safari WebDriver Demo

on:
  workflow_dispatch:
  push:
    branches:
      - main

permissions: {}

jobs:
  check-appointments:
    runs-on: macos-latest
    permissions:
      contents: read # if your repo is private
    steps:
      - name: Checkout repository
        uses: actions/checkout@08c6903cd8c0fde910a37f88322edcfb5dd907a8 # v5.0.0
        with:
          persist-credentials: false

      - name: Set up uv
        uses: astral-sh/setup-uv@d0cc045d04ccac9d8b7881df0226f9e82c39688e # v6.8.0

      - run: |
          uv run --script demo.py
```

No more mucking with different versions of ChromeDriver or geckodriver,
at least for my minimal use case.

Apple's documentation has [more details] on `safaridriver` as well.

[Beautiful Soup]: https://www.crummy.com/software/BeautifulSoup/

[ChromeDriver]: https://developer.chrome.com/docs/chromedriver/get-started

[geckodriver]: https://github.com/mozilla/geckodriver

[inline script metadata]: https://docs.astral.sh/uv/guides/scripts/#declaring-script-dependencies

[More details]: https://developer.apple.com/documentation/webkit/about-webdriver-for-safari
