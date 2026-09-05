---
name: 3vjia-order-to-production
description: Move a 3vjia order from store submission through factory sign-off to a production batch, and reverse it safely when something goes wrong.
api: 3vjia Open Platform API
generated: '2026-09-05'
method: generated
source: openapi/3vjia-technology-open-platform-openapi.yml and the 订单管理接口 / 生产批次管理接口 documentation at https://dev.3vjia.com/v1/document
operations:
  - apiV1ManufacturingGetShopOrderListByPage
  - apiV1ManufacturingGetPlatformOrderListByPage
  - apiV1ManufacturingGetOrder
  - apiV1ManufacturingGetOrderStateDictionary
  - apiV1ManufacturingSubmitOrder
  - apiV1ManufacturingSubmitAndSignOrder
  - apiV1ManufacturingSignOrder
  - apiV1ManufacturingUpdateOrderState
  - apiV1ManufacturingConfirmDisOrder
  - apiV1ManufacturingCreateOrderBatch
  - apiV1ManufacturingQueryCreateOrderBatchResult
  - apiV1ManufacturingGetBatchOrderList
  - apiV1ManufacturingGetBatchProductionXml
  - apiV1AimesFactoryOrderOpGetReturnSetting
  - apiV1AimesFactoryOrderOpReturnFactoryOrder
  - apiFactoryOrderOpBatchReturnCompletedFactoryOrderToShop
  - apiFactoryOrderOpInvalidateFactoryOrdersBySaleOrder
---

# Order to production on the 3vjia Open Platform

Authenticate first — see `3vjia-authenticate`.

## The identity you must hold on to

One purchase has several identifiers. `orderRecordId` is the common id shared by the store order and
the factory order; `orderNo` is the order number; `diyOrderNo` / `customOrderNo` is yours to set with
`apiV1ManufacturingUpdateDiyOrderNo`. Key your own records on `orderRecordId`.

## 1. Find the order

- `apiV1ManufacturingGetShopOrderListByPage` (获取门店订单列表接口) — store side, paged.
- `apiV1ManufacturingGetPlatformOrderListByPage` (获取工厂订单列表接口) — factory side, paged.
- `apiV1ManufacturingGetOrder` (获取订单信息接口) — one order.
- `apiV1ManufacturingGetOrderStateDictionary` (获取订单状态字典接口) — **call this first and cache it.**
  Order states are enterprise-configurable; do not hardcode them.

## 2. Submit and sign

- `apiV1ManufacturingSubmitOrder` (门店提交订单接口) — the store submits.
- `apiV1ManufacturingSubmitAndSignOrder` (门店提交订单工厂自动签收接口) — submit with automatic factory sign-off.
- `apiV1ManufacturingSignOrder` (工厂签收订单接口) — the factory signs for it.
- `apiV1ManufacturingConfirmDisOrder` (订单拆单确认接口) — confirm order splitting.

**There is no idempotency on any of these.** If a submit times out, do not resubmit. Re-read the order
with `apiV1ManufacturingGetOrder` and check its state before doing anything else — a duplicate factory
order is a real cost.

## 3. Generate a production batch

Asynchronous, same submit-then-poll shape as quotation:

1. `apiV1ManufacturingCreateOrderBatch` (生成批次接口)
2. `apiV1ManufacturingQueryCreateOrderBatchResult` (获取批次生成结果接口) — poll
3. `apiV1ManufacturingGetBatchOrderList` (获取已生成批次订单列表接口)
4. `apiV1ManufacturingGetBatchProductionXml` (获取批次拆单XML接口) — the machine data, returned as a URL

Batch numbers look like `3838-210330-01` (`<shop>-<yymmdd>-<seq>`).

## 4. Reversing — read this before you act

Reversal paths exist, but **no document states a time window for any of them**, so treat every window
as unknown and verify state before and after.

- **Returning a factory order.** Call `apiV1AimesFactoryOrderOpGetReturnSetting`
  (获取工厂单可退回节点信息接口) **first**. It tells you which production nodes still permit a return.
  This is the provider's substitute for a time window, and it is the only precondition query on the
  whole surface. Then call `apiV1AimesFactoryOrderOpReturnFactoryOrder` (退回工厂单接口).
- **Completed factory orders** can be batch-returned to the store with
  `apiFactoryOrderOpBatchReturnCompletedFactoryOrderToShop`.
- **Voiding everything under a sales order**: `apiFactoryOrderOpInvalidateFactoryOrdersBySaleOrder`.
- Cancellation is expressed as a state transition (`C2_SHOP_CANCEL`, `C2_PLATFORM_CANCEL`) via
  `apiV1ManufacturingUpdateOrderState`, not as a dedicated cancel operation.

Once material is cut, edge-banded or drilled the reversal is commercial, not technical. The status
webhook enumerates those points explicitly (`C2_PLATFORM_CUT`, `C2_PLATFORM_EDGE`, `C2_PLATFORM_HOLE`).

## 5. Watch progress without polling

Implement the order-status push contract and 3vjia will call you on every transition —
`CREATE_ORDER`, `C2_SHOP_SUBMIT`, `C2_PLATFORM_SIGN`, `C2_PLATFORM_BATCH`, `C2_PLATFORM_CUT`,
`C2_PLATFORM_SEND`, `C2_PLATFORM_COMPLETE` and the rest. Answer `{"code": 10000}` to acknowledge,
`{"code": 10001}` to decline permanently; anything else causes a bounded retry. Endpoints are
registered through a 3vjia project manager, not through the API. Full enum in
`asyncapi/3vjia-technology-webhooks.yml`.

## Error handling

Every call returns HTTP 200. Branch on `success` in the body. `code: 100100002` means your token is
missing or expired; `code: 100001012` means the path is wrong.
