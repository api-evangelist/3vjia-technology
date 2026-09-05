# Source documentation (verbatim)

These two files are the **verbatim, unmodified** JSON responses of 3vjia's own public
documentation API — the service that renders <https://dev.3vjia.com/v1/document>:

| File | Fetched from | Date | HTTP |
|---|---|---|---|
| `3vjia-technology-open-platform-api-catalog.json` | `https://devapi.3vjia.com/document/getNewApiDocumentCategoryList` | 2026-09-05 | 200 |
| `3vjia-technology-open-platform-legacy-catalog.json` | `https://devapi.3vjia.com/document/getApiDocumentCategoryList` | 2026-09-05 | 200 |

Each entry was then read individually from
`https://devapi.3vjia.com/document/getNewApiDocument?apiId=<id>` (430 records) and
`https://devapi.3vjia.com/document/getApiDocument?apiId=<key>` (99 legacy records).

`../3vjia-technology-open-platform-openapi.yml` is a **mechanical transform of those
records** — every path, method, parameter, type, required flag, description and example
is verbatim from them. It is marked `method: derived` and is not a specification 3vjia
publishes. 3vjia publishes no OpenAPI, Swagger, GraphQL SDL, AsyncAPI or Postman
collection at any probed location.
