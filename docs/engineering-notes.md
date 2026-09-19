# Engineering notes

Context on how SCRAP is built, at the level of *what* rather than *how*. Implementation, rule encodings, data-layer strategy and development methodology are proprietary and not described here.

## Shape of the codebase

SCRAP is a modular monolith. At the time of writing:

- **2,115 versioned files**, of which **452 are TypeScript**
- **82 data models** in a single tracked schema
- **70 database migrations** with a preserved progression, not a flattened snapshot
- **290 commits**
- Separate front-end SPA, internationalised for more than one locale
- Automated tests at unit, acceptance and integration level, the last running against a disposable local database

Every task carries its own acceptance test, and an acceptance test is never relaxed to make a build pass. That rule is written down, and it is the one that costs the most to keep.

## Stack

Deliberately conventional, because a product handling taxpayer data is a poor place for novelty:

**Backend** — Node.js 22 LTS · TypeScript 5 (strict) · NestJS 11 · PostgreSQL 18 · Prisma · job queue for asynchronous work

**Frontend** — React 18 · Vite · Tailwind CSS · shadcn/ui · react-i18next

**Testing** — Vitest, unit / acceptance / integration

## Security and tenancy posture

- Multi-tenant isolation enforced at the **database** layer, not only in application code
- Authentication with **MFA**
- **Audit trail** on every read or write of taxpayer data, recording purpose as well as actor
- Digital certificates held encrypted per tenant and administered under MFA
- Secrets excluded from version control by policy and verified as such

## Scale

The national prospecting data is the part that shapes the architecture. The public active-debt register publishes on the order of **tens of millions of rows per quarter**, with roughly **six years** of quarterly publications available. Keeping a usable history of that — rather than only the latest snapshot — while interactive queries stay responsive is the module's central engineering constraint.

## Measurement standard

A performance number obtained on a synthetic dataset smaller than production is not evidence that a query is fast. Measurements count only against real volume and real distribution. This is written into the project's rules, because the temptation to report the convenient number is constant.

## Engineering standard

The relevant claim about SCRAP is not that it covers a lot of ground. It is that the output carries the properties this domain requires — tenant isolation, audit trails with stated purpose, raw source records retained before interpretation, findings presented as evidenced indications rather than conclusions, and cost metered per tenant as a normal part of the path.

## Platform reuse

Core infrastructure — multi-tenancy, authentication, auditing, and the government-API client layer — is shared with other products in the same portfolio rather than rebuilt per project. New products in regulated domains start from a proven base.

---

*Back to the [project overview](../README.md).*
