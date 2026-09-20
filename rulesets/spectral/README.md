# Company Spectral ruleset

The machine check for Book II (Post. II.6). Published as `@company/spectral-ruleset`; services extend it from `.spectral.yaml`:

```yaml
extends: ["@company/spectral-ruleset"]
```

`ruleset.yaml` in this folder is the starting point. Each rule is named in the *Conformance* section of the proposition it enforces; a rule with no proposition is removed, and a MUST proposition with no rule is `Draft` (Prop. I.3).

| Rule | Proposition |
|---|---|
| `paths-kebab-case`, `paths-no-trailing-slash`, `paths-no-verbs`, `path-params-camel-id`, `paths-version-prefix`, `paths-max-depth` | II.2 |
| `operation-success-codes`, `operation-4xx-problem`, `patch-merge-patch-media-type`, `get-no-request-body`, `post-create-returns-201` | II.3, II.4 |
| `collection-returns-page`, `collection-has-cursor-limit`, `no-offset-params` | II.5 |
| `sort-param-shape`, `filter-params-documented` | II.6 |
| `post-has-idempotency-key` | II.8 |
| `item-get-has-etag`, `update-declares-if-match` | II.9 |
| `no-x-headers`, `item-has-metadata` | II.10 |
| `property-names-camel-case`, `no-integer-enums`, `date-time-format`, `money-is-shared-ref`, `id-fields-are-strings`, `top-level-object` | II.11 |
| `operation-has-security`, `security-scheme-oidc` | II.12 |
| `429-declared` | II.13 |
| `get-declares-cache-control` | II.14 |
| `202-has-location-and-operation-body` | II.15 |
| `health-paths-present` | II.16 |
| `id-fields-are-shared-ref`, `timestamp-fields-format`, `date-fields-format`, `no-epoch-numbers`, `enum-is-open-or-justified` | IV.2, IV.4, IV.9 |
