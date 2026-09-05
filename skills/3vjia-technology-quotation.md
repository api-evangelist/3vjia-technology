---
name: 3vjia-run-quotation
description: Price a 3vjia design scheme or order by submitting an asynchronous quotation task and polling for the result, handling the 609006 keep-polling signal.
api: 3vjia Open Platform API
generated: '2026-09-05'
method: generated
source: openapi/3vjia-technology-open-platform-openapi.yml and the 获取报价计算结果接口 documentation at https://dev.3vjia.com/v1/document
operations:
  - apiV1QuotationQuoteScheme
  - apiV1QuotationQueryQuoteSchemeResult
  - apiV1QuotationQuoteOrder
  - apiV1QuotationQueryQuoteOrderResult
  - apiV1SchemeGetSchemeProduct
  - apiV1QuotationGetSchemeProductPrice
---

# Run a quotation on the 3vjia Open Platform

Authenticate first — see `3vjia-authenticate`. Quotation is asynchronous: you submit, you get a key,
you poll.

## Choose the right pair

There are two parallel flows and they are not interchangeable:

| Subject | Submit | Poll |
|---|---|---|
| A design scheme | `apiV1QuotationQuoteScheme` (提交方案报价计算任务接口) | `apiV1QuotationQueryQuoteSchemeResult` (获取方案报价计算结果接口) |
| An order | `apiV1QuotationQuoteOrder` (提交订单报价计算任务接口) | `apiV1QuotationQueryQuoteOrderResult` (获取订单报价计算结果接口) |

## 1. Submit

Submit with the scheme or order identifier. The response hands back a `key` — the query token.

## 2. Poll with the key

```json
{ "key": "<the token from step 1>" }
```

Read `errorCode` on the legacy envelope (`success`/`errorCode`/`errorMessage`/`result`):

- **`609006` — still calculating. This is not an error. Keep polling.** It is the only documented
  continue signal, and treating it as a failure is the most common way this flow is implemented wrong.
- `609004` — calculation failed; the reason is in the message.
- `609005` — the key is invalid.
- `609007` — the key has expired. Stop polling and resubmit. The validity duration is **not published**,
  so cap your loop rather than assuming one.
- `-3` — the parameters you sent do not meet requirements.

Back off between polls and set a hard attempt ceiling. There is no published rate limit, but there is
also no idempotency, so do not restart the submit step on a timeout without first polling the key you
already hold.

## 3. Read the result

`result` is a `QuoteResultBO`: `schemeId`, `schemeName`, `organ`, `dept`, `author`, and `quoteInfo`.

**`quoteInfo` is configurable per customer.** The provider states it is set up per enterprise via
project management at `bj.3vjia.com`, so its shape differs between tenants. Its default structure
groups line items as `Hardware`, `Garnish`, `Functor`, `doorBoard`, `Sliding` and `Part`, alongside
`success` and `amountMoney`. Parse defensively: do not assume a fixed key set.

## 4. If you want the bill of materials instead of a price

- `apiV1SchemeGetSchemeProduct` (获取方案产品清单接口) — the product list on a scheme.
- `apiV1QuotationGetSchemeProductPrice` (获取方案成品清单接口) — the priced finished-goods list.

## Optional: receive the result by push

3vjia can also push `operateType: QUOTE_ORDER_CALCULATE` to an endpoint you host, carrying the result
file path and the query token in `extParam`. The endpoint is registered out of band through a 3vjia
project manager, and you must answer `{"code": 10000}` or 3vjia retries. See
`asyncapi/3vjia-technology-webhooks.yml`.
