# Paste-ready edits: pre-fill PRD and DISCO-740 (2026-09-02)

Drafts only. Nothing below has been applied. Keyed to the headings in each document so you can paste without recreating anything.

Sources for every number: [RCE-2026-09-02.md](RCE-2026-09-02.md) §2 to §5.

---

## A. Pre-fill PRD (gdoc 1OVZCVT4-Vr5m9wRy3_7y-CI0gLzh5EjTIX238EV6zYo, "BUC-5223: improve the pre-fill so underwriters stop correcting it")

**Metadata table, "Updated" row.** Change to `Sep 2, 2026`.

**TL;DR (italic paragraph).** Replace the second sentence

> Bad pre-fill shows up in more than one workstream, the dwelling-age alert and the RCE calculation among them.

with

> Bad pre-fill shows up in two places: the dwelling-age review, where underwriters fix it by hand, and the replacement cost estimate, which comes out low when the pre-fill leaves things out. Steadily RCE makes the second one bigger, because the new model only prices what it is given.

**Context, end of the paragraph.** Add one sentence:

> Since Sep 1 this PRD also has a second customer: the Steadily RCE rollout, where the same pre-fill gaps are the central risk (BUC-5271, and the Sep 10 call with Datha).

**Problem, intro paragraph, third cost.** Replace

> Third, the same wrong data feeds our replacement cost estimates, which come out low, so agents conclude we are underinsuring and place the business elsewhere.

with

> Third, the same wrong data feeds our replacement cost estimates, which come out low, so agents conclude we are underinsuring and place the business elsewhere. The new Steadily RCE model never uses values that came from 360Value, so whatever Smarty cannot supply goes blank, and blanks price low.

**Problem point 2, "The same pre-fill drags down our RCE."** Keep the four-plex sentences. Add after them:

> It gets bigger with Steadily RCE. The new model only prices what it is given and never uses values that came from 360Value, so when 360Value goes away the fields Smarty cannot supply go blank: bedrooms on 17% of quotes, exterior wall 13%, stories 9%, bathrooms 8%, basement 8% (David's table, BUC-5271). Smarty misses bedrooms or bathrooms about 35% of the time and has no kitchen count. Datha's own test quote came out low on Aug 31 because the pre-fill said slab and the house has an 1,800 sq ft basement. Not every low RCE is pre-fill: the Calabasas complaint ($241 per sq ft) sits at 360Value's normal Los Angeles level, so that one is a model question for David's team.

**Problem, optional new point 5.** Only if you want the model angle in the PRD itself. It is an inference until David confirms it.

> ### *Our own model learned from our pre-fill.*
> Steadily RCE is trained to reproduce 360Value's answers on our own quotes, and those answers were computed from the same pre-filled inputs. Where the pre-fill leaves out rooms and basements, the number the model learned to match was low for the same reason. The team's own read of where the model under-prices by 20% or more says missed basements are 31% of that tail. Better pre-fill is better training data.

**Goals, add one bullet:**

> - The per-field error rate doubles as the Steadily RCE rollout's list of which fields we will miss and how often (the Sep 10 agenda).

**Metrics.** No new row (the table stays at three). Add one line under the table:

> The same per-field error rate feeds the RCE rollout's "how much do the gaps move the valuation" read (BUC-5271; Sep 10 call).

**Appendix, add:**

> - [BUC-5271: what goes blank when 360Value goes away](https://steadily.atlassian.net/browse/BUC-5271) (David's fill-rate sheet)
> - [BUC-5155: Goodbye 360Value, enable Steadily RCE](https://steadily.atlassian.net/browse/BUC-5155)
> - [Pre-fill and the RCE: evidence memo](https://github.com/suph-steadily/suph-projects/blob/main/alert-north-star/dwelling-alert/prefill/RCE-2026-09-02.md)

---

## B. DISCO-740 (Jira idea, "Do better on 101+ homes: safely remove or smooth the dwelling-age (DAMR) alert")

Not posted. Apply yourself, or say the word and I will apply exactly this.

**"Why does it matter?", add a last line:**

> The pre-fill workstream also carries the replacement cost estimate: the same missing fields make the RCE come out low and cost deals (a SF four-plex 29% under another carrier on inputs alone), and the new Steadily RCE model only prices what it is given (BUC-5271: bedrooms blank on 17% of quotes without 360Value).

**"The plan", workstream 2 line.** Two changes:

1. Fix the PRD link. It points at the superseded doc (1Yxx-xK1NTvnuJFdcEk3lYCBrTKJaWih1C9gPc6tgULA). Replace with the live one: https://docs.google.com/document/d/1OVZCVT4-Vr5m9wRy3_7y-CI0gLzh5EjTIX238EV6zYo/edit
2. Append: `Second customer since 9/1: the Steadily RCE rollout (BUC-5155); what goes blank without 360Value is in BUC-5271. Epic: BUC-5223.`

**"How we know it worked", optional fourth bullet:**

> * Per-field pre-fill error rate measured (DISCO-746), and the Steadily RCE rollout's per-field "acceptable range" agreed from the same table.

---

## C. Brent's CA thread (Gmail thread 1a05f959136bdb83)

Your 5-bullet reply from Sep 1 stands as written (DRAFT-FINAL.md in ~/mockups/rce-prefill-brent-reply/). The Sep 1 draft id no longer resolves, so it was either sent or replaced; check the thread before touching it. Nothing new here changes the reply: the held-back facts (Calabasas = 360Value's LA level; model built to match 360Value; 20% rule is code) are in the notes if Brent or David push.
