+++
title = 'Rest'
date = 2024-10-02T20:06:12+03:00
+++

## http-status codes
- 3хх - additional action required
    - 304 Not Modified - client should use cache
- 4xx - client did something wrong, its better to include information about error
    - 410 Gone - resource was deleted
    - 401 Unauthorized - must be used with WWW-Authenticate header and therefore can be used only with HTTP-authentification
    - 403 Forbidden - should be used in all other cases
- 5xx - problem on the server side

## Cache-Control
Tricky example:
```http
Cache-Control: private, no-cache
```

- no-cache - cache always, but check with `If-Match` or `If-Modified-Since`
- private - can be cached in browser, but not in CDN or Proxy

How to forbid cache and remove old cache:

```http
Cache-Control: no-store, max-age=0
```
