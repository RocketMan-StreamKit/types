# Валюта

Чтение основной валюты из настроек приложения и конвертация сумм по тем же курсам, что использует основное приложение. Дополнительные разрешения не требуются.

Курсы загружаются и обновляются главным процессом (`Config.data.CURRENCY_RATES`). Конвертация выполняется через общий хелпер `ConvertCurrencyShared` (промежуточная валюта — USD).

## `currency.getCurrent()`

Возвращает код валюты, выбранной в **Настройки → Основные → Валюта**.

```js
const res = await currency.getCurrent();
if (res.success) {
  console.log('Валюта пользователя:', res.currency); // например "USD", "EUR", "RUB"
}
```

Возвращает `{ success: true, currency }`, где `currency` — `CurrencyCode` (например `USD`, `EUR`, `UAH`).

## `currency.convert(amount, from, to)`

Конвертирует числовую сумму между двумя валютами.

```js
const res = await currency.convert(100, 'USD', 'EUR');
if (res.success) {
  console.log('Сконвертировано:', res.amount);
}
```

| Аргумент | Тип | Описание |
| --- | --- | --- |
| `amount` | `number` | Сумма в исходной валюте |
| `from` | `CurrencyCode` | Код исходной валюты |
| `to` | `CurrencyCode` | Код целевой валюты |

Возвращает `{ success: true, amount }` со сконвертированным значением или `{ success: false, message? }`, когда:

- `amount` не является конечным числом
- один из кодов валюты неизвестен
- курс для одной из валют равен нулю или недоступен (например `CHEER`)

### Пример: донат в валюте пользователя

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
