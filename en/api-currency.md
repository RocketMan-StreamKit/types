# Currency

Read the user's primary currency from app settings and convert amounts using the same exchange rates as the main application. No extra permissions are required.

Rates are loaded and refreshed by the main process (`Config.data.CURRENCY_RATES`). Conversion uses the shared `ConvertCurrencyShared` helper (USD as the intermediate rate).

## `currency.getCurrent()`

Returns the currency code selected in **Settings → Main → Currency**.

```js
const res = await currency.getCurrent();
if (res.success) {
  console.log('User currency:', res.currency); // e.g. "USD", "EUR", "UAH"
}
```

Returns `{ success: true, currency }` where `currency` is a `CurrencyCode` (for example `USD`, `EUR`, `RUB`).

## `currency.convert(amount, from, to)`

Converts a numeric amount between two currency codes.

```js
const res = await currency.convert(100, 'USD', 'EUR');
if (res.success) {
  console.log('Converted:', res.amount);
}
```

| Argument | Type | Description |
| --- | --- | --- |
| `amount` | `number` | Value in the source currency |
| `from` | `CurrencyCode` | Source currency code |
| `to` | `CurrencyCode` | Target currency code |

Returns `{ success: true, amount }` with the converted value, or `{ success: false, message? }` when:

- `amount` is not a finite number
- either currency code is unknown
- the exchange rate for either currency is zero or unavailable (for example `CHEER`)

### Example: donation in user's currency

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
