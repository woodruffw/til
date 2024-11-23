---
title: Backlinking to Fediverse accounts with meta tags
date: 2024-11-23
tags: [web, fediverse, mastodon]
origin: personal correspondence
---

[Mike Fiedler] showed me this recently: Mastodon has support
for a special `<meta>` tag that allows journalists (and anybody else)
to backlink to the Fediverse (i.e., Mastodon) account that created
the content at a given URL.

This `<meta>` tag behaves similarly to other [OpenGraph] tags,
and looks like this:

```html
<meta name="fediverse:creator" content="@yossarian@infosec.exchange">
```

When added to a HTML page's `<head>` block, this will result in a nice
card rendering that links back to the creator's account.

You can see an example of that [here]. I've added it to by [personal blog]
and this TIL tracker more generally.

The [Mastodon Blog] has more details as well.

[Mike Fiedler]: https://github.com/miketheman/

[OpenGraph]: https://ogp.me/

[Mastodon Blog]: https://blog.joinmastodon.org/2024/07/highlighting-journalism-on-mastodon/

[here]: https://infosec.exchange/@yossarian/113504719158109951

[personal blog]: https://blog.yossarian.net
