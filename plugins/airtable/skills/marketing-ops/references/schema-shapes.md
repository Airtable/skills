# Schema shapes for marketing-ops scaffolding

Field-by-field detail for the schema shapes named in `SKILL.md`. Load the section that matches the scope answers; don't read the whole file.

Each shape comes in four variants — **B2B** (named accounts, ABM motions if relevant), **consumer / DTC** (cohorts and segments, lifecycle stages, app-store / channel-source ingestion), **mixed / B2B2C** (both), and **agency** (multi-client, retainer drawdown, client portal). The base structure is the same across variants; the variants add specific tables and fields. Pick the variant from the third scope question (audience shape) plus the optional agency branch.

**Two cross-cutting notes that apply across every shape**:

-   **AI fields** are a native typed-field capability — use them on any table that benefits from per-record AI output (categorization on Requests, expansion on Briefs, tagging on Assets, narrative summaries on Performance / Budget / Approvals, translation on locale-variants). Configurations and tier-gating evolve; check `support.airtable.com` for current AI-field capabilities before scaffolding. Each AI field below pairs with a human-review gate per the copilot pattern described in `references/sub-workflows.md`.
-   **Match the schema to the native view / Interface component it scaffolds.** Kanban needs a singleSelect (or single-link or collaborator) for stacking. Calendar needs a date field. Gallery wants an Attachment field as the cover. Timeline needs Start (and optionally End) dates. Gantt needs a self-linking linked-record on the work-items table for dependencies (FS-only); end-date-only records render as milestones. Form view doesn't surface computed fields and can't create new linked-records inline. For current required-field and tier-gating constraints on any native view or Interface component, verify against `support.airtable.com` before scaffolding — a wrong-shape schema produces a base that won't render the intended view.

## Lightweight tracker (2-3 tables)

For solo marketers, small teams replacing spreadsheets, music release drivers, and book publicity trackers. The dominant SMB shape. Customer language to expect: _"a mess of Excel and Google Sheets calendars,"_ _"replaces manual Excel tracking,"_ _"avoid the cost of [Hootsuite / Sprout],"_ _"team of two with high cognitive load."_ Don't impose multi-tier structure they won't use.

### Tables

