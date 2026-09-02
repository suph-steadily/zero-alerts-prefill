# Brent's CA RCE email: what the digging found (2026-09-01)

Skim version. Full research memos are in research/ (B = prior pre-fill findings, C = history, D = model and rules, A = the Calabasas quote lookup).

## The email
- From Brent Denton (AVP Agency Distribution) 9/1 7:47pm CT to David Curry, Suph, Bryan Hartwig. Forward of Marc Brune (ASM) forwarding Michael Ehrmann, Goosehead agency owner, Calabasas CA.
- Ehrmann: our RCEs average ~$250/sqft in LA and Ventura, about half of real rebuild cost. Example: 2,789 sq ft home, RCE ~$674,000 = $241/sqft. The 20% Coverage A bump isn't enough. UW won't consider other carriers' RCEs. He moves business elsewhere because of it.
- Marc: "too low consistently and seems to be costing us some biz. Will get comps."
- Brent's question: will our own model follow exactly how v360 works, or can we make adjustments?
- Brent was also on Marc's 8/18 SF four-plex thread, so he has seen the inputs example already.

## The Calabasas quote
- See section "A" below once the Metabase lookup lands.

## Model vs inputs (research/D-model-and-rules.md)
- Our own model (Plinko, velma repo) is built to reproduce 360Value. Version notes call it a "v360 emulator." Docs open with "Villa's job is to reproduce V360/Verisk as closely as our data allows." Validation = agreement with 360Value (median -0.12%, 88% within 20%). Switching it on alone will not raise LA.
- We own the levers 360Value never exposed: location factors, quality tiers, 11 rating curves, state calibration (NV and AZ overrides exist, no CA), cost bounds, training target. A CA adjustment is a config/retrain decision for UW, actuarial, and David's team, not a Verisk ask. Nobody has asked for it.
- In our own book, 360Value's median for LA County homes of 2,500 to 3,000 sq ft is $255/sqft (850K rows). The agent's $241 is 5% under that. It is a typical 360Value LA number, not an outlier.
- David Curry's Dec 12, 2025 report flagged "California Geographic Divergence ... I suspect Plinko is more correct here" (Bay Area V360 $246/sqft vs Plinko $400; SoCal $271 vs $280). Later versions were tuned back toward V360 (county floors that pushed above V360 in expensive markets were turned off).
- CA is in no rollout phase. Every plan: AZ (+NV), direct then inde, hold a month, "expansion batches TBD."
- Quality grade is the largest single lever in the model: Standard to Above Average +14%, Custom +38%, Premium +79%. UW's 360Value guide names sqft, roof material, and quality grade the "Big Three." Custom/Premium triggers a UW alert asking for plans, permits, invoices, or appraisal.
- The 20% rule is code: Coverage A must be 100% to 120% of RCE for every inde quote in every state, since June 2023. A 90% to 150% band shipped July 2026 (GLD-1538) but only for the direct self-serve funnel, DP3. UW can raise the ceiling by entering an RCE override (then up to 120% of the override). No written UW policy on accepting another carrier's RCE, appraisal, or contractor bid. The written UW method is to fix the inputs.
- CA complaints are not new. BUC-2803 (May-June 2025): Jenna, Verisk numbers "well under the $300/SF mark ... California much higher"; Datha, "noise and frustration from Inde agents ... mostly in CA"; Emily Schield, Safeco requires at least $350/sqft in CA. Outcome: $/sqft soft block raised $300 to $500; Curry's first project became the Steadily model.
- Counter-evidence: Bryan Hartwig, 7/15 #tmp-goosehead-deductibles: we quote higher Cov A than the market binds nationally; Goosehead's rater defaults Cov A on its own $/sqft assumption. Ron: "regardless of what the agent enters our RCE overrides it." The agent's "half" benchmark may be Goosehead's CA default rather than other carriers' RCEs. Marc's comps will settle it.

