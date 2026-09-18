# Cutting a production cloud bill by nearly three quarters

A sanitised case study of an AWS-to-Hetzner migration I led as the sole
infrastructure owner of a UK SaaS platform, built from 38 months of AWS
invoices and a full Hetzner inventory.

The employer's name, absolute amounts, vendor quotes and server identifiers
have been removed. Figures are **indexed to the baseline month = 100**, so the
method and the proportions survive and no confidential spend is disclosed.

---

## Result

| | Index |
|---|---|
| Baseline — 2024-04: AWS invoice plus managed vector-database and database quotes | 100.0 |
| Current — latest closed AWS invoice (2026-06) plus the whole Hetzner estate | 27.6 |
| **Reduction** | **72%** |

The internal report this comes from headlines about 76%. That figure left out
three servers that were planned for deletion at the time. They have since been
kept, so this write-up counts them, and the reduction is about **72%**. When
the assumptions change, the number should change with them.

## Where the reduction came from

Indexed to the same baseline (baseline total = 100):

| Component | Before | After | Change |
|---|---|---|---|
| Compute and other AWS |  74.4 |  17.4 | 77% lower |
| Vector database |  17.7 |   6.3 | 64% lower |
| Primary databases |   7.8 |   3.9 | 50% lower |
| **Total** | **100.0** | **27.6** | **72% lower** |

"After" for compute includes every other server in the Hetzner account,
including a few that are not part of the migrated workload. That overstates
current cost slightly, which makes the comparison conservative.

## AWS spend, month by month (indexed to the AWS peak = 100)

This series is AWS invoices only. It does not include the managed services or
Hetzner, so it shows how AWS spend moved, not total run-rate.

```
    2023-05    0.0  ············································
    2023-06    0.0  ············································
    2023-07    0.5  ············································
    2023-08   13.3  ██████······································
    2023-09   50.6  ██████████████████████······················
    2023-10   60.2  ██████████████████████████··················
    2023-11   64.1  ████████████████████████████················
    2023-12   60.4  ███████████████████████████·················
    2024-01   63.1  ████████████████████████████················
    2024-02   76.3  ██████████████████████████████████··········
    2024-03   90.9  ████████████████████████████████████████····
    2024-04  100.0  ████████████████████████████████████████████
    2024-05   98.1  ███████████████████████████████████████████·
    2024-06   83.8  █████████████████████████████████████·······
    2024-07   67.0  █████████████████████████████···············
    2024-08   39.3  █████████████████···························
    2024-09   30.7  ██████████████······························
    2024-10   25.8  ███████████·································
    2024-11   15.3  ███████·····································
    2024-12   14.5  ██████······································
    2025-01   14.3  ██████······································
    2025-02   14.0  ██████······································
    2025-03   14.4  ██████······································
    2025-04   14.2  ██████······································
    2025-05   21.6  ██████████··································
    2025-06   22.1  ██████████··································
    2025-07   30.2  █████████████·······························
    2025-08   40.8  ██████████████████··························
    2025-09   43.3  ███████████████████·························
    2025-10   42.7  ███████████████████·························
    2025-11   39.8  █████████████████···························
    2025-12   40.3  ██████████████████··························
    2026-01   36.5  ████████████████····························
    2026-02   29.3  █████████████·······························
    2026-03   28.6  █████████████·······························
    2026-04   11.8  █████·······································
    2026-05   12.8  ██████······································
    2026-06    8.0  ████········································
```

AWS spend peaked in 2024-04, fell sharply through the second half of
2024, rose again through mid-2025, and has fallen since early 2026 to about
8% of its peak.

## How the baseline was chosen

The comparison was fixed before the conclusion, and the choices are stated
openly because each one affects the result.

- **The baseline is the highest AWS invoice month.** That is the favourable
  end of the range, and it is chosen because it is a real invoice rather than
  a projection of what spend "would have become". It is stated here so a
  reader can discount it.
- **Managed-service costs are current vendor quotes, held flat.** Neither
  managed service appears in the exported AWS invoices, so their quotes stand
  in for them. Where a quote was a range, the **lower end** is used.
- **Hetzner is priced from inventory, not invoices.** Every server was pulled
  from the provider API and priced by creation date against the published
  rates. Volumes, load balancers and IPs are included.
- **One conversion rate is used throughout**, EUR to USD at 1.1392.

## What was deliberately excluded

The cloud provider granted a material amount of promotional credit during the
migration. **None of it is counted.** Credits lower the bill in the months they
apply. They do not lower the cost base, they expire, and counting them would
make the engineering look better than it was. The source report shows them on
a separate line.

## What moved, and in what order

Sequenced by risk rather than size, one piece at a time.

1. **Application compute and supporting services**, the largest share of spend
   and the least stateful, moved to owned servers behind a proxy with
   health-gated rolling deploys and automated rollback.
2. **The vector database.** A managed cluster was replaced by a self-hosted
   three-node cluster with its own load balancing, monitoring and verified
   snapshot backups. Built January 2026, cut over in February. The platform is
   published as [hetzner-qdrant-cluster](https://github.com/h4nz0x/hetzner-qdrant-cluster).
3. **The primary databases**, last because they carry the most risk. Five
   managed clusters moved onto self-hosted replica sets. The larger ones used
   continuous oplog replication, the two smallest used dump and restore, and
   every cutover had a reverse-replication rollback path. Built June 2026,
   final cutover July 2026.

Before any production cutover, the smallest cluster was restored into a
throwaway environment and the whole cutover rehearsed against real data,
including deliberately injected failures, before the rehearsal counted as done.

## Limits of the evidence

- Cutover dates come from the operator. Provider timestamps show when
  capacity was built, not when traffic moved.
- The managed-service costs are current quotes held flat, not historical
  invoices.
- Hetzner is priced from published rates. If the provider's invoice bundles or
  discounts anything, the invoice should replace the estimate.
- The baseline is the single highest AWS month. A multi-month average would
  give a smaller reduction.

## What I would do differently

- **Tag resources by service and environment from the start.** This analysis
  was rebuilt from exported invoices after the fact. It should have been a
  dashboard.
- **Give each service a cost budget and alert on it**, the same way we alert on
  latency.
- **Write down the cost of reversing each migration step**, not only the cost
  of taking it.

---

*Kyenshak David — [github.com/h4nz0x](https://github.com/h4nz0x) ·
[linkedin.com/in/kyenshak](https://www.linkedin.com/in/kyenshak)*
