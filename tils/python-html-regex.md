---
title: Python's html.parser uses regexes to parse HTML
tags: [python, html, regex]
date: 2025-08-21
origin: conversation on the Astral Discord server
---

Source: I learned this from [burntsushi] on Astral's Discord server.

Parsing HTML documents with regular expressions is
[widely understood to be a bad idea]: HTML's grammar is [context-free],
meaning that it can express constructs that can't be expressed
in a [regular grammar].

However, this doesn't stop Python's `html.parser`:

```python
# Regular expressions used for parsing

interesting_normal = re.compile('[&<]')
incomplete = re.compile('&[a-zA-Z#]')

entityref = re.compile('&([a-zA-Z][-.a-zA-Z0-9]*)[^a-zA-Z0-9]')
charref = re.compile('&#(?:[0-9]+|[xX][0-9a-fA-F]+)[^0-9a-fA-F]')
attr_charref = re.compile(r'&(#[0-9]+|#[xX][0-9a-fA-F]+|[a-zA-Z][a-zA-Z0-9]*)[;=]?')

starttagopen = re.compile('<[a-zA-Z]')
endtagopen = re.compile('</[a-zA-Z]')
piclose = re.compile('>')
commentclose = re.compile(r'--!?>')
commentabruptclose = re.compile(r'-?>')
# Note:
#  1) if you change tagfind/attrfind remember to update locatetagend too;
#  2) if you change tagfind/attrfind and/or locatetagend the parser will
#     explode, so don't do it.
# see the HTML5 specs section "13.2.5.6 Tag open state",
# "13.2.5.8 Tag name state" and "13.2.5.33 Attribute name state".
# https://html.spec.whatwg.org/multipage/parsing.html#tag-open-state
# https://html.spec.whatwg.org/multipage/parsing.html#tag-name-state
# https://html.spec.whatwg.org/multipage/parsing.html#attribute-name-state
tagfind_tolerant = re.compile(r'([a-zA-Z][^\t\n\r\f />]*)(?:[\t\n\r\f ]|/(?!>))*')
attrfind_tolerant = re.compile(r"""
  (
    (?<=['"\t\n\r\f /])[^\t\n\r\f />][^\t\n\r\f /=>]*  # attribute name
   )
  ([\t\n\r\f ]*=[\t\n\r\f ]*        # value indicator
    ('[^']*'                        # LITA-enclosed value
    |"[^"]*"                        # LIT-enclosed value
    |(?!['"])[^>\t\n\r\f ]*         # bare value
    )
   )?
  (?:[\t\n\r\f ]|/(?!>))*           # possibly followed by a space
""", re.VERBOSE)
locatetagend = re.compile(r"""
  [a-zA-Z][^\t\n\r\f />]*           # tag name
  [\t\n\r\f /]*                     # optional whitespace before attribute name
  (?:(?<=['"\t\n\r\f /])[^\t\n\r\f />][^\t\n\r\f /=>]*  # attribute name
    (?:[\t\n\r\f ]*=[\t\n\r\f ]*    # value indicator
      (?:'[^']*'                    # LITA-enclosed value
        |"[^"]*"                    # LIT-enclosed value
        |(?!['"])[^>\t\n\r\f ]*     # bare value
       )
     )?
    [\t\n\r\f /]*                   # possibly followed by a space
   )*
   >?
""", re.VERBOSE)
# The following variables are not used, but are temporarily left for
# backward compatibility.
locatestarttagend_tolerant = re.compile(r"""
  <[a-zA-Z][^\t\n\r\f />\x00]*       # tag name
  (?:[\s/]*                          # optional whitespace before attribute name
    (?:(?<=['"\s/])[^\s/>][^\s/=>]*  # attribute name
      (?:\s*=+\s*                    # value indicator
        (?:'[^']*'                   # LITA-enclosed value
          |"[^"]*"                   # LIT-enclosed value
          |(?!['"])[^>\s]*           # bare value
         )
        \s*                          # possibly followed by a space
       )?(?:\s|/(?!>))*
     )*
   )?
  \s*                                # trailing whitespace
""", re.VERBOSE)
endendtag = re.compile('>')
endtagfind = re.compile(r'</\s*([a-zA-Z][-.a-zA-Z0-9:_]*)\s*>')
```

([Permalink])

The trick here is twofold:

* `html.parser` doesn't do *all* of its parsing with regular expressions:
  the core of the parser is a SAX-style streaming parser that's sufficiently
  expressive.
* The regular expressions used by `html.parser` contain lookarounds for
  quoting and whitespace, which are [not *themselves* non-regular] but
  are typically implemented with backtracking (which expresses
  non-regular languages).

So, this ends up being okay: Python doesn't end up using regular expressions
for the non-regular parts of HTML parsing, although it *does* use constructions
(lookarounds) that happen to be implemented with backtracking in CPython's
implementation of regular expressions.

[widely understood to be a bad idea]: https://stackoverflow.com/a/1732454

[context-free]: https://en.wikipedia.org/wiki/Context-free_grammar

[regular grammar]: https://en.wikipedia.org/wiki/Regular_grammar

[Permalink]: https://github.com/python/cpython/blob/a8d9d947843200a09c154f3bc55f4e87e35edab3/Lib/html/parser.py#L20C1-L88C64

[not *themselves* non-regular]: https://www.jstage.jst.go.jp/article/ipsjjip/27/0/27_422/_pdf/-char/en

[burntsushi]: https://github.com/burntsushi
