# Pre-fill at Steadily: how long has the team been trying to fix it, and who owns it now

Prepared 2026-09-01 for Suph. Read-only research across Jira, Confluence, Slack, Gmail, Google Drive, local repo clones (landlordweb, ralphie, frizzle, ryerson) and GitHub (landlordhq/carlos and org-wide commit search). Nothing was created, edited, commented or sent anywhere.

"Pre-fill" here = auto-populating dwelling attributes (sqft, year built, property type, construction type, roof, bedrooms/baths/kitchens, garage, basement) on a quote from third-party property data.

---

## 1. Verdict

**"Three years" is supported, and it is conservative. The safer claim is "at least three years", and "since 2022" or "since the MGA launched" are both defensible.**

- Pre-fill itself is six years old. The first Estated lookup that populated the funnel landed in landlordweb on **2020-08-31** (Darren Nix, commits "Add EstatedCache model" and "Make Estated API production work"), two days after the repo's initial commit. The first Jira ticket literally titled "Prefill property data" is **SWA-43, 2021-02-08** (David Tulig): verify address with SmartyStreets, look the property up in Estated, push into Salesforce.
- The team started measuring and trying to fix its gaps almost four years ago:
  - **2022-08-29** SWA-3012 "Analyze Smarty vs Estated for address failures" (Feni Varughese; closed Won't Fix).
  - **2022-11-04** SWA-3792 "Spike: Validate how often we use our fallback data for value 360 lookups" (Sofie Pompa to Rob Soule): "Year built and square feet are defaulted to 1980 and 2300 ... How often are we relying on these values?"
- The cleanest "three years ago" anchor is **August 2023**: SWA-7457 (2023-08-02, Steven Balough) audited how often SSB got no sqft / year built from Estated, and **BUC-3 (2023-08-04) "improve SSB auto-populated property field data"** found "5-10% of the time we get no data and even more often 15-55% we get data, but important fields are missing." BUC-3 is still open (To Do, unassigned) three years later.
- Leadership has said the same thing in writing. David Tulig's Feb 2024 post-mortem: the 1980/2300 fallback "has existed since the MGA's existence" and "we've always fallen back to" it. Christine Luo's Apr 2025 DISCO-350: "Agents/customers are either not entirely truthful or not paying attention when we're asking them to update data we've prefilled."
- The chain of attempts since then is unbroken: guardrails on defaults (2024), source tagging / attribution (late 2024), Estated replaced by Smarty enrichment because of fill rate (Nov 2024 to Feb 2025), an ML classifier on existing vendor data (Apr 2025), a 360Value cross-comparison (Oct 2025), in-house RCE to drop 360Value (Oct 2025 onward), a ReportAll sales call (Dec 2025, stalled), Estated code deleted (Jan 2026), and now the bake-off plus provenance capture (Aug 2026).

One nuance worth knowing before you write the sentence: **nobody has ever tested a pre-fill vendor for accuracy.** The one vendor evaluation on record (DATA-141, Oct 2024) scored fill rate (Smarty 95% vs Estated 85%), not whether the values were right. The accuracy work that exists is partial: BUC-1826 (Sep 2024) noticed Estated year built vs self-reported often differed by more than 5 years; BUC-3622 (Oct 2025) diffed raw 360Value values against what we plug in; and your own Aug 2026 census measured UW corrections. So "trying to solve for three years" is true, but the attempts were mostly about coverage and guardrails, not about picking the most accurate source.

### Phrasings you can defend

- "This is a problem the team has been chipping away at for at least three years. The first 'improve our auto-populated property data' ticket is from August 2023 and is still open, and the fallback-defaults problem was being measured in late 2022."
- "We have been pre-filling dwelling data since 2020 and patching its gaps since 2022: default guardrails, source tagging, swapping Estated for Smarty, an in-house RCE. What we have never done is test a vendor for accuracy against what underwriters actually correct."
- Shortest safe version: "a problem the team has worked on, in one form or another, since 2022."

Avoid: "three years" stated as an exact start date (it is a floor, not the start), and "we have never looked at this" (they have, repeatedly; the gap is accuracy testing, not attention).

---

## 2. How the pipeline evolved (three eras)

| Era | Pipeline | Evidence |
| --- | --- | --- |
| 2020 to 2021 (landlordweb monolith) | Funnel address -> SmartyStreets normalize (Oct 2020) -> Estated lookup cached in EstatedCache -> pre-populate funnel / Salesforce. 360Value added Jul 2021 for RCE, seeded with sqft and year built. | landlordweb commits 2020-08-31, 2020-10-09, 2021-07-07, 2021-07-26; SWA-43 |
| Late 2021 to Feb 2025 (carlos, "Property data lookup") | carlos created 2021-09-30; Smarty for all address lookups (Dec 2021); 360Value moved in (SWA-870, Jan 2022); Estated in carlos by Jan 2022. Estated supplies sqft/year built/etc., 360Value overrides. Hard defaults 1980 / 2300 when Estated misses. | carlos commits; Confluence "Data Attribution" (Jan 2025): "Carlos gathers address information by calling Estated and Value360's APIs ... value360 will override the values from Estated." |
| Feb 2025 to now | Smarty US Address Enrichment replaces Estated as the seed (parallel run then cut over; flag removed 2025-05-14; Estated code deleted 2026-01-23). Per-field attribution stamps (ESTATED / SMARTY_ENRICHED_ADDRESS / VALUE_360 / USER / AI_WORKFLOW / STEADILY). 360Value still produces RCE and property details; Steadily RCE (Villa/Plinko) replacing it, AZ first (BUC-5155). | BUC-2055, BUC-2238, PR #426, BUC-4235, Goodbye 360Value doc, BUC-5155 |

---

## 3. Dated timeline (date | what | who | source)

2020-08-29 | landlordweb repo initial commit | Darren Nix | ~/landlordweb git log
2020-08-31 | "Add EstatedCache model" + "Make Estated API production work": funnel basics page pre-populated from Estated | Darren Nix | landlordweb 896ca4bea, 33ff8b9e5
2020-10-04 | "feat: prepopulate a lot of fields" in the funnel | Darren Nix | landlordweb 2744ee836
2020-10-09 | SmartyStreets address normalization added | Darren Nix | landlordweb 7c4efacab
2020-11-21 | Property info sheet shows Estated cache data to staff | Allen Chang | landlordweb 2b2f3efd5
2020-12-19 | "populate form from estated cache" refactor + tests (PR #51) | Darren Nix | landlordweb 476814fb
2021-01-18 | SWA-10 "Add property_type question"; property type looked up from Estated cache | Darren Nix -> David Tulig | SWA-10; landlordweb 4b21d3ba
2021-01-25 | SWA-20 "Try to guess answers on the lead property object after each funnel step"; SWA-33 map Estated flat roof to shape/pitch | David Tulig, Darren Nix | Jira
2021-02-08 | SWA-43 "Prefill property data using the address from a Salesforce Opportunity" (Smarty verify -> Estated -> Salesforce). Done 2021-02-10 | David Tulig -> Allen Chang | SWA-43; landlordweb PR #177
2021-02-10 | DISCO-98 "Design a schema with canonical addresses and layers of data verification": Estated cache vs customer vs sales verification drift out of sync. Still in Parking lot | David Tulig | DISCO-98
2021-04-20 | SWA-101 "improve property data accuracy for multi-property funnel" | Allen Chang | Jira
2021-05-04 | "only fill in smarty and estated if we need to" | Cameron Davison | landlordweb 31df84e01
2021-07-01 | Verisk 360Value integration guides (UAT) saved to Drive | (Drive) | folder 1pvg5_rzSHEeOS76n1w3KcjaBP2vdlMWO
2021-07-07 | "feat: add value 360 API"; 07-26 sqft + year built passed to 360Value RC calc; SWA-189 agents can modify 360Value inputs | Allen Chang | landlordweb #573, #600; SWA-189
2021-09-22 | SWA-459 "prefill - value360 & estated information on a property and return both" (Won't Fix); SWA-475 skip Estated when enough info; SWA-493 rate-limit Estated | Allen Chang, David Tulig | Jira
2021-09-30 | carlos repo created, description "Property data lookup" | Cameron Davison | GitHub repos/landlordhq/carlos; first commit 7069f90d
2021-10-01 | ralphie created as "fork carlos" | Allen Chang | ~/ralphie d5f0ebe13
2021-10-06 | Cape Analytics client added to carlos (SWA-560) | Allen Chang | carlos 8ed75b30
2021-10-18 to 12-14 | Smarty into carlos: lat/lng, county, cached lookups (SWA-781), "use smarty for all lookups" | Cameron Davison | carlos a81620ca, 3bf3d771, 9f475b9c
2021-11-09 | SWA-740 "move Estated into Carlos" (dup); 12-02 SWA-870 "move 360value into Carlos" (commit 2022-01-11) | Allen Chang | Jira; carlos ed92a2cf
2021-12-17 | SWA-957 "remove the redundancy entering data twice: Dwelling page and 360Value plugin" | Allen Chang | Jira
2022-01-19 | carlos calls both Estated and 360Value ("Dont log auth tokens in Estated and 360Value") | | carlos 1cff832b
2022-06-03 | "360Value Analysis.docx" + Arch Re 360Value analysis: 264 bound locations, how far agents deviated from the 360Value number | (Drive, reinsurer-facing) | 1gzS4MZQK34B6Weu1TqKME7UgdwXIU3_q
2022-08-29 | SWA-3010 load carlos lookup results into a DB for analysis (Done); SWA-3011 track failed Smarty lookups (Done); SWA-3012 "Analyze Smarty vs Estated for address failures" (Won't Fix); SWA-3013/3014 Smarty vs Cape / GuyCarp (Won't Fix) | Feni Varughese | Jira
2022-09-16 | SWA-3211/3212/3213/3214: carlos returns Estated land value, units/bathrooms/bedrooms/stories, Estated-only flag | Cameron Davison | Jira; carlos PRs #116 to #119
2022-11-04 | SWA-3792 spike: how often do we fall back to 1980 / 2300 for 360Value? "We have a known gap on providing a strong RCE, Estated doesn't provide RCE." Done 11-10 | Sofie Pompa -> Rob Soule | SWA-3792
2023-05-10 | gdoc "Value360 and MGA product interactions": "Value360 doesn't seem super effective as a prefill tool (we have to provide basic info like year built and sq ft) so edits are often necessary" | (author not shown) | Drive 1CqVT6jqOG0xMrP2TB268eUeRW202dhrNr02Gd9uXiAM
2023-06-21 | SWA-6753 "sMGA: Use Webfunnel, else Estated Value for Dwelling Property Type" + SWA-6755 spike comparing Estated vs 360Value property type on 200 properties | Sofie Pompa -> David Chen | Jira; carlos a130bbe5
2023-06-28 | SWA-6868 "SPIKE: Webfunnel Property Type Data Accuracy" (To Do); SWA-6890 "Validate Estated Data for Dwelling Details" (Won't Fix) | Sofie Pompa | Jira
2023-08-02 | SWA-7457 audit: how often SSB does not auto-fill sqft / year built (monthly 2023 prod query on estated_cache) | Steven Balough | SWA-7457
2023-08-04 | BUC-3 "improve SSB auto-populated property field data": 5-10% no data, 15-55% missing fields; idea: "call a more expensive API only after estated has missing information". Still To Do, unassigned | Steven Balough | BUC-3
2024-01-29 | SWA-9611 UW block when both 1980 and 2300 defaults present | (Fortegra UW check) | Jira; ryerson 64bf9d6f
2024-02-20 | Post-mortem "UW rule for location defaults was too narrow": 77 more policies bound on defaults; "We've always fallen back to the year built 1980 and square footage of 2300"; "it's existed since the MGA's existence"; SWA-9769 fix; long-term item: other defaults (DOB, property type, building quality, condition) | David Tulig (author), Rob Soule, Kayla McDermott | Confluence ENG 1888387074
2024-02-21 | BUC-776 "Enforce user action if year built or sqft is set to default" (Done 2024-05-07) | Kayla McDermott -> Tim Robertson | BUC-776
2024-05-21 | SWA-10402 "Don't run estated for estimates if enough information is provided" | David Tulig | Jira; ralphie 51fbf5936
2024-08-01 | BUC-1488 SSB prompts for input when 1980 / 2300 defaults | Rob Soule -> Francisco Martinez | Jira
2024-09-24 | BUC-1826 "Year Built Discrepancy Flag": Estated vs self-reported year built often off by more than 5 years, driving UW cancellations (Done 2025-06-19) | John Blair (cc Darren Nix, Jeff Greco) | BUC-1826
2024-10-08 | BUC-1829 document defaults and the override path (estated > v360; ralphie > carlos) | Rob Soule -> Phillip Bannister | BUC-1829
2024-10-15 | Epic BUC-1856 "Improving data default and Value360 handling (NAUI)": property_details_attribution (Done 2024-12-10) | Kayla McDermott -> Tim Robertson | BUC-1856
2024-10-18 | DISCO-174 "Generic data / default override and experience" (Done): "These tools include CAPE Analytics, Estated, and Value360. In some cases these tools do not return values, which results in us plugging in a default value. If the agent doesn't see or update these values, their policy may be canceled due to inaccurate information." Goals: flag values needing attention, edit without opening Value360, attribution. Links a PRD, an ADR by Tim, a "Generic data audit" by Phil, and Figma | Kayla McDermott | DISCO-174
2024-10-23 | DATA-141 "Can Smarty replace or supplement Estated?": "85% success rate in getting all the 3 critical data fields from Estated. That's good but not great." Done 11-10 | Darren Nix -> John Blair | DATA-141
2024-11-10 | Epic BUC-2055 "Replace Estated property data lookup with Smarty": "Smarty has a 95% fill rate vs 85% for Estated"; run both a week, then shut Estated off (Done 2025-03-01) | Darren Nix -> Phillip Bannister | BUC-2055
2024-11-27 | carlos PR #340 (BUC-1904) merged: data attribution, "Where did the data come from, Steadily default or Estated lookup"; ralphie BUC-1903 / 2061 / 2062 attribution plumbing | Phillip Bannister, Tim Robertson, Feni Varughese | GitHub; ralphie git log
2024-12-03 to 2025-01-13 | Defaults incident: carlos deploy sent 1980 / 2300 to 360Value while the UI showed correct values; 224 quotes, 10 issued; TECHSUP-918; action items BUC-2263, BUC-2264, "Simplify Carlos" | Rob Soule (author), Tim Robertson, Phillip Bannister, Kayla McDermott | Confluence ENG 2408841217
2025-01-06 | BUC-2238 Smarty Address Enrichment alongside Estated; BUC-2237 LLW calls carlos instead of Estated; BUC-2240 "Breakdown for Smarty/Estated" | Rob Soule -> Phillip Bannister | Jira
2025-01-09 | Zesty integration in carlos (COS-55) | | carlos 81433aa0
2025-01-27 | Confluence "Data Attribution": defaults must be confirmed before issuance; "Our next goal is to supplement the initial data set with a Smarty lookup in order to reduce the frequency that we use default values." Child page "Estated": year_built 1980, total_sq_feet 2300 | Phillip Bannister | Confluence ENG 2439380993, 2449244173
2025-02-06 | BUC-2376 Metabase Smarty fields for hit analysis; DISCO-321 "Replace Estated property data lookup with Smarty" (Done) | Rob Soule; Kayla McDermott | Jira
2025-02-19 | carlos PR #426 "Buc 2238 smarty estated enrichment" merged | Phillip Bannister | GitHub
2025-04-16 | DISCO-350 "Use existing data to write property attribute classifier": 1 in 5 policies gets a UW action; use Cape, Smarty, Smarty Enriched, Verisk, RedZone, Guy Carpenter, Google Elevation, Zesty (Estated struck: "planning to stop running this in favor of smarty enriched"). Done; RBT-3475 | Christine Luo; Joseph Barratt | DISCO-350, RBT-3475
2025-05-02 | BUC-2804 Smarty enriched on the estimate stage too | Rob Soule | Jira; carlos 93dae0e9
2025-05-14 | Smarty enriched feature flag removed (always on); BUC-2873 "remove dead estated path lookup in LLW" | Rob Soule | carlos bd579885; BUC-2873
2025-06-25 | BUC-3125 SSB stops pre-filling year built / building size when the lead only has defaults | Francisco Martinez | Jira
2025-08-22 | BUC-3415 "get all the Smarty enriched data" | | carlos a10b57a0
2025-09-25 | BUC-3622 "Value360 cross comparison effort": ~500 addresses, raw 360Value values vs what we plug in, diff of where we're wrong (Done 10-09; comparison JSON zip in Drive 2025-10-07) | Kayla McDermott -> Feni Varughese | BUC-3622; Drive 1IkVrFjSB3UiAb4JAG-h63quqFn0q1XPX
2025-09-26 | CoreLogic POC in v4-rating-model repo (rent service; property table Nov 2025). Rating work, not pre-fill | (Pluribus) | GitHub commits 8a84bea9, 75cd6433
2025-10-14 | DISCO-429 "Replace 360Value with a cheaper alternative for RCE calculation": 360Value ~$600k/yr; in-house model to ingest Smarty, CoreLogic/Cotality, BuildZoom (Eng Development) | David Tulig -> David Curry | DISCO-429
2025-12-10 | carlos pulls Cape permits data (RBT-4822); ATTOM ids ride inside | Cameron Davison | carlos 3b08a493
2025-12-18 | ReportAll follow-up after a call: data dictionary, pricing PDF, draft license, free-trial steps. Next steps: "Kayla will determine the estimated # of credits needed"; "another call in January with the head of finance" | Scott Nelson (ReportAll) -> Kayla McDermott, Rob Soule | Gmail thread 19fd7d8dfe41e93b
2026-01-22 | BUC-4235 "carlos - Clean up unused estated code" (PR #811 merged 01-23; estated_report table deprecated); BUC-4213 Smarty enriched attribution for property features; parent epic BUC-4263 "Compute attributions in carlos" | Joseph Barratt | Jira; GitHub
2026-02-17 | Steadily RCE model v2.1.3 initial release entry (XGBoost, 252 features) | David Curry | Confluence ENG 4022403075 "Model Update Log"
2026-04-20 | "Goodbye 360Value Staged Approach" gdoc: 8 stages; expand property detail values mapped from Smarty and 360Value; Steadily RCE shadow then primary; keep 360Value at ~10% for drift | Rob Soule | Drive 1y8WCaIs24Kf6SwZcEoqLjyfEXneDbfjwXhCCvlJ0LEI
2026-05-26 | "smarty enriched did not have any value for exterior wall or construction type, and we seeded value360 with EWF N. It seems like it's always been that way though since we switched from estated" | Joseph Barratt | #eng-sprint-goodbye-360value
2026-06-03 to 06-10 | Smarty incidence per attribute (Metabase dashboards 10887, 11481), historical usage analysis, recommendation doc (roof type 5 groups, heating sources its own epic) | Joseph Barratt, Carnell Washington Jr, Mitchell Thomson, Rob Soule | #eng-sprint-goodbye-360value
2026-07-28 | INBOX-1416 "Integrate landglide into our quoting or claims lookup process" (To Do, unassigned); Christine tagged Suph on it 08-02 | David Tulig; Christine Luo | Jira; Gmail
2026-08-04 | PLB-551 carlos removes property-override overlay (Ralphie owns patches policy-side). Done 08-17 | Tim Thomas | PLB-551
2026-08-06 | Rob forwards the ReportAll thread to Suph: "the backing data of Landglide, which UW is using as a source of truth and navigation to County records" | Rob Soule | Gmail
2026-08-07 | Pre-fill flip briefing + thesis validation: six fields pre-filled 99-100%; 65-79% of UW corrections land on untouched pre-fill; agent-entered values corrected 2-6x more often | Suph | Drive docs 1D0buzRPNCYGp9pLIufXTxhL9neSWU-BUZAbHXiVD_Ug, 17mB91NlbA8gV_uBqlGMJj62HjOBnqhtrkagRK8CQt0E; #eng-prod-leadership-team
2026-08-09 / 08-17 / 08-21 | DISCO-740 (DAMR umbrella), DISCO-746 (attribute accuracy), DISCO-756 (vendor exploration) | Suph | Jira
2026-08-10 | PLB-575 negative Smarty basement_sqft breaks Steadily RCE feature validation (32 occurrences in 90 days). Done 08-13 | Tim Thomas | PLB-575
2026-08-14 | BUC-5155 "Goodbye 360Value: Enable Steadily RCE" (In Progress); LaunchDarkly flag steadily-replacement-cost-enabled | Carnell Washington Jr -> Joseph Barratt | BUC-5155
2026-08-10 (page) | 6-week plan Aug 17 to Sep 26: "Enable Villa (Steadily RCE) in AZ" (XS, "V360 cutover"). No pre-fill line item | Christine Luo | Confluence PM 4308041735
2026-08-18 | Bryan Hartwig (Head of Insurance Product): "Do we have a team focused on improving our pre-fill? This seems to be a very frequent complaint" (SF quadplex pre-filled with 2 kitchens). David Curry: "Team Pluribus has this as our long-term mandate, however in the short term we are focused on state expansion. Christine or David Tulig would be able to help more with where pre-fill improvement lands." | Bryan Hartwig, David Curry | Gmail thread 1a016757a74b3667
2026-08-19 | Suph reignites the ReportAll thread: evaluate 3 or 4 vendors vs Smarty (backtest, shadow, live). Christine Luo (VP Product): "Absolutely, feel free to add it under Discussion Topics" for the product bi-weekly | Suph, Christine Luo | Gmail thread 19fd7d8dfe41e93b
2026-08-24 / 08-25 | PRD "Fix pre-fill data to stop underwriters correcting our quotes"; epic BUC-5223 "Improve the pre-fill so underwriters stop correcting it" (To Do, unassigned) | Suph | Drive 1Yxx-xK1NTvnuJFdcEk3lYCBrTKJaWih1C9gPc6tgULA; BUC-5223
2026-08-25 | Will Henry DM: agreed order = roof model, attestations, ask UW where the data came from, then test vendors | Suph, Will Henry | Slack DM
2026-08-28 | BUC-5253 follow-up epic for Steadily RCE beyond AZ | Joseph Barratt | #eng-sprint-goodbye-360value
2026-08-31 | Datha's KS property: neither 360Value nor Smarty had the basement, "pre-fill had nothing to fill" | Suph, Rob Soule | #eng-sprint-goodbye-360value

---

## 4. Sources and vendors

| Source | Role in pre-fill | Status | Key dates and evidence |
| --- | --- | --- | --- |
| Estated | Original attribute seed (sqft, year built, property type, roof, stories, units, beds/baths, land value) | **Dead.** Replaced by Smarty enrichment Feb 2025; LLW dead path removed 2025-05-14; carlos code deleted 2026-01-23; enum value and migrations remain | landlordweb 2020-08-31; carlos by 2022-01-19; DATA-141 (85% fill); BUC-2055; BUC-2873; BUC-4235 / PR #811 |
| Smarty (SmartyStreets address validation) | Address normalize, lat/lng, county, Smarty Key used as cache key | **Live** since 2020 (landlordweb) / 2021 (carlos) | landlordweb 7c4efacab (2020-10-09); carlos 9f475b9c "use smarty for all lookups" (2021-12-14) |
| Smarty US Address Enrichment | Current attribute seed for 360Value and Steadily RCE (construction, exterior walls, foundation, garage, heat, roof cover/frame, stories, year built, sqft, basement, etc.) | **Live, load-bearing** since Feb 2025 (flag removed 2025-05-14). Known holes: often no construction type / exterior wall (seeded "EWF N"), negative basement_sqft | DATA-141 (95% fill); BUC-2238 / PR #426 (2025-02-19); BUC-3415; BUC-4213; Joseph 2026-05-26; PLB-575 |
| Verisk 360Value | RCE engine; its parsed report overrides the seeded dwelling fields; hard defaults 1980 / 2300 when the seed is missing | **Live** (on ~96% of recent quotes per the Aug 2026 PRD); being replaced by Steadily RCE, AZ first; plan keeps it at ~10% for drift checks. ~$600k/yr | Drive guides 2021-07-01; landlordweb #573 (2021-07-07); SWA-870; DISCO-429; Goodbye 360Value doc; BUC-5155 |
| Steadily RCE (Villa / Plinko, velma repo) | Consumer of pre-fill, not a source; replaces 360Value's valuation | **Rolling out**: model v2.1.3 dated 2026-02-17; BUC-5155 In Progress; "Enable Villa in AZ" in the current 6-week plan | DISCO-429; Model Update Log; BUC-4961 doc gen; BUC-5253 |
| Cape Analytics | Roof age / condition / overhang / solar; permits (Dec 2025) | **Live** since Oct 2021 | carlos 8ed75b30 (SWA-560); RBT-4822 |
| Zesty | Roof pitch / wind-hail for required states | **Live** since Jan 2025 | COS-55; carlos 81433aa0 |
| ATTOM | No integration; ids appear inside Cape permit payloads | **Candidate** (named in the Aug 2026 PRD for property-type codes); never evaluated | carlos 3b08a493; UW guide notes it repackages assessor data |
| ReportAll (LandGlide's backing data) | UW's de facto source of truth via LandGlide | **Talked to, stalled.** Sales call and follow-up 2025-12-18 (pricing, draft license, free trial); planned January finance call has no trace; thread revived 2026-08-06 / 08-19; INBOX-1416 LandGlide integration idea (To Do). API covers only 2 of the 4 target fields per the PRD | Gmail 19fd7d8dfe41e93b; INBOX-1416; LandGlide trial email to Suph 2026-08-26 |
| CoreLogic / Cotality | Named as an RCE data input (DISCO-429); POC in v4-rating-model (Sep to Nov 2025); cat-model PDFs in Drive (2025) | **Evaluated for rating / cat, not for pre-fill** | GitHub v4-rating-model commits; DISCO-429; Drive CoreLogic PDFs |
| Regrid | None in code or Jira; UW guide lists it as an aggregator | **Not evaluated** (PRD uses it as a price anchor) | Confluence TU 3049619469 |
| First American | Nothing found anywhere | Not evaluated | (searched Jira, Slack, Drive, GitHub) |
| SmartSource (Verisk) | Nothing found outside Suph's PRD | Idea only | PRD Aug 2026 |
| Others in the data inventory | RedZone, Guy Carpenter, Google Elevation (hazard, not attributes); Google Maps Places (PLB-90, LLM grounding for prohibited business) | Live for other purposes | DISCO-350; PLB-90 |

Note on the UW side: the UW space page "Locating & Verifying Property Records" (Katie Campbell, last modified 2026-08-14) documents UW's own waterfall: county assessor card first, then appraisal / MLS, then aggregators (LandGlide, Regrid, Zillow), then aerial imagery. It explicitly says LandGlide, CoreLogic, ATTOM and Regrid "all ultimately trace their square footage field back to the same source, the county assessor's building card." That is the counter-argument any vendor bake-off will meet.

---

## 5. Who owns pre-fill today

Short answer: **nobody owns "make the pre-fill accurate."** The plumbing is owned by whoever is touching carlos that week; the mandate is claimed by Pluribus but deferred; the decision on where it lands sits with Christine Luo and David Tulig; the only active push is yours.

- **The question was asked out loud on 2026-08-18.** Bryan Hartwig (Head of Insurance Product): "Do we have a team focused on improving our pre-fill? This seems to be a very frequent complaint." (Gmail thread 1a016757a74b3667)
- **Pluribus (David Curry)** answered: "Team Pluribus has this as our long-term mandate, however in the short term we are focused on state expansion. Christine or David Tulig would be able to help more with where pre-fill improvement lands." No Confluence or Jira artifact writes that mandate down; the Pluribus Ops Review pages (Jul to Aug 2026) do not mention pre-fill. Pluribus does own the adjacent pieces: Steadily RCE / DISCO-429 (David Curry), and carlos hygiene tickets PLB-551 and PLB-575 (Tim Thomas).
- **Christine Luo (VP Product)** accepted the topic onto the product bi-weekly (2026-08-19), authored the 2025 classifier discovery (DISCO-350), and wrote the Aug 17 to Sep 26 six-week plan, which has no pre-fill line item (nearest: "Enable Villa (Steadily RCE) in AZ", "MLS photos for interior property condition detection").
- **David Tulig (CTO)** originated most of the history (SWA-43 2021, DISCO-98 2021, SWA-10402 2024, DISCO-429 2025, INBOX-1416 2026) and is the other person Curry named for "where pre-fill improvement lands."
- **Buc-ee's (Rob Soule, Director of Engineering)** has been the de facto owner of the plumbing since 2024: BUC-1856, BUC-2055, BUC-2238, BUC-3622, BUC-4235, BUC-4263, BUC-5155 are all BUC tickets. Engineers who did the work: Phillip Bannister (deactivated), Tim Robertson, Feni Varughese, Joseph Barratt, Carnell Washington Jr (deactivated). Your BUC-5223 sits here, To Do and unassigned; Will Henry's stated order puts vendor testing last (roof model, attestations, UW provenance question, then vendors).
- **Kayla McDermott** was the PM on the 2024 to 2025 pre-fill work (BUC-1856, BUC-776, DISCO-174, DISCO-321, BUC-3622) and took the ReportAll call in Dec 2025. She is still at Steadily (Rob referenced her PTO in Jul 2026); her recent tickets are partnerships.
- **carlos maintainers (2026 commits):** Joseph Barratt (BUC-4235, BUC-4213, PR #1342 on 2026-09-01), Tim Thomas (PLB-551, PLB-575, BUC-5058/5062/5133), Carnell Washington Jr (PR #1157), Parker Seidel (PLB-*), Cameron Davison (RBT-4822). The repo is shared across Buc-ee's, Pluribus and Robotnik projects, which is part of why no one owns the data quality end to end.

---

## 6. What I could not verify

- Whether the ReportAll "January call with the head of finance" ever happened, or why the evaluation stalled between Dec 2025 and Aug 2026. No email, Slack or Jira trace in that window.
- The exact production date Estated stopped being called. Proxies: Smarty-enriched flag removed 2025-05-14 (carlos bd579885); LLW dead path removed 2025-05-14 (BUC-2873); carlos code deleted 2026-01-23 (PR #811). Joseph's "since we switched from estated" is undated.
- Author of the 2023-05-10 gdoc "Value360 and MGA product interactions" (metadata does not show an owner; it lives in Drive 0AOn5ldM_jAQNUk9PVA).
- Anything in Slack before roughly May 2026. Searches for Estated, prefill, Smarty and vendor names returned nothing older than that, so the 2020 to 2025 chain rests on Jira, GitHub, Confluence, Drive and Gmail rather than Slack.
- Whether "Team Pluribus long-term mandate" is documented anywhere beyond David Curry's 2026-08-18 email.
- The Metabase dashboards referenced (10887 Smarty enriched values, 11481 Goodbye v360 phase 2, 1326 default values) were not opened.
- Jira coverage: I read 100% of the result pages I pulled (first 50 to 60 results, oldest first, for "prefill", "pre-fill", "Estated" (three pages through 2025-06-17), "Smarty", "360Value", the vendor-name search, the "smarty enriched / property data" search, and a property-filtered prefill search from 2023-03-13). I did not page further into "360Value" (after 2022-06-15), "Smarty" (after 2024-01-05), "pre-fill" (after 2025-07-17) or "Estated" (after 2025-06-17); the later epics were reached through direct ticket lookups and commit history instead.
- Book items I confirmed rather than took on faith: the pipeline shape, the 1980 / 2300 defaults, Estated as dead code, the ReportAll dates, David Curry's quote, BUC-5223 / DISCO-740 / 746 / 756, PLB-551, PLB-575, BUC-5155. Book items I did not independently re-check: that ATTOM ids specifically ride inside Cape payloads (commit 3b08a493 matches "attom" but I did not read the payload), and the Cape / Zesty field-level split.
