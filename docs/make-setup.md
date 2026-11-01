# Make workflow setup

This guide describes the working lesson project. Exported Make blueprints are not included. Module numbers vary: select your own mapping tokens.

## AI preview scenario

Custom webhook → AI extraction → Parse JSON → Iterator → item checks → Array aggregator → Router → preview or clarification response.

The webpage posts a form-encoded `message`. Extract `customer_name` and `items_json` (Text). Ask for every product and quantity as a bare JSON array. Normalise recognised products to Notebook, Pen, or School bag. Keep unsupported products as Unknown; use 0 for missing or vague quantities, never guess.

Wrap the mapped array text as `{"items": TOKEN}` in Parse JSON. Iterate `items[]`. Record `product_ok` and `quantity_ok` as 1 for valid and 0 for invalid. Quantities must be positive integers. Aggregate product, quantity and both scores without discarding invalid items.

The clear route requires a nonempty array and each score sum equal to array length. The lesson also checks the original sentence for vague words; this heuristic can produce false rejections. AI extraction and JSON parsing can fail, so production use needs appropriate error handling.

Return HTTP 200 with Content-Type application/json:

```json
{"status":"needs_confirmation","customer_name":"Emeka","items":[{"product":"Pen","quantity":2},{"product":"Notebook","quantity":2},{"product":"School bag","quantity":1}]}
```

The fallback returns a plain-text clarification. Neither route saves an order. Keep the incoming message mapped into AI input rather than using a fixed test sentence.

## Confirmed-order scenario

The webpage sends JSON containing `request_id`, `customer_name`, and `items`. See [sample request](../examples/order.json).

Webhook → Iterator → official price → item total → item checks → Array aggregator → grand total → validation Router → existence check → new/already-saved Router.

- Official prices: Pen 500, Notebook 2000, School bag 10000; unknown 0.
- Item total: quantity multiplied by official price.
- Product check: official price greater than zero.
- Quantity check: positive whole number.
- Aggregate product, quantity, official_price, total, product_ok and quantity_ok. Source: Iterator. Leave Group by empty and stop-on-empty off.
- Grand total: sum of item totals.
- Validate the entire order before saving. Require a nonempty list and every item check to pass. Do not filter individual items out before aggregation.

Reject invalid orders with HTTP 400. Test malformed, empty and non-numeric inputs explicitly; arithmetic errors must not cause partial saves.

Check existence using the webhook request_id as the data-store key.

**New order:** Create JSON containing calculated items → Save order with Key=request_id and overwrite disabled → HTTP 200 response with reference and grand total.

**Already saved:** Get the record by the same key → return HTTP 200 with the original saved total.

**Save error handler:** Webhook response HTTP 503 → Skip. The page retains the original ID for a retry. Existence-check then insertion can race; disabling overwrites on insertion remains necessary.

Run the scenario immediately as data arrives. The lesson encountered response warnings with Process data in order enabled; retain the working immediate-response settings and verify in your account. JSON requests from the webpage require appropriate CORS and OPTIONS handling.

## Data store

| Field | Value |
|---|---|
| Key | Webhook request_id |
| customer_name | Text from webhook |
| items_json | Text output from Create JSON |
| total | Number: grand total |
| status | Order received |

Keep old single-item fields optional if retaining earlier records. New multi-item orders use items_json.

## Test checklist

1. Preview two and three different products. Check that all appear.
2. Submit the sample: one record, three items, NGN 15,000.
3. Retry the same ID and payload: return the saved result without a second record.
4. Reject the entire order if any product is unknown or a quantity is zero, negative, fractional or missing.
5. Test vague messages, empty arrays and malformed input.
6. Test connection failure and retry without refreshing the page.
7. Test concurrent requests with one fresh ID; verify one record. Near-simultaneous launches do not guarantee a save race.

Do not reuse an ID for a changed order. The existing-record route looks up by ID and does not compare request payloads. Pending browser orders are held in memory and lost on refresh.
