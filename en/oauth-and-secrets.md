# OAuth and secrets

StreamKit+ integration addons run in a readable sandbox. **Never** embed `client_secret` or similar server-only credentials in addon code.

## Pattern

1. Perform OAuth in the browser (user authorizes your app).
2. Receive the authorization code on a local callback URL, e.g.  
   `http://localhost:{WEB_SERVER_PORT}/addon/{addonId}/auth`
3. Exchange the code via **`network.request.post`** to a **HTTPS proxy** you operate (or a shared StreamKit+ auth server URL the user configures in settings).
4. Store tokens in `api.config.updateParams` (fields without `editor` for secrets).

The sandbox has **no** `api.authServer` or built-in OAuth helper. Token exchange is a normal outbound HTTP call.

## Example flow (generic)

```js
// 1. Register callback endpoint
await network.endpoints.create('auth', 'GET', 'onOAuthCallback');

events.On('onOAuthCallback', async ({ query }) => {
  const { code, state } = query;
  if (state !== expectedState) {
    return { redirect: ui.auth.generateFail('Invalid state') };
  }

  const params = await api.config.getParams();
  const apiServer = params.api_server || 'https://example.com:2083';

  const raw = await network.request.post(`${apiServer}/myprovider/oauth/token`, {
    code,
    code_verifier: storedVerifier,
    redirect_uri: REDIRECT_URI,
  });

  const tokens = JSON.parse(raw);
  await api.config.updateParams({
    access_token: tokens.access_token,
    refresh_token: tokens.refresh_token,
  });

  return { redirect: ui.auth.generateSuccess('Connected') };
});
```

## PKCE and webhook signatures

Sandbox helpers:

```js
const { verifier, challenge } = crypto.createPkce();
const valid = crypto.verifyRsaSha256(publicKeyPem, rawBody, signatureBase64);
```

For webhooks, register `POST` endpoints; handlers receive `headers` and `rawBody` for signature verification.

## OAuth result pages

Redirect users to styled pages after login:

```js
return { redirect: ui.auth.generateSuccess('Account linked') };
return { redirect: ui.auth.generateFail('Authorization denied') };
```

URLs are served by the local web server at `/ui/auth/success` and `/ui/auth/fail`.

## Self-hosted proxy

If you need `client_secret`, run a small HTTPS API (e.g. FastAPI) that:

- Accepts JSON from the addon via `network.request.post`
- Adds secrets server-side
- Calls the provider token endpoint
- Returns tokens JSON to the addon

Document the base URL (`api_server`) in your addon settings schema so users can override host/port.
