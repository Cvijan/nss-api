# Northern Star Shop API — v2

Read-only access to Northern Star Shop orders, plus data files from the shop and from the Northern Star League (NSL). Responses are JSON, encoded in UTF-8.

**Base URL**

```
https://cvijan.github.io/nss-api/v2/
```

**Files URL**

```
https://cvijan.github.io/nss-api/v2/files/
```

**Authentication:** none. The API is public and read-only.

**Rate limits:** none enforced. Fetch each resource once per run and keep what you downloaded instead of calling again.

v1 is still available at `.../nss-api/v1/` but is not used in this project. See the changelog for the differences.

---

## Conventions

- **Dates** are ISO 8601 (`2026-08-15`).
- **Money** is in Serbian dinars (RSD).
- **Optional fields** are left out of the response when they are unknown, not sent as `null`.
- **Paths** in responses (`next`) are relative. Resolve them against the URL of the response they came from.
- **Data quality:** orders are entered through a web form and are sent as received. Expect inconsistent spacing and letter case, missing or invalid values, test orders and orders that were sent more than once. Validating the data is the client's responsibility.

---

## Pagination

The orders endpoint returns pages of at most 50 records, wrapped in this envelope:

```json
{
  "data": [ ... ],
  "page": 1,
  "total_pages": 38,
  "total_records": 1852,
  "next": "page-2.json"
}
```

| Field | Type | Description |
| --- | --- | --- |
| `data` | array | records on this page |
| `page` | integer | current page number, starting at 1 |
| `total_pages` | integer | number of pages |
| `total_records` | integer | number of records across all pages |
| `next` | string, nullable | relative path to the next page; `null` on the last page |

To get every record, follow `next` until it is `null`.

---

## Endpoints

### `GET status.json`

Service information. Use it to check your connection.

```json
{
  "service": "Northern Star Shop API",
  "version": "v1",
  "orders_url": "orders/page-1.json",
  "order_pages": 38,
  "files": {
    "products": "files/products.csv",
    "teams": "files/teams.csv",
    "matches": "files/matches.parquet"
  }
}
```

### `GET orders/page-1.json` (paginated)

All orders from 2026-08-15 to 2026-11-15, one product per order.

```json
{
  "order_id": "NSS-00003",
  "order_date": "2026-08-15",
  "customer": {"name": "Sara Živković", "city": "Beograd"},
  "product_id": 1041,
  "quantity": 1,
  "unit_price": 6990.0,
  "payment": "card"
}
```

| Field | Type | Description |
| --- | --- | --- |
| `order_id` | string | order ID; real orders start with `NSS-`, test orders with `TST-` |
| `order_date` | string | date the order was placed, `YYYY-MM-DD` |
| `customer.name` | string | customer's full name, as typed |
| `customer.city` | string, optional | delivery city, as typed; omitted if unknown |
| `product_id` | integer | product ID, see `products.csv` |
| `quantity` | integer | number of pieces; may arrive as text, be missing or be invalid |
| `unit_price` | number | price per piece in RSD; may be missing or invalid |
| `payment` | string | `card`, `cash` or `paypal` |

---

## Files

Download these files as they are and read them locally. They are not paginated and have no envelope.

### `files/products.csv`

The product catalog, exported from the shop's ERP system. Comma-separated, with a header row.

| Column | Description |
| --- | --- |
| `product_id` | product ID, as used in orders |
| `sku` | stock-keeping unit, e.g. `NPM-JER-1001` |
| `product_name` | product name |
| `team_code` | three-letter code of the team the product belongs to (see `teams.csv`); empty for league products that belong to no team |
| `category` | `jerseys`, `scarves`, `caps`, `accessories` or `balls` |
| `price` | current price in RSD |
| `active` | whether the product is still sold (`yes` / `no`) |

The export is not cleaned. Formatting of prices, codes, categories and `active` is not consistent, and some rows may appear more than once.

### `files/teams.csv`

NSL teams, provided by the league office. Comma-separated, with a header row.

| Column | Description |
| --- | --- |
| `team_id` | team ID, as used in `matches.parquet` |
| `code` | three-letter team code, as used in `products.csv` |
| `team_name` | full team name |
|