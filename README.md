# cloud disaster recovery: A Practical Guide to RPO, RTO, Failover Testing, and Building an Affordable Off-Site Recovery Plan

If you're searching "cloud disaster recovery," you're probably in one of two situations. Either someone upstairs asked what happens if the servers die tomorrow, or you've just realized that the nightly backup job you set up years ago is not, technically speaking, a recovery plan. Both are excellent reasons to be here.

This guide covers the parts that actually decide whether you recover or just own copies of old data: the two metrics everything hinges on (RPO and RTO), the four standard cloud DR strategies, what this stuff realistically costs, and how to assemble an off-site setup without a hyperscaler invoice that needs its own line of credit.

To keep the numbers grounded rather than theoretical, the second half walks through real, currently published pricing from Sharktech — a long-standing OpenStack cloud and hosting provider — as a worked example of what each piece of a cloud DR stack costs when you buy it à la carte.

## Backup vs. Disaster Recovery: The Difference That Decides Everything

These two terms get used interchangeably in meetings, and the confusion causes real damage. A backup is a copy of your data stored somewhere else. Disaster recovery is the capability to get your operation *running* again — servers, applications, network, and the data underneath them.

A backup answers the question "is my data safe somewhere else?" A DR plan answers "when can we work again?" You can have flawless backups and still be down for two weeks if nobody has thought through how to rebuild.

Two metrics govern everything, and they're worth internalizing:

- **RPO (Recovery Point Objective)** — how much data you can afford to lose, measured backward from the moment of disaster. If you back up hourly, your worst-case data loss is just under an hour. If you back up nightly, it's up to 24 hours of transactions, orders, and emails.
- **RTO (Recovery Time Objective)** — how long you can afford to be down, measured forward from the moment of disaster.

Some concrete examples of how these land in practice: an internal file share can often tolerate an RPO of 24 hours and an RTO of a full day, and nobody suffers. An e-commerce checkout flow can't — every minute of downtime is directly measurable in abandoned carts, and a four-hour data gap means re-shipping orders or refunding customers. A CI/CD build server might not need DR at all; rebuild it from scripts and call it a day.

The mistake teams make is setting one company-wide RTO/RPO. The right approach is per workload: rank what genuinely generates revenue or carries legal obligations, and spend your recovery budget there.

> A backup plan protects your data. A disaster recovery plan protects your business. You need both, and they are not the same document.

## The Four Cloud DR Strategies, From Cheapest to Fastest

The industry's shared vocabulary for cloud disaster recovery comes from the taxonomy AWS laid out in its DR documentation, and it sorts your options into four tiers by cost and recovery speed. The same pattern applies regardless of which cloud you actually use.

| Strategy | What runs before the disaster | Rough RTO | Relative cost | Who it fits |
| --- | --- | --- | --- | --- |
| Backup & restore | Nothing — just data copies in cloud storage | Hours to days | Lowest | SMBs that can tolerate a day+ of downtime |
| Pilot light | Only the core (usually the database) is replicated | Hours | Low–medium | Apps that need data fast but can wait on compute |
| Warm standby | A scaled-down but fully working copy of production | Minutes to an hour | Medium–high | Revenue-critical apps with tight RTOs |
| Multi-site active/active | Everything runs everywhere, all the time | Near zero | Highest | Very few organizations actually need this |

The important nuance: pilot light keeps only the essential data live in the recovery environment and can't serve traffic until you provision the rest, while warm standby can handle requests immediately, just at reduced capacity until you scale it up. That difference is often the entire decision.

For most small and mid-sized businesses, the honest answer is backup & restore for the long tail of workloads, and pilot light or warm standby for the two or three systems that actually print money. Active/active is impressive on architecture diagrams and brutal on invoices.

## What Cloud Disaster Recovery Actually Costs

Traditional DR meant a second physical site with duplicate hardware sitting mostly idle — a data center you paid full price for and hoped to never use. IBM has estimated that owning and running your own DR site costs roughly three times as much as using Disaster Recovery as a Service instead. That's the baseline cloud DR is competing against, and it explains most of its popularity.

For full managed DRaaS, one industry rule of thumb puts the running cost at roughly 120–150% of the cost of the infrastructure being protected — you're essentially paying for a second environment plus someone's engineers to orchestrate it.

