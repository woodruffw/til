---
title: The POST/redirect/GET pattern
date: 2024-10-17
tags: [python, web]
origin: personal correspondence
---

I'm not much of a web developer, but I'm vaguely aware of a number of common
footguns, like the "resubmitted form" problem: during form submission
users will sometimes attempt to refresh the page, causing the underlying
`HTTP POST` to be sent twice. This results in "classic" failure modes like
users purchasing the same item twice, toggle states resetting themselves
(going `on -> off -> on`), and so forth.

I vaguely knew that there was a way to work around this, but I didn't know
what it was called until today: the [POST/redirect/GET pattern].

The pattern itself is simple:

1. Accept the user's `POST` as normal;
2. Respond to the `POST` with a HTTP redirect, typically an [HTTP 303];
3. The agent receives the 303 and performs a `GET` on the `Location:`
   in the 303's response headers.

The end result of this flow is that the user agent performs two requests
instead of one: the `POST` as expected, followed immediately by a `GET`
to a location controlled by the `POST`'s response. This results
in the last request to the server being a `GET`, meaning that a user
who then refreshes cannot accidentally re-submit the original `POST`ed
form.

This pattern isn't 100% foolproof, since the user could still perform
a refresh in the small interval between the `POST` is send and the 303
redirect is processed. However, this window is small and fundamentally
closed, unlike the inherently open-ended window of a non-redirect
response to a `POST`.

[POST/redirect/GET pattern]: https://en.wikipedia.org/wiki/Post/Redirect/Get

[HTTP 303]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/303
