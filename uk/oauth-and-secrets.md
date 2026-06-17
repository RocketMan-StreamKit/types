# OAuth і секрети

Інтеграційні аддони StreamKit+ працюють у читабельній пісочниці. **Ніколи** не вбудовуйте `client_secret` або подібні серверні облікові дані в код аддона.

## Патерн

1. Виконуйте OAuth у браузері (користувач авторизує ваш застосунок).
2. Отримайте authorization code на локальному callback URL, наприклад  
   `http://localhost:{WEB_SERVER_PORT}/addon/{addonId}/auth`
3. Обміняйте code через **`network.request.post`** на **HTTPS proxy**, який ви керуєте (або спільний URL auth server StreamKit+, який користувач налаштовує в settings).
4. Зберігайте токени в `api.config.updateParams` (поля без `editor` для секретів).

У пісочниці **немає** `api.authServer` або вбудованого OAuth helper. Обмін токенів — звичайний вихідний HTTP-виклик.

## Приклад потоку (загальний)

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

## PKCE і підписи webhook

Допоміжні функції пісочниці:

```js
const { verifier, challenge } = crypto.createPkce();
const valid = crypto.verifyRsaSha256(publicKeyPem, rawBody, signatureBase64);
```

Для webhook реєструйте `POST`-ендпоінти; обробники отримують `headers` і `rawBody` для перевірки підпису.

## Сторінки результату OAuth

Перенаправляйте користувачів на стилізовані сторінки після входу:

```js
return { redirect: ui.auth.generateSuccess('Account linked') };
return { redirect: ui.auth.generateFail('Authorization denied') };
```

URL обслуговуються локальним web-сервером за `/ui/auth/success` і `/ui/auth/fail`.

## Self-hosted proxy

Якщо потрібен `client_secret`, запустіть невеликий HTTPS API (наприклад, FastAPI), який:

- Приймає JSON від аддона через `network.request.post`
- Додає секрети на сервері
- Викликає token endpoint провайдера
- Повертає JSON токенів аддону

Задокументуйте базовий URL (`api_server`) у схемі налаштувань аддона, щоб користувачі могли змінити хост/порт.
