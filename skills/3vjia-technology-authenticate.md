---
name: 3vjia-authenticate
description: Obtain and maintain a 3vjia Open Platform access_token, and read the response envelope correctly. Every other 3vjia skill depends on this one.
api: 3vjia Open Platform API
generated: '2026-09-05'
method: generated
source: https://dev.3vjia.com/v1/document?apiId=3a985690d1a94edd924372f8c10187ca (获取access_token接口) and https://dev.3vjia.com/v1/document?apiId=d39aed16895f4f4bb88df99d38f93fb3 (新手指南)
operations:
  - commonApiJoinAuthAuthorize
---

# Authenticate against the 3vjia Open Platform

## Before you start

You need an `appId` and `appKey`. They are not self-serve: an enterprise administrator applies at
<https://dev.3vjia.com/manage/my-app/developer>, waits for approval, then registers an application at
<https://dev.3vjia.com/manage/my-app/app-manage> and waits for that to be approved too. There is no
sandbox and no test credential.

## 1. Get a token

```http
POST https://graph.3vjia.com/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=<appId>&client_secret=<appKey>
```

Success (HTTP 200):

```json
{ "access_token": "<token>", "expires_in": 7200 }
```

Failure (non-200):

```json
{ "error": "invalid_client", "error_description": "invalid client_id and client_secret" }
```

## 2. Do NOT fetch a token per process

This is the rule most integrations get wrong, and the provider states it explicitly. **Only one token
is valid per application at a time — requesting a new one invalidates the previous one.** If two
services each refresh on their own schedule they will knock each other offline.

Run one central token service that refreshes on a timer and hands the current value to every caller,
and give it a passive refresh path too, so a caller that observes an expiry can trigger a refresh
rather than waiting for the timer. Treat `code: 1700200026` ("凭证已过期，请重新授权") from
`graph.3vjia.com` as the expiry signal.

## 3. Call the API

Current gateway:

```http
POST https://open-gateway.3vjia.com/api/v1/user/listByPage
Content-Type: application/json;charset=utf-8
```

Legacy gateway (still documented, older operation set):

```http
POST https://open.3vjia.com/<path>?sysCode=external&access_token=<token>
Content-Type: application/json;charset=utf-8
```

## 4. Read the envelope, not the status code

**The gateway returns HTTP 200 for failures.** Verified by live probe on 2026-09-05: a call with no
credential, a call to a path that does not exist, and a successful call all return `200`.

```json
{ "success": false, "code": 100100002, "msg": "缺少访问凭证信息" }
```

- Branch on `success`. Never on the HTTP status.
- `code: 100100002` — missing access credential. Get or refresh a token.
- `code: 100001012` — the path is not routed. Check the path; this is the gateway's soft-404.
- Keep the `magiccube-req-id` response header. It is undocumented but it is the only correlation
  identifier the API emits, and support will want it.

## 5. If you also need browser SSO

Separate mechanism, separate host. Redirect the user to
`https://sso.3vjia.com/JointLogin/Index?userid=&appid=&time=&sign=&redirect_uri=`, where
`sign = MD5(userId + appId + time + appKey)` and `time` is a 10-digit Unix timestamp accepted within
±5 minutes. `redirect_uri` must be URL-encoded and points at the 3vjia app to open, e.g.
`https://admin.3vjia.com/3DLoading?SchemeId=123456`.
