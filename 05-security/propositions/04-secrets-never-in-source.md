# Prop. V.4 - Secrets are never in code, configuration files or committed environment

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A secret ([Def. V.8](../definitions.md#Def.%20V.8%20-%20Secret)) MUST NOT appear in source code, in any configuration file, in a container image, or in an environment variable definition checked into source control. A service MUST read each secret at start-up or on demand from AWS Secrets Manager or SSM Parameter Store ([Post. V.3](../postulates.md#Post.%20V.3%20-%20Managed%20Secret%20Stores)) using its IAM role ([Post. V.4](../postulates.md#Post.%20V.4%20-%20Least%20Privilege)).

## Given

[Def. V.8](../definitions.md#Def.%20V.8%20-%20Secret), [Post. V.3](../postulates.md#Post.%20V.3%20-%20Managed%20Secret%20Stores), [Post. V.4](../postulates.md#Post.%20V.4%20-%20Least%20Privilege), [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification), [Def. I.21](../../01-foundations/definitions.md#Def.%20I.21%20-%20Environment).

## Demonstration

By [Post. V.3](../postulates.md#Post.%20V.3%20-%20Managed%20Secret%20Stores) the managed stores are the only place secrets exist; a secret elsewhere is a second copy that is neither rotated nor access-controlled. Source control is readable by every engineer and every CI job, so a committed secret is disclosed to every one of them, which by [Def. V.8](../definitions.md#Def.%20V.8%20-%20Secret) is the harm the word names. Contracts are identical across environments and only configuration differs ([Def. I.21](../../01-foundations/definitions.md#Def.%20I.21%20-%20Environment)), so the reference to a secret (its name) may be committed while its value is resolved per environment at runtime. [Post. I.6](../../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification) says the rule is only real if a scanner enforces it. ∎ Q.E.D.

## Corollaries

* **Cor. V.4.1** - Local development uses the same code path against a developer's own AWS credentials or `dotnet user-secrets`; there is no "dev" branch of configuration that contains values.
* **Cor. V.4.2** - A secret that is ever committed is treated as disclosed and rotated, even if the commit is later rewritten.

## Construction

* AWS Secrets Manager for credentials with rotation; SSM Parameter Store SecureString for static secrets; KMS customer-managed key per account.
* `Amazon.Extensions.Configuration.SystemsManager` and `Kralizek.Extensions.Configuration.AWSSecretsManager` (or the AWS SDK directly) as `IConfiguration` providers, named by path prefix per service and environment.
* IAM role per service with `secretsmanager:GetSecretValue` and `ssm:GetParameters*` scoped to that service's path.
* ECS task definition `secrets` block or Lambda extension for Parameters and Secrets where injection at start-up is preferred.
* Pre-commit and CI secret scanning: `gitleaks` with the company rule set.

## Conformance

CI policy: `gitleaks` blocks merge on any finding. AWS Config rule: no ECS task definition or Lambda configuration has plaintext environment variables matching secret patterns. Architecture test: no `appsettings*.json` contains a key matching the secret naming convention.

## Scholium

Environment variables set by the platform at runtime from Secrets Manager are permitted; the prohibition is on values committed to source. The distinction is where the value is written down, not how the process reads it.
