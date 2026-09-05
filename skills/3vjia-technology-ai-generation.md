---
name: 3vjia-ai-generation
description: Submit 3vjia AI generation work — text-to-image, video, image-to-model, floor-plan recognition — and poll the task to completion, without burning tenant creation points on duplicate submissions.
api: 3vjia Open Platform API
generated: '2026-09-05'
method: generated
source: openapi/3vjia-technology-open-platform-openapi.yml (三维家AI, 玄尺AI and 楼盘户型 categories) at https://dev.3vjia.com/v1/document
operations:
  - apiV1AsynAiimageT2i
  - apiV1AiimageGetTaskResult
  - apiV1AiModelCreate
  - apiV1AiModelResult
  - apiV1VideoDoubaoVideo
  - apiV1VideoGetTaskStatus
  - apiV1AigcGetResult
  - apiV1AsyncTaskBatchGetTaskStatus
  - apiRenderGetRenderTaskById
  - apiV1BuildingroomGetRoomContentByImage
  - apiV1BuildingroomCheckHouse
  - apiV1ComputeTenantQuantityStatistics
  - apiV1ComputeAccountQuantityStatistics
---

# 3vjia AI generation

Authenticate first — see `3vjia-authenticate`. Every AI operation is submit-then-poll, and every one
of them **spends the tenant's creation points (创作点)**, which are bought in blocks and are finite.

## Check the budget before you spend it

- `apiV1ComputeTenantQuantityStatistics` (查询整个机构全部账号创点数量汇总) — the whole enterprise.
- `apiV1ComputeAccountQuantityStatistics` (查询指定账号自身创点数量统计) — one account.

Do this before a batch. There is no dry-run mode anywhere on this API, and no way to ask what a job
will cost before you submit it.

## Submit, then poll

| Job | Submit | Poll |
|---|---|---|
| Text to image | `apiV1AsynAiimageT2i` (提交异步文生图任务) | `apiV1AiimageGetTaskResult` (查询出图任务结果) |
| Image to model | `apiV1AiModelCreate` (提交AI生模任务) | `apiV1AiModelResult` (查询AI生模任务结果) |
| Video | `apiV1VideoDoubaoVideo` (创建视频生成任务) | `apiV1VideoGetTaskStatus` (获取视频任务状态) |
| Rendering | — | `apiRenderGetRenderTaskById` (AI任务状态查询) |
| Generic AIGC | — | `apiV1AigcGetResult` (根据任务ID获取AI处理结果接口) |
| Many at once | — | `apiV1AsyncTaskBatchGetTaskStatus` (批量查询任务状态) |

A submit returns `data.taskId` — a UUID — usually alongside a queue message. Use
`apiV1AsyncTaskBatchGetTaskStatus` rather than a poll per task when you have several in flight.

## Retry rule

**Never resubmit a job whose submit call timed out.** There is no idempotency key on this API, the
gateway is intermittently slow from outside China, and a duplicate submission is a duplicate charge
against creation points. Poll for the task first; only resubmit if you never received a `taskId`.

## Floor plans

- `apiV1BuildingroomGetRoomContentByImage` (户型识别) — recognise a floor plan from an image.
- `apiV1BuildingroomRecognitionScaleByRoomImg` — recover the drawing's scale.
- `apiV1BuildingroomCheckHouse` (户型检测) — validate a floor plan.
- `apiV1BuildingroomGetRoomContentBySchemeId` — the structural data for an existing scheme.
- `apiV1BuildingroomSearch` (搜索户型) / `apiV1BuildingroomAdd` / `apiV1BuildingroomAddBatch` — the
  cloud property/floor-plan library.

## Uploading source assets

Model, texture and moulding uploads go through a signed direct-to-OSS handshake — get a signature,
then upload to object storage yourself:

- `apiV1ModelOssGetUploadToken` (获取模型上传签名接口)
- `apiV1TextureOssGetUploadToken` (获取贴图上传签名接口)
- `apiV1LineOssGetUploadToken` (获取线条上传签名接口)
- `apiV1NjvrStsGetSign` (获取OSS上传签名接口)

Treat the returned signature as a credential: short-lived, never logged, never persisted.

## Error handling

HTTP 200 on failure, as everywhere on this gateway. Branch on `success`, and expect Chinese-language
`msg` values.