## How wrong pre-fill is (research/B-prefill-factsheet.md, section 1)
- 28.2% of dwelling-age (101+) approvals include the UW fixing one of our six pre-filled fields (Will's BUC-4962 census, 10,428 quotes Jan 15 to Jul 22). Verified.
- UW edits per field on approvals: 2.6% to 8.4%. 65% to 79% of that volume lands on pre-fill the agent never touched. Verified.
- Agent-modified values get corrected 1.4x to 5.7x MORE than untouched pre-fill (sqft 25.3% vs 4.5%). When UW overrides an agent edit, 58% of the time it lands on a third value. Verified.
- Sqft skews BIG: 58% of UW sqft fixes go DOWN, median 357 sq ft / 17.5%. Year built skews NEW: 71% of fixes make the home older, median 10 years. Verified. So "pre-fill makes RCE low" is true for missing kitchens/bedrooms/bathrooms/garage/basement/finishes, not for square footage.
- Hard defaults when 360Value errors: sqft 2,300, year built 1980. 25.5% of year-built "edits" on new builds are agents replacing the 1980 placeholder (~960/mo). Verified.
- Smarty misses bedrooms OR bathrooms ~35% of the time; kitchen does not exist in Smarty (default 1). Joseph, 8/31. Verified.
- Curry's 9/1 fill-rate table (BUC-5271): dropping 360Value loses bedrooms 17.3%, exterior wall 12.7%, stories 9.0%, bathrooms 8.4%, basement 8.1% of quotes' values that Smarty cannot replace.
- UW perception: "pre-fill is wrong more often than not" (DISCO-746); LaNae 8/18: "We got so used to it being wrong, we stopped even bringing it up."

## What it causes (research/B, section 2)
- RCE too low. SF four-plex SP3-CA-33450708-00 (agent Will Kouvaris via Marc, 8/17-8/18): same 360Value version, same Above Average grade; we pre-filled 2 kitchens, 6 medium bedrooms, 90% carpet, 1 heating system, no garage; competitors had 4 kitchens, 8 large bedrooms, hardwood and tile, 2 heating systems, 2-car attached garage. $993k vs $1,393k, 29% lower on inputs alone. Datha's KS home (8/31): pre-fill said Concrete Slab, Smarty raw data had a 1,804 sq ft basement.
- Underwriting time. Dwelling age is the highest-volume alert: ~5,044 UW touches/mo, ~2,200 forced reviews/mo. UWs run LandGlide, then Zillow/Google, then county records to fix our data. Nothing is learned: the value overwrites, the source is thrown away.
- Premium. UW six-field corrections move premium up 3.4x as often as down, ~$699k net Jan-Jul across all approvals (78% never bind); bound-only ~$26K/mo (~$310K/yr). Unreviewed old homes forgo $3,034 per 100 bound, ~0% recovered at renewal. The "+$800K/yr" figure has no written reconciliation.
- Customer moments. Errors caught at inspection mean policy reformation back to inception with premium back-charged. Lost-looking CA deals via the same ASM twice in three weeks.
- Post-bind: pre-fill errors do NOT cause NOCs (2.4% of cancellation reasons are data discrepancies, zero for sqft). They show up as RCE and premium. That is why the pre-fill workstream is ranked last in Will's alert-work order.
- New builds: BUC-5239 Cape roof-score override; the 1980 fallback lane is the pre-fill angle.

## Three years and ownership (research/C-prefill-history.md)
- "Three years" is supported and conservative. Pre-fill exists since Aug 2020 (Estated, landlordweb). Gap-fixing starts 2022 (SWA-3012 Smarty vs Estated, Won't Fix; SWA-3792 fallback-rate spike). BUC-3 "improve SSB auto-populated property field data" opened 2023-08-04 and is still To Do.
- Chain: default guardrails (2024), attribution stamps (late 2024), Estated swapped for Smarty on FILL RATE (DATA-141 Oct 2024: 95% vs 85%; cutover Feb 2025), DISCO-350 classifier (Apr 2025), BUC-3622 360Value cross-comparison (Oct 2025), in-house RCE (Oct 2025 on), ReportAll call (Dec 2025, stalled), Estated deleted (Jan 2026), DISCO-740/746/756 + BUC-5223 (Aug 2026).
- Nobody has ever tested a source for accuracy. DATA-141 scored fill rate only.
- Ownership: Curry 8/18, Pluribus long-term mandate, state expansion short term. Christine's Aug 17 to Sep 26 plan has no pre-fill line. Buc-ee's owns the plumbing de facto. BUC-5223 is unassigned.

## Already in motion (research/B, section 4)
- BUC-5223 epic + pre-fill PRD (ask UW where the value came from; test providers per field). Unassigned.
- Bake-off brief (artifact f743c2ea) punted to next cycle 8/26; vendor tests last in Will's order.
- Source-capture Storybook prototype built 8/24 (artifact c40359f2).
- Curry's BUC-5271 fill-rate table Done 9/1; valuation impact per field is agenda item 2 for the 9/10 Datha call (Thu Sep 10, 3:30 CDT; Datha, Curry, Rob accepted).
- Pre-ship fixes in flight: carlos #1342 (ignore v360 details when not wanted), velma #131 (user-entered rooms), Joseph capping outliers and adding bathroom prefill.

## Cautions for the email
- The Calabasas case is not proven to be inputs. It reads as the 360Value LA level. Say "may be more model than inputs," not "pre-fill."
- Don't claim sqft is under-filled. It skews big.
- Use 28%, not "a third" (the third was a meeting statement, different cohort).
- Pick one premium figure and name the denominator. The email uses the 3.4:1 ratio only.
- Don't restate the 8/31 Datha diagnosis (it was corrected 9/1).
