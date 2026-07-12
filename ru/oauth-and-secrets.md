# OAuth и секреты

Интеграционные аддоны StreamKit+ работают в читаемой песочнице. **Никогда** не встраивайте `client_secret` и другие серверные учётные данные в код аддона.

## Паттерн

1. Выполните OAuth в браузере (пользователь авторизует ваше приложение).
2. Получите код авторизации на локальный callback URL, например  
   `http://localhost:{WEB_SERVER_PORT}/addon/{addonId}/auth`
3. Обменяйте код через **`network.request.post`** на **HTTPS-прокси**, который вы разворачиваете (или общий URL auth-сервера StreamKit+, который пользователь указывает в настройках).
4. Сохраните токены в `api.config.updateParams` (поля без `editor` для секретов).

В песочнице **нет** `api.authServer` и встроенного OAuth-хелпера. Обмен токенами — обычный исходящий HTTP-запрос.

## Пример потока (общий)

```js
// 1. Регистрация callback-эндпоинта
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

## PKCE и подписи вебхуков

Хелперы песочницы:

```js
const { verifier, challenge } = crypto.createPkce();
const valid = crypto.verifyRsaSha256(publicKeyPem, rawBody, signatureBase64);
```

Для вебхуков регистрируйте `POST`-эндпоинты; обработчики получают `headers` и `rawBody` для проверки подписи.

## Страницы результата OAuth

Перенаправляйте пользователя на оформленные страницы после входа:

```js
return { redirect: ui.auth.generateSuccess('Account linked') };
return { redirect: ui.auth.generateFail('Authorization denied') };
```

URL обслуживаются локальным веб-сервером по `/ui/auth/success` и `/ui/auth/fail`.

## Собственный прокси

Если нужен `client_secret`, поднимите небольшой HTTPS API (например, FastAPI), который:

- Принимает JSON от аддона через `network.request.post`
- Добавляет секреты на сервере
- Вызывает token endpoint провайдера
- Возвращает JSON с токенами аддону

Укажите базовый URL (`api_server`) в схеме настроек аддона, чтобы пользователь мог менять хост и порт.
