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

## `currency.getList()`

Returns every known currency as an array of `{ key, name }` objects (English display names). Order matches the Main settings picker: primary currencies first (`USD`, `EUR`, `UAH`, `RUB`, `BYN`, `PLN`), then alphabetical.

```js
const res = await currency.getList();
if (res.success) {
  console.log(res.currencies[0]); // { key: 'USD', name: 'United States Dollar' }
  for (const { key, name } of res.currencies) {
    console.log(key, name);
  }
}
```

Returns `{ success: true, currencies }` where each entry has:

| Field | Type | Description |
| --- | --- | --- |
| `key` | `string` | Currency code (e.g. `USD`, `EUR`) |
| `name` | `string` | English display name |

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