The cloud's real contribution is letting you buy *partial* standby instead of a full mirror. A backup-and-restore strategy costs roughly whatever off-site storage costs. A pilot light costs storage plus a small always-on core. Only warm standby starts looking like "double infrastructure," and even then it's a scaled-down double.

One cost category deserves special suspicion: **egress fees**. Most hyperscalers charge for outbound data — roughly $0.09 per GB at standard list rates, which is about $90 per TB. Now think about what a disaster recovery *is*: a mass exodus of your data out of a cloud provider. Restoring 10 TB after an incident can mean a four-figure bill to retrieve your own data, at the single worst possible moment. Any DR design should price the "getting it back out" step before committing, not after.

## Building Your Cloud DR Plan, Step by Step

With the concepts in place, the plan itself is mostly unglamorous diligence:

1. **Risk assessment and business impact analysis.** List what can break — hardware failure, ransomware, accidental deletion, a misconfigured deploy, a data center fire — and what each would actually cost per hour.
2. **Set RPO and RTO per workload.** Not per company. Per workload. This ranking is what determines everything you spend afterward.
3. **Match each tier to a strategy.** Critical systems get warm standby or pilot light; everything else gets backup & restore. Don't gold-plate the file server.
4. **Apply the 3-2-1 rule.** Three copies of your data, on two different types of media, with one copy off-site — ideally in a different provider's infrastructure. The rule has survived decades of ransomware evolution for a reason.
5. **Write the runbook.** Who declares the disaster, in what order systems get restored, where credentials live, and how you communicate with staff and customers while systems are down.
6. **Test it.** Which is important enough to get its own section.

## Testing: The Part Everyone Skips

An untested DR plan is a hope document. Backups that were never restored from are Schrödinger's backups — simultaneously fine and corrupt, and you only find out which during the incident.

Standard practice layers testing from cheap to expensive:

- **Tabletop exercise** — walk through a disaster scenario on paper; argue about who does what. Costs an afternoon, finds gaps in the runbook.
- **Partial failover** — move one non-critical service to the recovery environment and see if it actually works.
- **Data recovery test** — restore individual files and full systems from backup, regularly. This is the single highest-value test per hour spent.
- **Communication drill** — confirm the contact chain actually reaches people at 2 a.m.
- **Full-scale simulation** — fail over everything, once a year or so, and feel the pain in a controlled setting rather than during an actual emergency.

A reasonable cadence for most SMBs: restore tests quarterly, tabletop exercises a couple of times a year, one full simulation annually. If that sounds like a lot of calendar time, compare it to the RTO you just promised the business.

## Assembling the Pieces: A Worked Example with Real Pricing

Here's where it gets concrete. A functional cloud DR setup needs three ingredients: an off-site copy of your data, cheap durable storage for archives, and somewhere to rebuild when things go wrong. Sharktech — a hosting provider that's been around for about two decades per its own site, serving 1,000+ business customers from five data centers (Los Angeles, Las Vegas, Denver, Chicago, Amsterdam) — sells all three pieces on an OpenStack-based cloud. Its published prices are low enough to be worth walking through even if you ultimately build elsewhere.

**Piece one: off-site backups.** Sharktech resells Acronis Cyber Protect as a hosted service starting at **$4.00/month for 200 GB** of protected cloud storage, scaling up from there (their store lists the sync-and-share component up to 100 TB). Beyond raw copies, the service bundles ransomware protection, anti-malware, URL filtering, and patch management, with deduplication to keep storage efficient and encryption on the backups. It covers Windows, Linux, and macOS across physical and virtual machines, on-prem or in a cloud. Backup schedules run daily or hourly — and an hourly schedule puts a hard ceiling of roughly one hour on your RPO. Sharktech's own FAQ states the service includes disaster recovery options for restoring critical systems with minimal downtime, though how you configure that is on you. If you want the entry-level plan, 👉 you can order the 200 GB Acronis Cloud Backup package here.

**Piece two: the archive tier.** Sharktech's S3-compatible object storage is advertised at a flat **$4.90/TB/month** with 1 TB of bandwidth bundled at no charge (the current order-page listing shows the 1 TB package starting at $6.00/month, so expect the entry point in the five-to-six-dollar range). Flat per-TB pricing with no commitment makes it a natural home for the 3-2-1 rule's off-site copy, long-retention archives, and compliance data you rarely touch but must keep. It speaks the standard S3 API, so Veeam-style tools, restic, and CI pipelines all integrate without custom work. For current rates, 👉 check the S3 Object Storage ordering page here.

