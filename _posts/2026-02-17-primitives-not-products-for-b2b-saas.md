---
title: "Revenue Tooling Is Having Its Salesforce Moment"
date: 2026-02-17
tags: [strategy, platforms, b2b-saas, primitives, revenue-tooling]
excerpt_text: "Salesforce, Slack, Shopify, Figma, K8s, Claude Code — the most durable platforms all followed the same playbook. What happens when you apply it to B2B SaaS companies serving revenue teams?"
---

A few weeks after Claude Code shipped hooks and MCP support, I noticed something. A site called [aitmpl.com](https://aitmpl.com) appeared — a community marketplace of Claude Code agents, hooks, skills, MCP servers, commands, and settings. None of it built by Anthropic. All of it built by users who took six extension points and started composing solutions the Claude Code team never anticipated.

I'd written previously about [the primitives philosophy](/blog/ship-primitives-not-products/) — the pattern where the most durable platforms ship composable building blocks rather than finished products. Unix did it with pipes, files, and processes. Kubernetes did it with Pods, Services, and CRDs. Claude Code is doing it with rules, permissions, tools, skills, hooks, and output styles.

But aitmpl.com made the pattern visceral. This wasn't design theory. It was an ecosystem appearing in real time, because the right primitives had been shipped and the right restraint had been exercised.

Which raised a question I couldn't stop thinking about: **can this work for B2B SaaS?**

Specifically — can it work for the kind of B2B software that serves revenue teams? The corner of the software world I spend my days in?

---

## The Tension

B2B SaaS companies sell opinionated products to buyers who want outcomes. A VP of Sales doesn't buy "composable primitives." They buy "a system that tells my reps which deals are at risk and what to do about it." They want answers, not building blocks.

The primitives philosophy says something different. It says the real moat isn't the opinionated answer — it's an ecosystem of user-built solutions. If people don't like your solution, and given that they are the users and they have something better in mind, you should give them an easy way to build it and make it available to others.

Buyers want products. The most durable platform strategy says ship primitives. Can both coexist?

---

## Why Revenue Tooling Is Perfect for This

**RevOps teams are natural builders.** They build Salesforce flows, configure Outreach sequences, write Zapier automations daily. They already think in composable workflows. Unlike a finance team that wants a finished tool, RevOps teams are accustomed to assembling solutions from parts.

**The market is fragmented because every org's GTM motion is different.** A PLG company's revenue workflow looks nothing like an enterprise company with 6-month sales cycles. An outbound-heavy team's needs are unrecognizable to a partner-led team. This is why there are hundreds of revenue tools, each serving a slightly different slice, and why RevOps teams glue them together with Zapier and spreadsheets.

**The ecosystem moat is structurally superior here.** If a competitor builds a better single workflow, they can eat your lunch. But if hundreds of users have published their own compositions on your platform — each tuned to their specific GTM motion — no competitor can replicate that. They'd have to out-build your entire community, not just your product team.

---

## The Salesforce Playbook

This isn't hypothetical. We've watched it happen before — in the same market.

Salesforce started as an opinionated CRM. Sales Cloud was simply better than on-prem Siebel. That's what drove adoption. But then customers wanted customization. Every sales org ran differently, and the one-size-fits-all product left gaps. So Salesforce did something that changed the trajectory of the company: they exposed primitives. Custom objects. Apex. Flows. Visualforce. Then they built AppExchange — a marketplace where admins and ISVs published solutions.

The ecosystem made Salesforce irreplaceable. Not Sales Cloud itself — the 7,000+ apps and the army of Salesforce admins who compose solutions on the platform. The product generated the gravity. The primitives generated the moat.

Here's what makes this relevant today: **Salesforce became the system of record for customer data. But there is no system of record for revenue workflows.**

The orchestration layer — what happens when a deal hits stage 3, who gets notified when churn risk appears, what triggers a retention playbook — doesn't live anywhere. It lives in tribal knowledge, Zapier chains, Slack threads, and spreadsheets. It's the same kind of fragmented, manually-stitched-together landscape that CRM was before Salesforce platformized it.

The opportunity isn't to compete with Salesforce for the data layer. It's to own the orchestration layer above it — the same way Kubernetes owns the orchestration layer above cloud infrastructure without competing with AWS or GCP for compute.

---

## The Bootstrap Problem

Here's a nuance the pure primitives philosophy can gloss over.

**Every successful primitives platform started with a product.**

| Platform | The product that bootstrapped it | Primitives came... |
|---|---|---|
| Salesforce | Sales Cloud (an opinionated CRM) | Years later (Apex, Flows, AppExchange) |
| Slack | A working team messaging app | Webhooks and bots came after mass adoption |
| Shopify | A working e-commerce storefront | App store and Liquid templates followed |
| Figma | A working collaborative design tool | Plugins and community files came later |
| Kubernetes | A working container orchestrator | CRDs came in v1.7, a year+ in |
| Claude Code | A working AI coding assistant | Hooks/MCP/skills layered on after core adoption |

Nobody adopted Salesforce because of AppExchange. Nobody adopted Slack because of the App Directory. Nobody adopted Shopify because of the theme store. They adopted these products because they solved a core problem well. They *stayed* because the ecosystem made them irreplaceable.

You need the opinionated product to generate the gravity that attracts builders. A primitives platform with no users is just an empty SDK.

This means the instinct to skip the product and go straight to the platform is exactly wrong. You need the product. You just need to build it in a way that doesn't foreclose the primitives layer underneath.

---

## The Layered Architecture

The question isn't primitives OR products. It's the layering:

```
Layer 3: Community marketplace (shared workflows, templates)
Layer 2: Opinionated defaults (pre-built solutions — the "golden path")
Layer 1: Composable primitives (the atomic building blocks)
```

Most B2B SaaS today sells Layer 2 without exposing Layer 1. You get the opinionated workflow, and if it doesn't fit your needs, your options are: submit a feature request, build a workaround, or switch vendors.

The right answer is all three layers:

1. **Expose Layer 1** — the composable primitives underneath your product.
2. **Keep Layer 2** as pre-built compositions that demonstrate what's possible. These are your opinionated defaults — the solutions that work for most customers out of the box.
3. **Let Layer 3 emerge** — the community marketplace where users publish and share their own compositions.

Your current opinionated solutions become what Claude Code's `/commit` skill is — an opinionated default workflow that anyone can replace or fork. The default, not the ceiling.

When a customer says "your deal risk scoring doesn't work for our sales process," the answer shifts from "let me file a feature request" to "here are the primitives — compose a scoring workflow that fits your process, and publish it for others who sell the same way."

---

## The Displacement Gradient

Users become the creators of what they want. This fundamentally changes the competitive dynamic — but it's important to understand exactly how.

Primitives don't eliminate displacement risk. They shift what level the competition happens at.

Without primitives, competition is at the **feature level**. A competitor builds a better deal execution workflow, and your customers leave. This is the easiest kind of displacement — one product team out-shipping another.

With primitives, competition shifts to the **abstraction level**. Now a competitor doesn't just need a better workflow — they need better building blocks. And even then, they're competing against the accumulated value of every workflow your community has published. The switching cost isn't your product anymore; it's the ecosystem's output.

K8s had extension points. So did Docker Swarm and Mesos. K8s won anyway — because it picked **better abstractions**. Pod, Service, Deployment mapped more naturally to how people think about workloads. The ecosystem followed the better abstractions.

So the moat isn't "we have primitives and an ecosystem." It's "we have the **right** primitives and an ecosystem built on them." The competition doesn't disappear — it moves one level deeper, to the quality of your abstractions. And at that level, the switching cost equation fundamentally changes: a community doesn't migrate unless the advantage of better primitives decisively outweighs the pain of abandoning everything they've built.

This is a much stronger position than competing on features. Not invincible — but structurally harder to displace.

---

## Finding the Right Atoms

This is the hardest question in the entire strategy: what are the right primitives for revenue orchestration?

My previous post closed with: "The restraint to ship only primitives is hard. The wisdom to pick the right ones is harder." In B2B SaaS, this is even more true. Infrastructure problems have clean abstractions — containers, pods, services. Revenue problems are fuzzy, human, and contextual. The right primitives need to be abstract enough to cover all the variation across GTM motions but concrete enough that a RevOps team can compose them without a PhD in systems thinking.

I have a working framework for this — a specific set of atoms I believe covers the revenue orchestration problem space — but it deserves its own dedicated exploration. I'll lay out that framework, the reasoning behind each primitive, and how they compose in an upcoming post.

---

## The Strategic Sequence

Putting it all together:

**Phase 1: Build the product.** Ship an opinionated solution that solves the core problem well. Generate gravity. This is your Layer 2 — the golden path that earns adoption.

**Phase 2: Expose the primitives.** Refactor your internals so the atomic building blocks are accessible to users. Your opinionated solution becomes a composition of primitives rather than a monolithic product.

**Phase 3: Enable the ecosystem.** Build the infrastructure for users to publish, share, and discover each other's compositions. It doesn't need to be elaborate at launch — every marketplace that exists today started as a simple directory. The value comes from the content, not the chrome.

**Phase 4: Shift the center of gravity.** Over time, the community's compositions become the primary value — and your original opinionated product becomes one template among many. The best default, but not the only option.

---

## The Honest Conclusion

The primitives philosophy is elegant in hindsight. Unix pipes look inevitable now. AppExchange looks obvious. But in the moment — when you're deciding what to abstract and what to keep opinionated, when you're choosing which atoms cover an entire problem space — it's genuinely difficult.

The revenue tooling market is ripe for this. The fragmentation is real. The builder persona exists. The historical playbook is proven. And the white space — a system of record for revenue workflows, not just revenue data — is wide open.

The hardest part isn't deciding to do this. It's identifying the right primitives and sequencing the transition from product to platform. Get it right, and you build an ecosystem that no competitor can replicate by shipping a better product. Because you're not competing on product anymore. You're competing on what your community has built.
