# Prop. II.1 - The contract is written first and is the source of truth

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every API MUST have exactly one OpenAPI 3.1 document, committed in the service repository under `contracts/openapi.yaml`, that passes the company Spectral ruleset in CI. The document MUST be authored or updated *before* the implementing code is merged. Every shared concept in the document MUST be a `$ref` to the shared schema catalogue (Book IV), never a local copy.

## Given

Def. I.2, Def. II.1, Post. I.3, Post. I.6, Post. I.7, Post. II.2, Post. II.6, CN 3, CN 5.

## Demonstration

Teams interact only through contracts (Post. I.3), and for the consumer the contract is the whole interface (Def. I.2, CN 3). Post. I.7 says it is possible to write the contract first and generate from it; Post. II.2 says the document, not the code, is authoritative. If the document were written after the code it would describe what happened to be built rather than what was agreed, and Post. I.6 tells us the two would drift within a year unless a machine checked them; hence the linter and the CI gate (Post. II.6). Finally, by CN 5 a concept with a shared schema may not be re-described locally, so shared concepts appear only by reference. ∎ Q.E.D.

## Corollaries

* **Cor. II.1.1** - Server stubs, clients and validators are generated from the document or verified against it in CI; hand-written DTOs that diverge are defects.
* **Cor. II.1.2** - A change to the document is reviewable as a diff, and a breaking diff (Def. I.14) is detectable by tooling before merge (see Prop. IX.1).
* **Cor. II.1.3** - Documentation is generated from the document; there is no separate API documentation to keep in sync.

## Construction

```
service-repo/
  contracts/
    openapi.yaml            # the contract
    examples/               # request/response examples referenced from the document
  .spectral.yaml            # extends: ["@company/spectral-ruleset"]
  src/Company.Orders.Api/   # implementation
```

* Lint: `spectral lint contracts/openapi.yaml` in CI (fails the build).
* Verify implementation matches: run the generated OpenAPI from the running app (Swashbuckle / built-in `Microsoft.AspNetCore.OpenApi`) and diff against `contracts/openapi.yaml` with `oasdiff`; any difference fails the build.
* Breaking-change gate: `oasdiff breaking <main> <branch>` fails unless the major version changed (Prop. II.7).
* Reference shared schemas by absolute `$id`: `$ref: "https://schemas.company.com/shared/v1/money.json"`.

## Conformance

CI job `contract-lint` (Spectral) and `contract-diff` (oasdiff) present and green. Architecture test: no type in the API project duplicates a catalogue schema (checked by comparing generated schema `$id`s against the catalogue index).

## Scholium

"Contract first" does not mean "contract complete before any code". It means the *change* to the contract is proposed, reviewed and merged before or with the implementation, and that the implementation is derived from it. Iterating on both in the same pull request is fine; merging code whose behaviour the document does not describe is not.
