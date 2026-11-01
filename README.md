# AI Order Automation

A learning project that turns a customer's message into a multi-product order preview, asks for confirmation, and saves the order using Make.

Example: “I'm Emeka. I want 2 pens, 2 notebooks and 1 school bag.”

## Features

- AI-assisted extraction of customer names, products, and quantities.
- Preview of every item before confirmation.
- Positive whole-number quantity validation.
- Official prices calculated in Make, not supplied by the browser.
- One `request_id` per order, reused when retrying an uncertain save.
- Make data-store lookup and unique keys to prevent duplicate orders.
- Saved item breakdown and grand total.
- Separate responses for invalid orders and temporary save failures.

## Project files

- `index.html`: the latest multi-product customer form, with webhook addresses replaced by placeholders.
- `docs/make-setup.md`: workflow reconstruction guide and data contracts.
- `examples/order.json`: a fictional three-product request.

The Make scenarios run in Make, not in this repository. Their exported blueprints are **not included**; the setup guide describes the configuration built during the lessons. This is not a one-click import.

## Run locally

1. Configure the two Make scenarios described in [the setup guide](docs/make-setup.md).
2. In `index.html`, replace `YOUR_AI_WEBHOOK_URL` and `YOUR_ORDER_WEBHOOK_URL` with your own endpoints.
3. Serve the folder locally, for example using VS Code Live Server. Use localhost or HTTPS so `crypto.randomUUID()` is available.
4. Open the webpage, send a message, check every previewed item, then confirm.
5. Inspect Make History and your orderbook to verify the result.

For the JSON order request, the endpoint must support browser cross-origin requests, including any OPTIONS preflight. Check this before treating a browser fetch failure as a failed save.

Do not commit real webhook addresses or customer records. Addresses configured in a browser remain visible to its users; placeholders only keep the original endpoints out of this repository.

## Example prices

| Product | Unit price (NGN) |
|---|---:|
| Pen | 500 |
| Notebook | 2,000 |
| School bag | 10,000 |

The sample order totals NGN 15,000.

## Validation and limits

The project author reported a successful end-to-end multi-product test in Make. The repository copy has been checked for JavaScript syntax; live webhook tests were not run as part of publication.

This is an educational prototype, not a production checkout. Pending retries are held in browser memory and are lost on refresh. AI extraction can miss or misinterpret items, so the customer must review the preview. Server-side validation is required even though the browser also checks quantities. There is no payment processing, inventory management, authentication, or durable browser retry storage. The current confirmation handler treats a nonempty successful response other than `Accepted` as confirmation; a production version should use a structured response contract.

The planned shawarma project is a separate adaptation and is not included here.