-   **Campaigns** — the master object.
    -   `Name` (singleLineText, primary)
    -   `Status` (singleSelect: Idea, In progress, Approved, Live, Done, Won't do) — color-code red / yellow / blue / green / grey
    -   `Channel` (multipleSelects: Email, Social, Web, Paid, SMS, PR, Event, Other)
    -   `Owner` (singleCollaborator)
    -   `Start date` (date), `End date` (date)
    -   `Brief / description` (multilineText)
    -   `Assets` (multipleAttachments) or `Linked assets` (multipleRecordLinks → Assets table if one exists)
    -   `Created` (createdTime), `Last updated` (lastModifiedTime)
-   **Tasks / Deliverables** — work items per campaign.
    -   `Title` (singleLineText, primary)
    -   `Campaign` (multipleRecordLinks → Campaigns)
    -   `Status` (singleSelect: To do, In progress, In review, Done, Blocked)
    -   `Assignee` (singleCollaborator)
    -   `Due date` (date)
    -   `Notes` (multilineText)
-   **Assets** (optional third table) — when there's enough creative work to warrant separating asset versions from campaigns.
    -   `Name` (singleLineText, primary)
    -   `Type` (singleSelect: Image, Video, Copy, HTML, Print, Other)
    -   `Campaign` (multipleRecordLinks → Campaigns)
    -   `Status` (singleSelect: Draft, In review, Approved, Live)
    -   `File` (multipleAttachments)

### Variants

-   **B2B variant** — usually skip; if the team explicitly tracks per-account campaigns, push up to small / mid.
-   **Consumer variant** — base shape works as-is; add `Channel source` (singleSelect: iOS / Android / Web / Retail / Other) on Campaigns when relevant.
-   **Agency variant** — add `Client` (singleSelect or multipleRecordLinks → Clients table if more than ~5 clients). For per-client confidentiality, use per-client bases instead of one base with `Client` field.

### Views and interfaces to hand off

-   Calendar view on Campaigns keyed by Start date or End date — the most universal hand-off.
-   Form view on Campaigns or Tasks for "marketing request intake" — even at this scale, intake forms are high-leverage.
-   Filtered grid view: "Active this week" using a formula `IF(AND(Start <= TODAY(), End >= TODAY()), 1, 0)`.
-   Single Interface page summarizing campaigns by status — leadership-friendly read-only view.

## Solo / small (3-4 tables)

The default starter when the user wants more structure than a lightweight calendar — Campaigns + Briefs + Performance + (optional) Channels. Covers the dominant small-team needs: what we're running (Campaigns), what we asked creative to build (Briefs), what shipped well (Performance).

### Tables

-   **Campaigns** — initiatives across channels.
    -   `Name` (singleLineText, primary)
    -   `Status` (singleSelect: Now, Next, Later, Live, Done, On hold)
    -   `Goal / objective` (multilineText)
    -   `Channel` (multipleSelects: Email, Social, Web, Paid, SMS, PR, Event, Other)
    -   `Owner` (singleCollaborator)
    -   `Start date` / `End date` (date)
    -   `Budget` (currency)
    -   `Linked briefs` (multipleRecordLinks → Briefs)
    -   `Linked performance` (multipleRecordLinks → Performance)
    -   `UTM campaign` (formula — auto-generates a slug from Name)
-   **Briefs** — creative briefs feeding the production pipeline.
    -   `Title` (singleLineText, primary)
    -   `Campaign` (multipleRecordLinks → Campaigns)
    -   `Type` (singleSelect: Image / Video / Copy / HTML / Print / Other)
    -   `Audience` (multipleSelects or multipleRecordLinks → Personas / Cohorts)
    -   `Brief body` (multilineText)
    -   `Status` (singleSelect: Draft, In review, Approved, In production, Final, Live)
    -   `Owner / requester` (singleCollaborator)
    -   `Designer / copy` (singleCollaborator)
    -   `Due date` (date)
    -   `Assets` (multipleAttachments)
-   **Performance** — measurement / attribution per campaign + channel.
    -   `Name` (singleLineText, primary) — usually `[Campaign name] - [Channel] - [Period]`
    -   `Campaign` (multipleRecordLinks → Campaigns)
    -   `Channel` (singleSelect)
    -   `Date` (date)
    -   `Impressions` / `Clicks` / `Conversions` / `Revenue` / `Spend` (number / currency)
    -   `ROAS` (formula = `Revenue / Spend`)
    -   `Notes` (multilineText)
-   **Channels** (optional fourth table) — when channel-level KPI baselines matter.
    -   `Name` (singleLineText, primary)
    -   `Owner` (singleCollaborator)
    -   `KPI baseline` (number or text)
    -   `Integration metadata` (singleLineText) — UTM medium prefix, link to MAP segment, etc.

### Variants

-   **B2B** — add an Accounts table (or sync from Salesforce). Add `Account` (multipleRecordLinks → Accounts) to Performance. Add an ARR rollup on Campaigns (`Total ARR of linked accounts`) for ARR-weighted prioritization.
-   **Consumer** — add a Cohorts / Segments table. Add `Cohort` (multipleRecordLinks → Cohorts) on Campaigns. Replace ARR rollup with `Audience volume` rollup.
-   **Mixed (B2B2C)** — both Accounts and Cohorts tables. Performance links to one or the other (or both); campaigns roll up volume AND weighted ARR.
-   **Agency** — add a Clients table central. Add `Client` (multipleRecordLinks → Clients) to Campaigns. Add retainer drawdown formula on Clients (`Hours used vs. hours retained per period`).

### Views and interfaces to hand off

-   Kanban on Campaigns grouped by Status (Now / Next / Later).
-   Form view on Briefs for "creative brief intake."
-   Calendar view on Campaigns keyed by Start date.
-   Interface page: "Marketing calendar overview" — read-only Campaigns table filtered to Live + Next, grouped by Channel.
-   Form view on a generic "Marketing request" table feeding Briefs or Campaigns depending on type.

## Mid (5-6 tables)

The 3-4 table shape plus assets, channels, and audience modeling. Approval workflows become first-class; stakeholder-specific interfaces are common. The dominant mid-market shape.

### Tables added on top of the 3-4 table shape

-   **Assets** — creative assets / variants, distinct from briefs. **This table IS the DAM** for moderate asset volumes — Airtable's Attachment field stores the file, with rich typed metadata around it. The native **Asset Review** feature supports pixel-perfect annotation directly on image / video attachments; **Proofing** adds versioning with side-by-side comparison and an annotation toolset across supported document formats. For current plan-tier gates, supported formats, and file-size limits, see `support.airtable.com`. Only push the user toward an external DAM (Bynder / Frontify / Adobe / Cloudinary) at very-high-volume enterprise scale or when they already have one.
    -   `Name` (singleLineText, primary)
    -   `Type` (singleSelect: Image / Video / Copy / HTML / Print / Other)
    -   `Brief` (multipleRecordLinks → Briefs)
    -   `Locale` (singleSelect) — only when localization matters
    -   `Channel` (multipleSelects)
    -   `Status` (singleSelect: Draft, In review, Approved, Live, Archived)
    -   `Brand-compliance status` (singleSelect: Pending, Approved, Rejected)
    -   `File` (multipleAttachments) — the asset itself. **Configure the attachment-field format to "Versions"** to enable Proofing (each newly uploaded file becomes the next version; supports side-by-side comparison and annotation directly on the attachment). Asset Review provides native pixel-perfect feedback on images / videos. Plan-tier and configuration specifics evolve — check `support.airtable.com` for current requirements before scaffolding.
    -   `Version` (number) — the explicit version number, useful for cross-referencing in Approval audit-trail records (Proofing tracks versions implicitly via the attachment field, but an explicit number simplifies downstream rollups)
    -   `Usage rights / license` (multilineText) — optional, common in regulated and brand-asset-library use cases
    -   `Approved-for-external-use` (checkbox) — surfaces in any partner / agency portal interface
-   **Channels** — execution channels with owners and KPI baselines.
    -   `Name` (singleLineText, primary)
    -   `Owner` (singleCollaborator)
    -   `KPI baseline` (currency or number)
    -   `Integration metadata` (singleLineText) — MAP segment, social handle, etc.
-   **Personas / Cohorts / Segments** — audience modeling (pick one based on B2B vs consumer).
    -   `Name` (singleLineText, primary)
    -   `Description` (multilineText)
    -   `Size` (number) — for consumer cohorts
    -   `ICP fit` (singleSelect) — for B2B personas
    -   `Linked campaigns` (multipleRecordLinks → Campaigns)
    -   `AI persona summary` (AI field) — synthesizes top concerns + watering-hole channels + recent campaign-engagement signal from linked Performance. Human marketer reviews before using in brief audience sections.
-   **Tasks** (optional sixth table) — when day-to-day execution needs its own table separate from Briefs (e.g., creative ops with designer queues).
    -   Title / Assignee / Status / Due date / linked to Brief or Campaign

### Variants

-   **B2B** — Personas table central; campaigns roll up `Total ARR impacted` via linked Performance → Accounts. Add `Funnel stage` (singleSelect: TOFU / MOFU / BOFU) on Campaigns.
-   **Consumer** — Cohorts table central; campaigns roll up `Audience volume` and `Sentiment distribution.` Add `Lifecycle stage` (singleSelect: New / Active / At-risk / Churned) on Cohorts.
-   **Mixed** — both Personas and Cohorts; campaigns can link to either.
-   **Agency** — Personas / Cohorts per client; Clients table central; SOWs and deliverables. Per-stage SLA timing.

### Views and interfaces to hand off

-   Calendar view on Campaigns keyed by Start date, color-coded by Channel.
-   Kanban on Briefs grouped by Status (the creative-ops board).
-   Interface page: "Marketing leadership view" — Campaigns rollup by Status × Channel with KPI summary.
-   Interface page: "Designer queue" — Tasks or Briefs filtered to current assignee, sorted by Due date.
-   Form view on a Marketing Request table feeding intake (conditional fields by request type).
-   Sync setup wizard — Slack notification on Brief status change, HubSpot / Marketo sync for campaign metadata.

## Large (canonical 7-8 tables)

The mid shape plus approvals, vendors / agencies, and explicit budget tracking. Stakeholder-specific interfaces multiply (Leadership / MOps / Designer / Agency / Legal). Approvals become an explicit table for audit-trail purposes.

### Tables added on top of the mid shape

-   **Approvals** — audit trail for brand / legal / compliance reviews.
    -   `Asset` or `Brief` (multipleRecordLinks)
    -   `Approver` (singleCollaborator)
    -   `Decision` (singleSelect: Approved, Rejected, Approved with conditions, Pending)
    -   `Decision date` (date)
    -   `Notes / conditions` (multilineText)
    -   `AI pre-flag` (AI field on the linked Asset / Brief) — reads asset content + linked Claim Library + Disclaimer Library and surfaces likely issues (unsupported claims, missing disclaimers, claim-locale mismatches). Human reviewer makes the final decision; the pre-flag accelerates by surfacing what to check first.
-   **Vendors / Agencies** — external partners producing work.
    -   `Name` (singleLineText, primary)
    -   `Type` (singleSelect: Creative agency / Production house / Influencer agency / DAM / MAP / Other)
    -   `Owner` (singleCollaborator — internal AM)
    -   `Active` (checkbox)
    -   `Contract end date` (date)
    -   `Linked briefs` / `Linked campaigns` (multipleRecordLinks)
-   **Budget** — marketing spend by line item.
    -   `Line item` (singleLineText, primary)
    -   `Quarter` (singleSelect: 2026.Q1, 2026.Q2, …)
    -   `Campaign` or `Channel` (multipleRecordLinks)
    -   `Planned amount` (currency)
    -   `Committed amount` (currency, often via PO linkage)
    -   `Actual amount` (currency)
    -   `Variance` (formula = `Actual - Planned`)
    -   `Vendor` (multipleRecordLinks → Vendors)
    -   `AI variance narrative` (AI field) — synthesizes the variance + linked POs / invoices + program owner notes into a narrative explanation with reallocation suggestions. MOps director reviews before sharing in QBR / CFO briefs.

### Variants

-   **B2B** — Accounts central; ARR rollups drive prioritization. Add a Sales Pipeline link if marketing supports specific deals.
-   **Consumer** — Cohorts central; add app-store-source / retail-source ingestion via sync.
-   **Mixed** — both tables coexist.
-   **Agency** — Clients table central with retainer drawdown; per-client interfaces for client review.

### Views and interfaces to hand off

-   Org-level campaign rollup interface — Campaigns by Brand × Quarter, filtered to Live + Next.
-   Vendor / agency capacity view — Vendors with active brief counts.
-   Budget interface — Budget by Quarter with variance highlights.
-   Approval queue interface — Approvals filtered by Approver = current user.
-   Cross-base sync configuration — hand off the sync wizard for Salesforce / HubSpot / Workfront / etc.

## Enterprise / multi-brand portfolio

The large shape plus sub-brand tables, multi-region rollups, PO tracking integrated with finance, and compliance gates. Hub-and-spoke architecture: each brand / region has its own base syncing into a master campaign hub. Capex / opex on Initiatives; multi-currency rollups; locale-aware Approvals.

### Tables added on top of the large shape

-   **Sub-brands** — each brand in the portfolio (when applicable).
    -   `Brand name` (singleLineText, primary)
    -   `Region` (singleSelect or multipleSelects)
    -   `Owner` (singleCollaborator — brand lead)
    -   `Campaign hub base ID` (URL or text) — link to that brand's base if separate
    -   `Quarterly campaign volume` (rollup or count)
-   **Regions / Locales** — multi-market metadata.
    -   `Locale code` (singleLineText, primary) — e.g. `en-US`, `fr-FR`
    -   `Country / Region` (singleSelect)
    -   `Currency` (singleSelect)
    -   `Compliance regime` (multipleSelects) — e.g. GDPR, CCPA, alcohol-advertising rules
    -   `Local owner` (singleCollaborator)
-   **POs** — purchase orders for finance integration.
    -   `PO number` (singleLineText, primary)
    -   `Vendor` (multipleRecordLinks → Vendors)
    -   `Budget line` (multipleRecordLinks → Budget)
    -   `Amount` (currency)
    -   `Currency` (singleSelect)
    -   `Status` (singleSelect: Draft, Submitted, Approved, Invoiced, Paid, Closed)
    -   `Submitted date` / `Approved date` / `Paid date` (date)
-   **Compliance gates** (optional, regulated industries) — required reviews per phase.
    -   `Gate name` (singleLineText, primary) — e.g. `Legal review`, `MLR review`, `Brand compliance`
    -   `Phase` (singleSelect)
    -   `Required for` (multipleSelects: Asset type / Locale / Channel)
    -   `Approver role` (singleSelect)

### Campaign field additions at this tier

-   `Sub-brand` (multipleRecordLinks → Sub-brands)
-   `Locales` (multipleRecordLinks → Regions / Locales)
-   `Capex` / `Opex` (currency) — when business-case finance fields matter
-   `Expected revenue` / `Expected savings` (currency)
-   `ROI` (formula)
-   `Multi-currency rollup` (formula or rollup with currency conversion)

### Views and interfaces to hand off

-   Multi-brand portfolio rollup interface — Campaigns by Sub-brand × Quarter, filtered to Live.
-   Multi-region timeline interface — Campaigns by Locale × Channel.
-   PO reconciliation view — Budget vs POs vs Actual variance per Quarter.
-   Compliance audit interface — Approvals filtered by Compliance gate and Phase.
-   Cross-base sync — each Sub-brand base syncs Campaigns into the master hub.

## Regulated marketing (niche — surface on demand)

For pharma, alcohol, finance, insurance, healthcare, lottery — where assets and campaigns go through phased compliance gates with required approvals. Triggered when the user uses _"MLR review,"_ _"compliance gate,"_ _"legal sign-off,"_ _"claim validation,"_ _"audit trail,"_ or regulated-industry signals.

### Tables added on top of the large or enterprise shape

-   **Claim library** — approved claims that assets can reference.
    -   `Claim` (multilineText, primary)
    -   `Approved by` (singleCollaborator)
    -   `Approved date` (date)
    -   `Expiration date` (date)
    -   `Reference source` (singleLineText)
    -   `Locales` (multipleRecordLinks → Regions / Locales)
    -   `Status` (singleSelect: Active, Expired, Withdrawn)
-   **Disclaimer library** — required regulatory disclaimers per locale / product / channel.
    -   `Disclaimer text` (multilineText, primary)
    -   `Required for` (multipleSelects: Locale / Channel / Product / Audience)
    -   `Reference regulation` (singleLineText) — e.g. _"FTC 16 CFR Part 255"_, _"FDA OPDP,"_ _"AGCO."_
    -   `Active` (checkbox)
-   **MLR / Compliance reviews** — Medical / Legal / Regulatory review records.
    -   `Asset` (multipleRecordLinks → Assets)
    -   `Reviewer role` (singleSelect: Medical / Legal / Regulatory / Brand)
    -   `Reviewer` (singleCollaborator)
    -   `Decision` (singleSelect)
    -   `Notes` (multilineText)
    -   `Cycle number` (number) — for multi-round reviews
    -   `AI compliance pre-flag` (AI field on linked Asset) — same pattern as Approvals: reads asset content + Claim Library + Disclaimer Library + Locale and flags likely issues for human reviewer attention. Heaviest leverage in MLR cycles where reviewers are the explicit bottleneck.

### Asset / Brief field additions

-   `Required claims` (multipleRecordLinks → Claim library)
-   `Required disclaimers` (multipleRecordLinks → Disclaimer library)
-   `Compliance status` (formula or rollup over linked MLR reviews)
-   `Locale-specific compliance status` (rollup)

### Views and interfaces to hand off

-   Compliance review queue interface — Assets at each phase, sortable by Due date.
-   Claim library audit — Claims with expiration warnings.
-   Approval audit log — MLR reviews filtered by Phase or Reviewer.
-   Disclaimer enforcement view — Assets missing required disclaimers per locale.

## Agency multi-client (niche — surface on demand)

For agencies, freelancers, and consultancies running marketing for multiple clients. The dominant SMB shape and a meaningful Enterprise shape (in-house agencies). Triggered when the user uses _"clients,"_ _"multi-client,"_ _"retainer,"_ _"agency,"_ _"client portal,"_ or runs marketing for external orgs.

### Schema-choice decision

-   **Single base with `Client` field** — most common; easier to manage; cross-client reporting and capacity rollups are simple. Use when client confidentiality is moderate (clients don't see each other but the agency team can).
-   **Per-client base** — required when client confidentiality is strict (e.g., NDA-bound brands in the same category — two competing apparel makers, two competing auto dealers). Heavier to maintain; cross-client capacity reporting requires sync into a central agency hub.

### Tables added on top of the small or mid shape

-   **Clients** — each client org the agency serves.
    -   `Client name` (singleLineText, primary)
    -   `Account owner` (singleCollaborator — internal AM)
    -   `Tier` (singleSelect: Retainer / Project / On-demand)
    -   `Active` (checkbox)
    -   `Industry` (singleSelect)
    -   `Onboarded date` (date)
    -   `SOW links` (multipleAttachments or URL)
    -   `Retainer hours per period` (number)
    -   `Period` (singleSelect: Monthly / Quarterly / Annual)
    -   `Hours used this period` (rollup from Tasks)
    -   `Hours remaining` (formula)
-   **SOWs / Engagements** (optional) — when SOWs change frequently.
    -   `Name` (singleLineText, primary)
    -   `Client` (multipleRecordLinks → Clients)
    -   `Start date` / `End date` (date)
    -   `Deliverables` (multilineText or multipleRecordLinks)
    -   `Hours estimate` / `Hours actual` (number)
    -   `Status` (singleSelect: Draft / Signed / In progress / Closed)

### Campaign / Brief field additions

-   `Client` (multipleRecordLinks → Clients) on Campaigns, Briefs, Tasks
-   `SLA stage` (singleSelect) — first concept / pre-production / final / delivered
-   `Hours actual` (number) on Tasks for retainer drawdown

### Views and interfaces to hand off

-   Client portal interface (one per client OR one with `current user → their client` filter) — read-only view of their campaigns + approval queue.
-   Retainer drawdown view — Clients with hours used vs retained, color-coded.
-   Account-manager dashboard — Clients owned by current user with current campaign status.
-   New-client onboarding form — captures client metadata and creates Campaigns table records.

## Lightweight marketing CRM (niche — surface on demand)

**Airtable can BE the lightweight marketing CRM** for moderate contact volumes when the user has no existing Salesforce / HubSpot CRM, or for marketing-only contact tracking sitting in front of an existing sales-side CRM. Typed contact fields + segments + linked-record account hierarchy + automations cover the marketing-side job. Surface this shape when the user uses _"contact tracker,"_ _"prospect list,"_ _"marketing CRM,"_ _"lifecycle marketing without Salesforce,"_ _"warm leads list,"_ or describes a contact-management need without an existing CRM. Don't push specialized vertical CRMs unless contact volume exceeds what Airtable's relational model handles cleanly (typically tens-of-thousands of contacts plus heavy query workload) or the user explicitly asks.

### Tables added on top of the small or mid shape

-   **Contacts** — the central record.
    -   `Name` (singleLineText, primary)
    -   `Email` (email)
    -   `Phone` (phone)
    -   `Company` (multipleRecordLinks → Accounts, or singleLineText for unmapped)
    -   `Title` (singleLineText)
    -   `Lifecycle stage` (singleSelect: Lead / MQL / SQL / Customer / Lost / Dormant) — adapt naming to the user's funnel
    -   `Source` (singleSelect: Inbound form / Event / Webinar / Outbound / Referral / Other)
    -   `Owner` (singleCollaborator) — assigned marketer / SDR
    -   `Segments` (multipleRecordLinks → Segments) — for campaign targeting
    -   `Linked campaigns` (multipleRecordLinks → Campaigns) — historical engagement
    -   `Engagement score` (formula or AI field) — rolled up from event / campaign / email touches
    -   `Created` (createdTime), `Last touched` (lastModifiedTime)
    -   `Notes` (multilineText) — free-form rep notes
    -   `AI lifecycle suggestion` (AI field) — recommends a lifecycle stage transition based on recent activity; human marketer approves before transition.
-   **Segments** — saved audience definitions.
    -   `Name` (singleLineText, primary)
    -   `Criteria` (multilineText) — the segmentation logic in plain English
    -   `Linked contacts` (multipleRecordLinks → Contacts) — manually curated OR maintained via automation
    -   `Owner` (singleCollaborator)
    -   `Linked campaigns` (multipleRecordLinks → Campaigns)
-   **Accounts** (optional — when B2B) — the company-level record contacts roll up to.
    -   `Account name` (singleLineText, primary)
    -   `Industry` (singleSelect)
    -   `Size` (singleSelect: 1-10 / 10-50 / 50-200 / 200-1000 / 1000+)
    -   `Linked contacts` (multipleRecordLinks → Contacts)
    -   `Account owner` (singleCollaborator)

### When to push toward a dedicated CRM instead

-   Contact volume above ~10-50K records (Airtable's relational model slows on heavy query workload at that scale; the dedicated CRM's indexes matter).
-   The user already has Salesforce / HubSpot CRM — sync marketing-ops contacts to it; don't build a parallel layer.
-   Sales already lives in a CRM — marketing needs to play nice with their pipeline data; sync rather than fork.
-   Heavy automation needs around contact state transitions (lifecycle, lead scoring, lifecycle workflows) — MAPs / CRMs do this with battle-tested infra; Airtable's automations cover lighter needs.

### Views and interfaces to hand off

-   Lifecycle Kanban view on Contacts grouped by Lifecycle stage (the lead-to-customer pipeline view).
-   Gallery view on Contacts cover-image = company logo or contact photo (for relationship-mapping views).
-   Form view on a "Lead submission" or "Event sign-up" intake.
-   Interface page: Account 360 — Account with linked Contacts, linked Campaigns, recent engagement timeline.
-   Sync into the user's MAP for actual email sends; Airtable holds the audience definition, MAP holds the send infrastructure.

## Choosing between shapes

If the answers to the scope questions don't obviously map to one shape, lean smaller — it's easier to add tables than to strip them. The MCP can extend the schema cleanly as the team grows; over-scaffolding a 10-table base for a single marketer creates clutter and abandoned views.

When in doubt:

-   Default to **lightweight (2-3 table)** for solo marketers, music releases, book publicity, very small agencies.
-   Default to **small (3-4 table)** for in-house teams under 10 with basic campaign tracking needs.
-   Default to **mid (5-6 table)** for 10-50 person marketing orgs with creative ops + audience modeling.
-   Default to **large (7-8 table)** for 50+ person marketing orgs with multi-channel + approval + budget needs.
-   Default to **enterprise / multi-brand** only when the user has multiple sub-brands or multi-region complexity.
-   Surface **regulated** only when the user uses MLR / compliance / claim-validation vocabulary or operates in alcohol / pharma / finance / insurance / lottery / healthcare.
-   Surface **agency multi-client** only when the user explicitly serves external clients OR runs an in-house agency.
-   Surface **lightweight marketing CRM** when the user describes contact / prospect / lifecycle tracking without naming an existing CRM, or explicitly says they want marketing CRM functionality but don't have Salesforce / HubSpot.

The user can always ask for more tables; pushing all 10 on a 5-person team is overcorrection.
