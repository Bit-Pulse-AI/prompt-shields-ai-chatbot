# Prompt Shields AI Chatbot

[![Licence](https://img.shields.io/badge/licence-Apache--2.0-blue.svg)](LICENSE)

A Next.js chat application used as the reference host for demonstrating Prompt Shields controls against a realistic large language model front end. **The repository name anticipates that work: the code currently on this branch is Vercel's `ai-chatbot` template with no Prompt Shields functionality in it.** See [What this does not do](#what-this-does-not-do).

## The problem

Security controls for AI chat are argued about in the abstract and bought on a slide. What a security architect actually needs before approving one is a running system where the control can be turned off and on: somewhere to see what an unprotected chat interface leaks, what the detection layer catches, what it misses, and what the interruption costs a user mid-task. Without a reference host, evaluation collapses into vendor screenshots, and the false negative rate — the number that decides whether the control is worth deploying — never gets measured.

## Quickstart

```bash
git clone https://github.com/Prompt-Shields/prompt-shields-ai-chatbot.git && cd prompt-shields-ai-chatbot
cp .env.example .env    # set AUTH_SECRET, OPENAI_API_KEY, POSTGRES_URL, BLOB_READ_WRITE_TOKEN
pnpm install
pnpm db:migrate
pnpm dev
```

Runs on http://localhost:3000. Requires Node.js 18 or later, pnpm, a PostgreSQL database, and an OpenAI API key. `pnpm build` runs migrations before building, so the database must be reachable at build time as well as at run time.

## How does it work?

**A reference host is a deliberately ordinary application used as the test subject for a control, so that the control's behaviour is measured against a realistic interface rather than a contrived one.** This is a standard Next.js App Router chat application: server components render the interface, a streaming route handler proxies to the model, and conversation state persists to PostgreSQL.

```
   browser
      |
      v
  +-------------------------------------------+
  | Next.js App Router                        |
  |                                           |
  |  (auth)/  ....... NextAuth v5, credentials |
  |  (chat)/  ....... chat interface, history  |
  |                                           |
  |  api/chat ....... streaming completions    |
  |  api/document ... artefact CRUD            |
  |  api/files ...... upload to Vercel Blob    |
  |  api/vote ....... message feedback         |
  +----------+--------------------+-----------+
             |                    |
             v                    v
   +------------------+   +------------------+
   | Vercel AI SDK    |   | Drizzle ORM      |
   | @ai-sdk/openai   |   | PostgreSQL       |
   +--------+---------+   +------------------+
            v
      OpenAI (gpt-4o, gpt-4o-mini)

   [ intended: a redaction interstitial between the
     composer and api/chat, blocking submission until
     the user accepts or rejects a redaction.
     Not present on this branch. ]
```

**The Vercel AI SDK is a TypeScript library that gives a single interface for streaming completions and tool calls across model providers**, which is why swapping OpenAI for Anthropic or an Azure-hosted model is a change to `lib/ai/index.ts` rather than to the application. Models are declared in `lib/ai/models.ts`; the default is `gpt-4o-mini`.

The interception point for a redaction control is the composer submit handler, before the request reaches `api/chat`. Detection has to run client-side to be honest about the threat model: once the text has been posted to the server, it has already left the user's machine.

## What this does not do

- **It does not detect or redact anything.** This is the material limitation. There is no PII detection, no redaction interstitial, and no Prompt Shields integration in this repository. A prototype implementing those exists in a separate personal working copy and has not been pushed here. Despite the repository name, installing this gives you an unmodified chat template.
- **It is not a security product and must not be presented as one.** Nothing here constitutes a control. If it is demonstrated to a customer or an auditor, it demonstrates the host, not the protection.
- **It is not production-ready.** Credential-based authentication with no rate limiting, no multi-tenancy, no audit logging, and no content retention policy. It is a template.
- **Prompts and completions are stored in plaintext.** Full conversation history persists to PostgreSQL, and uploads go to blob storage. Do not put real customer data into it, and do not deploy it anywhere a real user might.
- **API keys sit in environment variables in the server process.** No key management, no rotation, no per-user attribution of spend.
- **Client-side detection is bypassable by design.** Any control added at the composer can be circumvented by a user who calls the API directly. That is inherent to protecting a user from their own mistake rather than enforcing policy against an adversary, and any evaluation run here should say so.
- **No test suite.** There are no tests in this repository, so any control added here would be unverified by continuous integration.

## Free versus Prompt Shields Cloud

This repository is free and Apache 2.0 licensed in full, and stays that way. The boundary across the product line: **anything an individual engineer needs is free; anything an organisation or an auditor needs is paid.** No capability moves from the free side to the paid side.

| | Free — this repository | Prompt Shields Cloud |
|---|---|---|
| Reference chat host | Complete, no feature gating | Not applicable; Cloud is not a chat client |
| Detection | Intended: the same local scanners shipped in the open-source SDK | Managed detection models, retrained continuously |
| Deployment | Self-hosted, single project, single user | Managed, multi-project, multi-tenant |
| Dashboard | None | Managed risk dashboard, OWASP LLM Top 10 and MITRE ATLAS mapping |
| Telemetry | None; add the SDK yourself | Hosted retention, cross-project alerting and anomaly detection |
| Governance | None | Organisation-wide policy enforcement, versioning and approval workflows, SSO and SAML, SCIM, RBAC |
| Compliance evidence | None | Hash-chained tamper-evident audit logs, EU AI Act Article 12 exports, one-click incident reports |
| Support | Community issues, best effort | Service level agreements, named support, data processing agreement and penetration test report handling |

We do not monetise the code. We monetise hosting, enterprise controls, compliance evidence, and accountability.

## Links

- Documentation: [docs.promptshields.com](https://docs.promptshields.com)
- The controls this is meant to host: [Prompt-Shields/prompt-shields-sdk](https://github.com/Prompt-Shields/prompt-shields-sdk)
- Security policy: this repository has no `SECURITY.md`. Report vulnerabilities privately to security@promptshields.com, never via a public issue. The canonical policy is [prompt-shields-sdk/SECURITY.md](https://github.com/Prompt-Shields/prompt-shields-sdk/blob/main/SECURITY.md).
- Contributing: this repository has no `CONTRIBUTING.md`. See [prompt-shields-sdk/CONTRIBUTING.md](https://github.com/Prompt-Shields/prompt-shields-sdk/blob/main/CONTRIBUTING.md) for the workflow we follow.
- Upstream: [vercel/ai-chatbot](https://github.com/vercel/ai-chatbot), from which this is derived

## Licence

Apache 2.0 — see [LICENSE](LICENSE). Copyright 2024 Vercel, Inc. for the upstream template; modifications are licensed under the same terms.