**Piece three: somewhere to rebuild.** This is the interesting part. Sharktech's Public Cloud doesn't sell fixed VM presets — you get a resource pool (CPU, RAM, and NVMe/SSD/HDD storage) that you carve into as many VMs as the pool allows, which is exactly the shape you want for a recovery environment that has to be reassembled quickly. The platform claims 99.999% uptime with automatic failover across redundant nodes, and — the detail that matters most for DR — **you can download your full server disk images at any time**, for off-site backup, disaster recovery, or simply to take your workloads to another provider. You can also upload your own ISOs and qcow2 images. That's a structural answer to the lock-in problem that makes multi-provider 3-2-1 designs genuinely practical instead of theoretical.

Egress also behaves differently here: inbound traffic is free, 5 TB of outbound is included per month on cloud services, and additional outbound runs $0.002 per GB — $2 per TB, versus roughly $90/TB at typical hyperscaler list rates. On the worst day of your year, when you're pulling everything out to rebuild, that difference is not a rounding error. If you're evaluating the compute side, 👉 the Public Cloud plans and live pricing are listed here.

Below is the full set of currently listed plans relevant to a DR build, with the order-page prices as verified at the time of writing.

| Plan | Role in a DR setup | Key specs | Starting price | Billing | Order |
| --- | --- | --- | --- | --- | --- |
| Acronis Cloud Backup | Off-site backup + anti-ransomware | From 200 GB protected storage; Win/Linux/macOS; physical & virtual; hourly or daily schedules | $4.00/mo | Monthly, usage-based above base | [Get Acronis backup](https://portal.sharktech.net/cart.php?a=add&pid=648&configoption%5B1862%5D=200&configoption%5B1863%5D=0&billingcycle=monthly&aff=1611) |
| S3 Object Storage | Archive / off-site 3-2-1 copy | From 1 TB, flat per-TB rate, S3 API, 1 TB bandwidth bundled | $4.90/TB advertised; 1 TB package listed from $6.00/mo | Monthly | [Get S3 storage](https://portal.sharktech.net/cart.php?a=add&pid=643&carttpl=s3_storage_cart&billingcycle=monthly&configoption%5B1858%5D=13673&configoption%5B1859%5D=1&aff=1611) |
| Public Cloud – Small | Recovery compute / pilot light | 4–16 vCPU, 8–32 GB RAM, 300–2,400 GB SSD (+ NVMe/HDD tiers), 20 TB+ bandwidth | $39.00/mo | Monthly + hourly overage | [Deploy Small](https://portal.sharktech.net/index.php?rp=/store/public-cloud-hosting/public-cloud-hosting-los-angeles-small&aff=1611) |
| Public Cloud – Medium | Warm standby for small apps | 8–32 vCPU, 16–64 GB RAM, 800–6,400 GB SSD | $79.00/mo | Monthly + hourly overage | [Deploy Medium](https://portal.sharktech.net/index.php?rp=/store/public-cloud-hosting/public-cloud-hosting-los-angeles-medium&aff=1611) |
| Public Cloud – Large | Warm standby, serious workloads | 32–128 vCPU, 64–256 GB RAM, 1,500–12,000 GB SSD | $249.00/mo | Monthly + hourly overage | [Deploy Large](https://portal.sharktech.net/index.php?rp=/store/public-cloud-hosting/public-cloud-hosting-los-angeles-large&aff=1611) |
| Public Cloud – Enterprise | Full production-class recovery | 64+ vCPU, 128 GB+ RAM, 5,000 GB+ SSD, uncapped resource maximums | $499.00/mo | Monthly + hourly overage | [Deploy Enterprise](https://portal.sharktech.net/index.php?rp=/store/public-cloud-hosting/public-cloud-hosting-los-angeles-enterprise&aff=1611) |
| Dedicated Cloud | Fixed, predictable warm standby | 8–512 vCPU, 16–1,024 GB RAM, SSD/HDD/NVMe, 5–300 TB transfer | $86.23/mo | Fixed monthly (prepaid resources) | [Get Dedicated Cloud](https://portal.sharktech.net/index.php?rp=/store/dedicated-public-cloud-bare-metal/dedicated-cloud&aff=1611) |

For context on the overage rates: above a plan's included commit, public cloud resources bill hourly at $0.0025 per vCPU core, $0.0035 per GB of RAM, and $0.00006 per GB of SSD per hour, with NVMe and HDD tiers cheaper still — so a burst-capable plan stays capped on cost by design. If your recovery target is genuinely tiny, Sharktech also sells Proxmox-based Smart VPS from $7.95/month (Xeon Gold cores, NVMe, 60 Gbps DDoS protection, multi-region), though note that during research the VPS order page was showing a temporary out-of-stock notice — worth confirming availability before you anchor a DR plan to it. 👏 The full catalog, including VPS, is browsable here through [the Sharktech portal](https://bit.ly/SharKTech).

Now put the pieces together and the math becomes tangible. A pilot-light design for a small business: Acronis on an hourly schedule for the critical servers, 1 TB of S3 as the independent archive copy, and a Small cloud pool provisioned and rehearsed but only really consumed during recovery — roughly **$10–13/month at rest**, plus compute when you actually need it. A warm standby: same backup layer, plus a Medium cloud tier running your scaled-down production at $79/month. Compare that to the 120–150%-of-infrastructure rule of thumb for managed DRaaS, and you can see why self-assembled stacks appeal to teams with a sysadmin on staff. The trade-off is the next section.

## Before You Buy: The Honest Part

A few things worth saying plainly, because a DR decision is the wrong place for marketing vapor.

First, **this is infrastructure, not managed DRaaS.** Nobody at Sharktech is contractually obligated to drive your failover at 3 a.m. or sign an SLA around your RTO. You're assembling the ingredients; the recovery orchestration, monitoring, and runbook are yours (or your MSP's). If you need an external team on the hook for a guaranteed recovery time, enterprise DRaaS platforms are a different product at a different price — and that's a legitimate choice for organizations without in-house ops.

Second, the reviews picture is thin. Sharktech's Trustpilot score sits in the mid-3s out of 5, but from a small sample of reviews — not enough data to say much either way. The customer testimonials the company publishes lean heavily on DDoS protection and support responsiveness (gaming and IDC operators feature prominently), and its HostAdvice recognition is self-cited. For a workload this important, do a reference check or a paid trial month before you bet the runbook on it.

Third, treat the headline claims as claims. "40–80% cheaper than hyperscalers" is Sharktech's own framing. The verifiable part is the bandwidth math — free inbound, $0.002/GB outbound versus roughly $0.09/GB list at the big three — which holds up. The rest depends entirely on your workload's shape, so run your own numbers.

Finally, a DR plan that depends on a single provider is itself a single point of failure. The 3-2-1 rule exists precisely so that your recovery copy lives somewhere structurally different from your production copy. Sharktech's free disk-image downloads make that easy here, but the principle applies regardless of who you buy from.

## Quick Answers to Common Questions

**Is cloud backup the same as cloud disaster recovery?** No. Backup secures the data; disaster recovery restores operations. Every DR plan contains backups, but backups alone are not a DR plan.

**How much data will we lose in a disaster?** Exactly as much as your RPO allows. Whatever your least frequent backup interval is, that's your worst-case data loss. Hourly backups cap it near an hour; nightly backups cap it near a day.

**Do we need a warm standby?** Only if the cost of downtime justifies running a second environment continuously. A useful exercise: multiply your revenue per hour by your plausible outage duration, then compare it to $79/month.

**How often should we test?** Restore individual backups at least quarterly — most backup corruption is discovered here. Run a tabletop review twice a year and one full failover simulation annually.

## The Bottom Line

The disaster recovery plans that actually work are boring: defined RPO and RTO per workload, an off-site copy following 3-2-1, somewhere to rebuild, and a test calendar that gets honored. The cloud made the "somewhere to rebuild" part dramatically cheaper — whether that's Sharktech's OpenStack pools at $39/month or anyone else's — and providers with free inbound traffic and cheap egress have quietly removed the most perverse cost in the old model: paying a toll to recover your own data.

Start with the $4 backup plan and an hourly schedule if that's what the budget allows this quarter. An imperfect, tested, off-site recovery path beats a beautifully architected diagram that exists only in a slide deck.
