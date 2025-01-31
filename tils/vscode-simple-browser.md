---
title: VS Code has a built-in "simple" browser
date: 2025-01-31
tags: [vscode, web]
---

VS Code is, of course, really just a web browser pretending to be an IDE.

But this TIL isn't about that: it's about the built-in "simple" browser
that can be used to render HTML content without leaving the editor.

The simple browser can be launched from the command pallette with the
`Simple Browser: Show` command. That command prompts for a URL to
open, which then appears as a normal editor tab that can be moved around/laid
out like any other VS Code pane/tab.

As the name suggests, the simple browser can't do very much. However,
it's more than sufficient for doing live development against a local web server.

It even supports hot reloading, which means it works out of the box
with `mkdocs` and most other site generators that offer a local development mode.
