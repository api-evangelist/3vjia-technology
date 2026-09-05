# 3vjia Technology

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

## 3vjia Technology (三维家 / AiHouse)

3vjia Technology — Guangdong Sanweijia Information Technology Co., Ltd., international brand
**AiHouse** — is a Guangzhou home-furnishing industrial-software company founded in 2013. It runs a
cloud 3D design and manufacturing platform for the interior-decoration and custom-furniture industry:
3D cloud design, AI Dream Home, AI Light Design, CAD and rendering engines, the DMS order-splitting
system, the MOS/MCS manufacturing execution systems and the AIMES manufacturing platform, joining
design, quotation, order placement, splitting and factory production into one chain.

### What we found

3vjia runs a public **Open Platform** at <https://dev.3vjia.com/> whose gateway
`open-gateway.3vjia.com` exposes **429 documented operations**. It publishes **no OpenAPI, Swagger,
GraphQL SDL, AsyncAPI, Protobuf, WSDL or Postman collection** — but it does publish a complete,
anonymously readable, machine-readable documentation service at `devapi.3vjia.com`, which returns per
operation the HTTP method, absolute URL, every parameter with type and required flag, every response
field, and request/response examples.

`openapi/3vjia-technology-open-platform-openapi.yml` is a **mechanical transform of those records**
(`method: derived`), and the verbatim catalogs it was built from are kept in
`openapi/_source-documentation/`. Nothing was invented: every path, method, parameter and example is
the provider's own, and each operation carries an `x-source-documentation` link back to the page it
was read from. A live probe on 2026-09-05 confirmed the paths are real — documented paths return
`code 100100002` (missing credential), invented paths return `code 100001012` (resource does not exist).

### Things an integrator should know

- **Failures come back as HTTP 200.** Success, missing credential and unrouted path all return `200`;
  the outcome is in the body's `success` field.
- **There is no idempotency** on any of the 429 operations, and no dry-run mode.
- **Only one `access_token` is valid per application** — a new one invalidates the previous one, so
  the provider requires an enterprise-wide central token service.
- **No published rate limits, no API changelog, no status page, no deprecation policy, no SLA.**
- **No SDK** in any public registry; the two 3vjia GitHub organizations hold zero public repositories.
- **No MCP server, no agent card, no `llms.txt`, no `/.well-known/` documents** on any of eight hosts.
- Documentation is Chinese-only.

### Links

- Website — <https://www.3vjia.com/>
- Open Platform — <https://dev.3vjia.com/>
- API documentation — <https://dev.3vjia.com/v1/document>
- Help center — <https://www.3vjia.com/helpcenter>
- Pricing (design software; API access is contact-sales) — <https://mall.3vjia.com/>
- International brand — <https://www.aihouse.com/>

_Surfaced via the API Evangelist harvest backlog (source: secondary-market)._
