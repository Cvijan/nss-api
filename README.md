# Northern Star Shop API — v1

Read-only access to the Northern Star Shop product catalog and orders. All responses are JSON, encoded in UTF-8.

**Base URL**

```
https://cvijan.github.io/nss-api/v1/
```

**Authentication:** none. The API is public and read-only.

**Rate limits:** none enforced. Fetch each resource once per run and keep what you downloaded instead of calling again.

---

## Conventions

- **Dates** are ISO 8601 (`2026-11-09`).
- **Money** is in Serbian dinars (RSD).
- **Optional fields** are left out of the response when they are unknown, not sent as `null`.
- **Paths** in responses (`next`) are relative. Resolve them against the URL of the response they came from.
- **Data quality:** orders are entered through a web form and are sent to the API as received. Expect inconsistent spacing and letter case, missing or invalid values, test orders and orders that were sent more than once. Validating the data is the client's responsibility.

---

## Pagination

List endpoints return pages of at most 25 records, wrapped in this envelope:

```json
{
  "data": [ ... ],
  "page": 1,
  "total_pages": 5,
  "total_records": 125,
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
  "products_url": "products/page-1.json",
  "orders_url": "orders/page-1.json",
  "product_pages": 2,
  "order_pages": 5
}
```

### `GET products/page-1.json` (paginated)

The product catalog.

| Field | Type | Description |
| --- | --- | --- |
| `product_id` | integer | unique product ID |
| `name` | string | product name |
| `category` | string | one of `jerseys`, `scarves`, `balls`, `caps`, `accessories` |
| `price` | number | current price in RSD |

### `GET orders/page-1.json` (paginated)

All orders, one product per order.

```json
{
  "order_id": "NSS-0003",
  "order_date": "2026-11-09",
  "customer": {"name": "MILICA PETROVIĆ", "city": " NIŠ "},
  "product_id": 122,
  "quantity": 3,
  "unit_price": 1490.0,
  "payment": "paypal"
}
```

| Field | Type | Description |
| --- | --- | --- |
| `order_id` | string | order ID; real orders start with `NSS-`, test orders with `TST-` |
| `order_date` | string | date the order was placed, `YYYY-MM-DD` |
| `customer.name` | string | customer's full name, as typed |
| `customer.city` | string, optional | delivery city, as typed; omitted if unknown |
| `product_id` | integer | product ID, see `products` |
| `quantity` | integer | number of pieces; may arrive as text, be missing or be invalid |
| `unit_price` | number | price per piece in RSD; may be missing or invalid |
| `payment` | string | `card`, `cash` or `paypal` |

---

## Errors

| Status | Meaning |
| --- | --- |
| `200` | OK |
| `404` | the resource does not exist, e.g. a page past the last one |

Error responses are not JSON. Check the status code before parsing the body.

---
