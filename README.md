# SCRAP

**Tax intelligence over Brazil's official fiscal sources — with every finding traceable back to the record it came from.**

SCRAP is a multi-tenant SaaS platform for accounting firms and tax-law practices. It reads Brazil's official fiscal sources through contracted and public APIs, builds each taxpayer's fiscal timeline, and surfaces evidence-backed *indications* for a qualified professional to review.

This is the public presentation repository. **SCRAP is proprietary software. No source code, prompts, rule definitions, credentials or customer data are included here or will be.**

---

## The problem

An accounting firm carrying hundreds of client companies has no consolidated view of what each one has paid, owes, declared or been assessed. That information is spread across separate federal systems, each with its own access path and its own format.

Two things go unnoticed as a result. Money sits unallocated or paid twice, and nobody reconciles it. Debts pass their statutory limitation date without anyone raising the question. Finding either one means a person pulling reports one taxpayer at a time — which is why, in practice, it does not get done.

There is a second, separate problem. A firm looking for new clients has no way to tell which companies around it are carrying federal debt. The data is public, but it is published as quarterly bulk files of tens of millions of rows. It exists and it is unusable.

## What SCRAP does

Two products on one platform.

### Portfolio

For taxpayers who have formally authorised the firm to act for them.

| Stage | What happens |
|---|---|
| **Authorise** | Registers the firm's client portfolio and the electronic authorisation that permits access |
| **Ingest** | Reads the official sources, persisting every raw response before anything is interpreted |
| **Normalise** | Reconciles payments, debts, instalment plans and judicial events into one fiscal timeline per taxpayer |
| **Analyse** | Applies rule engines to surface indications — unallocated or duplicated payments, debts approaching or past limitation |
| **Evidence** | Attaches the source records behind every indication, so the finding can be checked rather than trusted |
| **Report** | Produces dossiers and exports for the professional who signs off |

### Radar Nacional

Prospecting over Brazil's public active-debt register. Search by company or by profile — region, sector, size, debt band — and see the picture over time: whether a company's debt is growing or shrinking, whether it has entered and left the register repeatedly, or whether it has never appeared at all.

The register is published quarterly at national scale. SCRAP keeps a usable history of it rather than only the current snapshot, which is what makes the trajectory answerable.

## Sources

SCRAP reads only contracted or public APIs. **There is no scraping.**

- **Integra Contador (SERPRO)** — contracted federal tax API, accessed over mTLS
- **PGFN open data** — the national active-debt register, published quarterly
- **DataJud** — judicial proceedings
- **Official federal registry data** — for corporate identification and cross-referencing

## Design principles

These are the properties the system was built around, and the reason it can be used on client data at all.

**Indications, not determinations.** SCRAP does not decide that a debt is time-barred or that a payment was misapplied. It presents the evidence and the reasoning, and a qualified professional decides. The distinction is deliberate and it is enforced in the product's language, not only in a disclaimer.

**Raw before normalised.** Every response from an official source is persisted with its origin, timestamp and content hash *before* anything parses it. If an interpretation is later found wrong, the original record is still there to re-derive from.

**Tenant isolation at the database.** Multi-tenancy is enforced at the data layer, not only in application code. Authentication requires MFA. Access to a taxpayer's data — read or write — writes an audit event recording who, what, when and for what purpose.

**Data minimisation by construction.** The national prospecting index holds legal entities only. Personal data that arrives masked from the source is not stored. This is a structural choice, not a filter applied at query time.

**Metered by design.** Every call to a paid external service is counted per tenant and per service as part of the normal path, not as an afterthought. Cost is a first-class property of the system.

**Idempotent ingestion.** Any ingestion can be re-run without duplicating what it already brought in.

**Bulk data stays away from the screen.** National-scale tables are never queried directly by anything serving a user interface. Interactive work runs against prepared structures; full scans belong to maintenance.

## Current status

SCRAP operates against SERPRO's **trial environment** and against a **test** payment rail. It is not, at the time of writing, transacting against production federal services or processing real payments.

The product is in active development. Portfolio administration, client management, the Radar Nacional module and the federal tax-document flows are implemented; other catalogue entries are in progress. Interface language is available in more than one locale.

## What this project does not claim

Stated plainly, because in this domain the boundaries matter more than the features:

- It does **not** determine that a debt is time-barred. It raises an evidenced indication for a tax professional to assess.
- It does **not** replace the judgement of an accountant or a tax lawyer.
- It does **not** access any taxpayer's data without that taxpayer's formal electronic authorisation.
- It does **not** guarantee that the official sources are complete or current. Its findings are bounded by what those sources publish.
- It does **not** cover every federal obligation. Coverage is explicit and bounded.
- Prospecting data is public information republished by the PGFN, with the accuracy and the lag that implies.

## Licensing

SCRAP is proprietary. Publishing this material does not grant any licence to the product, its source code, its rule definitions, or its methodology. No content licence has been applied to this repository.

---

*A product by [LM TEC](https://www.lm.tec.br) · Contact: [Luciano Manetta](https://www.linkedin.com/in/luciano-manetta-45276514/)*
