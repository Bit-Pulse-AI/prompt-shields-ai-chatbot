# Contributing to Prompt Shields AI Chatbot

This repository is a reference host: a deliberately ordinary chat application
used to evaluate Prompt Shields controls against a realistic interface. It is
not a security product, and contributions should not make it look like one.

## Before you start

Open an issue first for anything larger than a fix. It is derived from Vercel's
`ai-chatbot` template, so gratuitous divergence from upstream makes future
rebases harder — say why the change needs to live here.

## Development setup

Requires Node.js 18 or later, pnpm, a PostgreSQL database, and an OpenAI API key.

```bash
cp .env.example .env    # AUTH_SECRET, OPENAI_API_KEY, POSTGRES_URL, BLOB_READ_WRITE_TOKEN
pnpm install
pnpm db:migrate
pnpm dev
```

## Checks that must pass

```bash
pnpm lint
pnpm build
```

There is no test suite. Any detection logic added here should arrive with one —
an evaluation host whose own behaviour is unverified proves nothing.

## If you add a control

- **Say plainly what it does not catch.** The value of this repository is
  measuring a false negative rate, which requires being honest about it.
- **Client-side detection is bypassable, by design.** It protects a user from
  their own mistake; it does not enforce policy against an adversary. Do not
  describe it as though it does.
- **Never put real data in it.** Prompts and completions persist to PostgreSQL
  in plaintext.
## Coding conventions

Match the surrounding code. Comment density, naming, and idiom should be
indistinguishable from what is already there. A change that reads as though it
were written by a different person is harder to review, whatever its merits.

## Never commit

- Live credentials, API keys, tokens, or connection strings with real passwords
- Customer data, real prompt text, or anything that identifies a person
- Generated artefacts, build output, or editor and OS scratch files
- Roadmap phases, customer names, pricing strategy, or other internal material.
  This repository is public.

If you believe a credential has been committed, email
**security@promptshields.com** immediately rather than opening a pull request
that removes it — a public commit that deletes a secret advertises the secret.

## Pull requests

- One logical change per pull request.
- Say what you changed and why. If you fixed a defect, say how you reproduced it.
- State what you verified, and how. "Tests pass" is only useful if you ran them.
- If a claim in the README stops being true because of your change, update the
  README in the same pull request.

By contributing you agree that your contributions are licensed under the same
terms as this repository, and you confirm you have the right to grant that
licence.

## Conduct

Participation is governed by [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).
Vulnerabilities go to [SECURITY.md](SECURITY.md), never to a public issue.
