# Seekr

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
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Seekr Technologies builds explainable, auditable, sovereign AI for regulated industries and
high-stakes government missions. Its platform, **SeekrFlow**, covers AI-ready data preparation,
vector databases and retrieval, fine-tuning, model deployment, OpenAI-compatible inference and
agent orchestration — wrapped in an explainability layer that traces every response back to the
chunks, tool calls, spans and training examples that produced it.

- Website: https://www.seekr.com/
- Developer docs: https://docs.seekr.com/
- API reference: https://docs.seekr.com/flow/reference/getting-started-with-your-api
- API base URL: `https://flow.seekr.com/v1`
- MCP server: https://docs.seekr.com/mcp

### What this profile holds

| Artifact | Notes |
|---|---|
| `openapi/` | **Four OpenAPI 3.1.0 documents, 212 operations.** Seekr publishes no static spec URL — these were harvested through Seekr's own MCP server (`query_docs_filesystem_seekr`), which exposes `/openapi/{agents,explainability,llm-training,serving}.json`. Byte-verbatim copies are under `openapi/_original/`. |
| `mcp/` | The published documentation MCP server, its live `tools/list`, and a tool crosswalk. |
| `a2a/` | A real A2A agent card served from `docs.seekr.com`, graded against A2A 1.0.0. |
| `skills/` | Seekr's own published Agent Skill (verbatim) plus four generated skills grounded in verified `operationId`s. |
| `conventions/`, `errors/`, `lifecycle/`, `changelog/`, `data-model/`, `conformance/`, `security/`, `packages/`, `well-known/`, `overlays/`, `agentic-access/` | Derived, probed or searched — see each file's `method:` frontmatter. |

Company also tracked on the secondary market at https://forgeglobal.com/seekr_stock/.
