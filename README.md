# Cutting a production cloud bill by two thirds

A sanitised case study of an AWS-to-Hetzner migration I led as the sole
infrastructure owner of a UK SaaS platform, covering 38 months of invoices.

Absolute figures, the employer's name, vendor quotes and server identifiers
have been removed. Everything is **indexed to the peak month = 100**, so the
shape and the method are intact and no confidential spend is disclosed.

---

## Result

| Measure | Index | Reduction | Evidence |
|---|---|---|---|
| Peak monthly run-rate (2024-04) | 100.0 | — | invoice |
| Latest **closed invoice** month (2026-06) | 32.9 | **67.1%** | invoice |
| Projected after planned deletions | 23.7 | **76.3%** | estimate |

Two numbers, deliberately. The **67% is what the invoices actually show
today**. The 76% is where the run-rate lands once servers already scheduled
for deletion are removed, and it is an estimate, not a receipt. Reporting only
the larger number would have been easy and would not have survived anyone
checking.

## Monthly run-rate, indexed (peak = 100)

```
    2023-05    0.0  ············································
    2023-06    0.0  ············································
    2023-07    0.3  ············································
    2023-08    9.7  ████········································
    2023-09   36.9  ████████████████····························
    2023-10   43.9  ███████████████████·························
    2023-11   46.8  █████████████████████·······················
    2023-12   44.1  ███████████████████·························
    2024-01   73.1  ████████████████████████████████············
    2024-02   82.7  ████████████████████████████████████········
    2024-03   93.4  █████████████████████████████████████████···
    2024-04  100.0  ████████████████████████████████████████████
    2024-05   98.6  ███████████████████████████████████████████·
    2024-06   88.2  ███████████████████████████████████████·····
    2024-07   75.9  █████████████████████████████████···········
    2024-08   55.8  █████████████████████████···················
    2024-09   49.5  ██████████████████████······················
    2024-10   45.9  ████████████████████························
    2024-11   38.2  █████████████████···························
    2024-12   37.6  █████████████████···························
    2025-01   37.5  ████████████████····························
    2025-02   37.3  ████████████████····························
    2025-03   37.6  █████████████████···························
    2025-04   37.4  ████████████████····························
    2025-05   42.8  ███████████████████·························
    2025-06   43.2  ███████████████████·························
    2025-07   49.1  ██████████████████████······················
    2025-08   56.8  █████████████████████████···················
    2025-09   58.7  ██████████████████████████··················
    2025-10   58.2  ██████████████████████████··················
    2025-11   56.1  █████████████████████████···················
    2025-12   56.5  █████████████████████████···················
    2026-01   53.7  ████████████████████████····················
    2026-02   48.4  █████████████████████·······················
    2026-03   47.9  █████████████████████·······················
    2026-04   35.7  ████████████████····························
    2026-05   36.4  ████████████████····························
    2026-06   32.9  ██████████████······························
```

Three things are visible in that curve and worth naming, because two of them
are not flattering.

1. **The ramp to 2024-04 was us**, not the provider. Spend grew with the
   product, and managed-service defaults grew faster than the workload did.
2. **The fall through late 2024 was the compute migration.** Straightforward,
   and the largest single win.
3. **The rise across mid-2025 is real and was not a regression to fix.** The
   platform grew: more services, more environments, a vector database cluster
   built and run in-house. Cost work is not a one-off project you finish; it
   is a rate you manage while the thing underneath you keeps growing.

## How the baseline was established

The temptation in a cost write-up is to pick the flattering comparison. The
method here was fixed before the numbers were looked at.

- **Baseline is the highest invoice-backed month**, not a projection of what
  spend "would have become". Projected counterfactuals are unfalsifiable.
- **Managed-service quotes are held flat across the period.** Two managed
  services (a vector database and a hosted database tier) had no invoice
  history in the exported data, so current vendor quotes were applied as a
  constant baseline rather than modelled. This is conservative in one
  direction and generous in the other; it is stated rather than hidden.
- **Only closed invoice months count.** The month in progress at the time of
  writing was excluded.
- **Provider currency converted at a single stated rate**, not at whatever
  rate made the result look best.

## What was deliberately excluded from the savings claim

The provider granted a material amount of promotional credit across the
migration window. **None of it is counted as savings.**

Credits reduce cash outflow. They do not reduce the cost base, they expire,
and mixing them into an engineering result inflates it by something the
engineering did not do. They are reported in the source document as a separate
line so a reader can see both without the two being confused.

This is the part of the report I would defend hardest. A cost number you
cannot source is not worth making.

## What moved, and in what order

Sequenced by blast radius, not by size, and one at a time.

1. **Application compute** — the bulk of the spend and the least stateful.
   Containerised services moved to owned servers behind a proxy with
   health-gated rolling deploys and automated rollback.
2. **Vector database** — a managed cluster replaced by a self-hosted three-node
   cluster with its own monitoring, load balancing and verified snapshot
   backups. The open-source version of this platform is at
   [hetzner-qdrant-cluster](https://github.com/h4nz0x/hetzner-qdrant-cluster).
3. **Primary databases** — last, deliberately, because they carry the most
   risk. Five managed clusters onto self-hosted replica sets, using
   oplog-tailed continuous replication for the large tiers and dump/restore for
   the two smallest, with a reverse-replication rollback path at every cutover.

Before touching production, the smallest cluster was restored into a
throwaway environment and the entire cutover rehearsed end to end against real
data. That rehearsal took the better part of two weeks and surfaced four
problems that would otherwise have been incidents.

## Honest limits of the evidence

- Cutover dates are operator-provided. Provider resource-creation timestamps
  corroborate when capacity was *built*, not when traffic moved. Vendor
  cancellation records or connection-string changes would make this stronger.
- The managed-service baselines are current quotes held flat, not historical
  invoices. If those services were cheaper earlier, the baseline is slightly
  overstated.
- The 76% figure depends on deletions that were planned, not yet executed, at
  the time of writing.

## What I would do differently

- **Instrument from the start.** The analysis was reconstructed from exported
  invoices after the fact. Tagging resources by service and environment from
  day one would have turned a forensic exercise into a dashboard.
- **Set a cost budget per service** and alert on it the way we alert on
  latency. Nothing paged anyone when spend tripled through 2024.
- **Write the rollback cost down too.** Every migration decision has a reversal
  price, and we reasoned about it without ever recording it.

## Sanitisation

The source document is a four-page internal report generated from exported
provider invoices. Removed for publication: employer and product names,
absolute currency amounts, vendor quotes, credit amounts, server names and
identifiers, internal file paths, and precise cutover dates. Retained: the
method, the indexed series, the sequencing, and the caveats.

A redacted copy of the full report is available on request.

---

*Kyenshak David — [github.com/h4nz0x](https://github.com/h4nz0x) ·
[linkedin.com/in/kyenshak](https://www.linkedin.com/in/kyenshak)*
