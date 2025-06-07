---
title: Adding chips to MkDocs
tags: [web, mkdocs, docs]
date: 2025-06-06
origin: https://github.com/squidfunk/mkdocs-material/discussions/4725
---

TL;DR: The end result of this can be seen in [zizmor's docs].

[MkDocs] is one of my favorite ways to build documentation sites.
When combined with [Material for MkDocs], it provides both great UX
and DX right out of the box.

*Another* thing thing that I like when writing documentation is
"chips" (also known as "pills" or "tags") &mdash; little rounded
elements that sit next to text and give users additional context,
like "this is recommended" or "this is intended for experts".

Neither MkDocs nor Material for MkDocs supports chips out of the box,
but adding them is pretty easy. To start, I followed
a hint in [discussion #4725]:

```css
.chip-recommended {
  display: inline-block;
  background: green;
  color: white;
  padding: 0px 6px;
  border-radius: 10px;
}
```

This got added to a `chips.css` file, which then got included
in the MkDocs configuration:

```yaml
extra_css:
  - assets/chips.css
```

From there, I could use the `chip-recommended` class in my Markdown
via the [`attr_list`] extension that comes with Material for MkDocs:

```md
foo *recommended*{ .chip-recommended }
```

This produced some reasonably rendered chips,
with `*recommended*` as their content.

However, this came with a significant downside: a chip placed in a
title element would appear verbatim in tables of contents, meaning
that users would see something like:

```md
### foo recommended
```

instead of just:

```md
### foo
```

To fix this, I employed two tricks:

1. Instead of using the element's content for the chip's text,
   I inserted the text through CSS's `content` property.
2. `attr_list` still needs _some_ element to apply attributes to,
   so I used a [zero-width space] inside italics to produce an effectively
   empty `<em>` that could be given my class.

The final chip CSS looks like this:

```css
.chip {
    display: inline-block;
    color: white;
    padding: 0px 6px;
    border-radius: 10px;
    font-size: small;
    vertical-align: middle;
    font-style: normal;
}

.chip-recommended {
    background: green;
}

.chip-recommended::before {
    content: "recommended";
}
```

...and the final Markdown looks like this:

```md
### foo *&#8203;*{ .chip .chip-recommended }
```

It's not very pretty to read as source, but it works well enough!

Of course, I have no idea whether this is a good idea or not.

[MkDocs]: https://www.mkdocs.org/
[Material for MkDocs]: https://squidfunk.github.io/mkdocs-material/
[discussion #4725]: https://github.com/squidfunk/mkdocs-material/discussions/4725
[`attr_list`]: https://squidfunk.github.io/mkdocs-material/setup/extensions/python-markdown/#attribute-lists
[zizmor's docs]: https://docs.zizmor.sh/usage/#integration
[zero-width space]: https://en.wikipedia.org/wiki/Zero-width_space
