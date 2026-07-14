# Валюта

Читання основної валюти з налаштувань застосунку та конвертація сум за тими ж курсами, що використовує основний застосунок. Додаткові дозволи не потрібні.

Курси завантажуються та оновлюються головним процесом (`Config.data.CURRENCY_RATES`). Конвертація виконується через спільний хелпер `ConvertCurrencyShared` (проміжна валюта — USD).

## `currency.getCurrent()`

Повертає код валюти, обраної в **Налаштування → Основні → Валюта**.

```js
const res = await currency.getCurrent();
if (res.success) {
  console.log('Валюта користувача:', res.currency); // наприклад "USD", "EUR", "UAH"
}
```

Повертає `{ success: true, currency }`, де `currency` — `CurrencyCode` (наприклад `USD`, `EUR`, `RUB`).

## `currency.convert(amount, from, to)`

Конвертує числову суму між двома валютами.

```js
const res = await currency.convert(100, 'USD', 'EUR');
if (res.success) {
  console.log('Сконвертовано:', res.amount);
}
```

| Аргумент | Тип | Опис |
| --- | --- | --- |
| `amount` | `number` | Сума у вихідній валюті |
| `from` | `CurrencyCode` | Код вихідної валюти |
| `to` | `CurrencyCode` | Код цільової валюти |

Повертає `{ success: true, amount }` зі сконвертованим значенням або `{ success: false, message? }`, коли:

- `amount` не є скінченним числом
- один із кодів валюти невідомий
- курс для однієї з валют дорівнює нулю або недоступний (наприклад `CHEER`)

### Приклад: донат у валюті користувача

```js
events.On('onDonation', async ({ body }) => {
  const { amount, currency: from } = body;
  const current = await currency.getCurrent();
  if (!current.success) {
    return;
  }

  if (from === current.currency) {
    console.log(`${amount} ${from}`);
    return;
  }

  const converted = await currency.convert(amount, from, current.currency);
  if (converted.success) {
    console.log(`${amount} ${from} (${converted.amount} ${current.currency})`);
  }
});
```
