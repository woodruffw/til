---
title: Problem matchers in GitHub Actions
date: 2024-12-11
tags: [github, github-actions]
origin: https://github.com/woodruffw/zizmor/issues/69#issuecomment-2536407680
---

I already knew about
[GitHub Actions annotations](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/workflow-commands-for-github-actions),
but today I've learned about "problem matchers."

GitHub doesn't seem to document these, outside of
[a page in `actions/toolkit`](https://github.com/actions/toolkit/blob/main/docs/problem-matchers.md),
but the basic idea is this:

* You have a tool that emits diagnostics/logs/whatever else, and you
  want to transform that tool's output into something that fits into
  a GitHub Actions annotation (like `::warning` or `::error`).
* You define a *problem matcher*, which is a JSON document that contains
  one or more regular expressions that match your tool's output.
* You *register* your problem matcher by running `::add-matcher::path-to-your-matcher.json`,
  and optionally *deregister* it with `::remove-matcher owner=your-matcher-name::`.
* While your matcher is registered, GitHub Actions will use it to scan all
  output, capture matches, collect from their capture groups, and mash
  everything into the annotations data model.

The indirections here can be a little hard to follow, so here's a concrete
example of a matcher from GitHub's own docs:

```json
{
    "problemMatcher": [
        {
            "owner": "eslint-compact",
            "pattern": [
                {
                    "regexp": "^(.+):\\sline\\s(\\d+),\\scol\\s(\\d+),\\s(Error|Warning|Info)\\s-\\s(.+)\\s\\((.+)\\)$",
                    "file": 1,
                    "line": 2,
                    "column": 3,
                    "severity": 4,
                    "message": 5,
                    "code": 6
                }
            ]
        }
    ]
}
```

This matches output lines that look like ESLint's "compact" format, puts
their relevant fields into capture groups, and then maps those
capture groups (by index) to their corresponding parts of the annotation
data model.

On top of this, problem matchers can (apparently) be somewhat stateful,
to enable multi-line matching. Using another example from GitHub's own
docs:

```json
{
    "problemMatcher": [
        {
            "owner": "eslint-stylish",
            "pattern": [
                {
                    // Matches the 1st line in the output
                    "regexp": "^([^\\s].*)$",
                    "file": 1
                },
                {
                    // Matches the 2nd and 3rd line in the output
                    "regexp": "^\\s+(\\d+):(\\d+)\\s+(error|warning|info)\\s+(.*)\\s\\s+(.*)$",
                    // File is carried through from above, so we define the rest of the groups
                    "line": 1,
                    "column": 2,
                    "severity": 3,
                    "message": 4,
                    "code": 5,
                    "loop": true
                }
            ]
        }
    ]
}
```

This matches output lines that look like ESLint's "stylish" format,
via two separate patterns: the first pattern matches the `file`
for the annotation, while the second pattern matches the majority of
the annotation's body (line, column, &amp;c.). What makes it *special*
is the `"loop": true` field, which tells GitHub Actions to
*keep producing annotations* until the rule stops matching lines.

All in all, I think this is a pretty neat feature, and GitHub's own
official actions seem to make use of it. I don't fully understand why
it isn't documented more publicly.
