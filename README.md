# XAI-Research-Journal

Research journal for Explainable AI, Block 1 - Year 3 (Immersive Software Engineering, University of Limerick).

**Repository:** https://github.com/SamMcl1/XAI-Research-Journal  
**Latest paper:** [journal/Paper/Final-Paper.tex](journal/Paper/Final-Paper.tex)

## Members

Sam Mcloughlin - 24403822  
Hugh Feehan - 24405523  
Danny Quinn - 24422622  
Andy Cao - 24422703

## Topic

**Explaining the Flag, Not the Player: A Critical Analysis of SHAP Explanations for Reviewing False Positives in AI Anti-Cheat**

We look at whether SHAP explanations help human reviewers spot players who were wrongly flagged as cheating. Our main case study is XGuardian (USENIX Security 2026), a server-side aim-assist detector for FPS games. We checked its pipeline against the authors' released code.

**Research question:** Can SHAP-based explanations be expected to help reviewers recognise false-positive cheating flags, and what evidence currently supports that expectation?

**Scope:** A structured case study of XGuardian's paper and released code. We did not train our own model (decided 18 Oct). Behavioural detection only, so visual aimbots are out of scope (decided 2 Oct).

## Key findings

- XGuardian's explanations don't really explain why a player was flagged. The match-level one just repeats the score, and the per-kill one shows where to look but not why.
- The reported false positive rate doesn't show how many flagged players are actually innocent.
- There isn't enough evidence yet that explanations help reviewers catch false flags.
- Explanations are best used to find evidence, not as proof of cheating.

## How the report traces to the journal

| Report section | Where it came from |
| --- | --- |
| Topic and research question | [2026-09-17](journal/Meetings/2026-09-17.md), [2026-09-24](journal/Meetings/2026-09-24.md), [2026-10-02](journal/Meetings/2026-10-02.md) (Option B chosen), [2026-10-18](journal/Meetings/2026-10-18.md) |
| Background: Behavioural Anti-Cheat | [Player_behaviour_anticheat.md](journal/Research/Player_behaviour_anticheat.md), [XGuardian_anticheat_research.md](journal/Research/XGuardian_anticheat_research.md) |
| Background: Evaluating Explanations with People | [2026-10-08-SHAP_alert_review_research.md](journal/Research/2026-10-08-SHAP_alert_review_research.md) |
| Technical Approach: The XGuardian Pipeline | [XGuardian deeper dive, 2026-10-05](journal/Profiles/Deeper%20Dive%20on%20ai%20in%20anticheats%20-Regarding%20XGuardian%20anticheat%20study.md) |
| Neither Level Decomposes the Verdict | Code check of 5 Oct, discussed in [2026-10-18](journal/Meetings/2026-10-18.md), Q2, revised in [2026-10-23](journal/Meetings/2026-10-23.md) |
| Detector Metrics Are Not Reviewer Outcomes | [2026-10-18](journal/Meetings/2026-10-18.md), Q3 |
| Limitations (no new model) | [2026-10-18](journal/Meetings/2026-10-18.md), Q1, revised in [2026-10-23](journal/Meetings/2026-10-23.md) |

Dead ends we dropped: the other five domains ([2026-09-24](journal/Meetings/2026-09-24.md)), YOLO visual detection and adversarial robustness ([2026-10-02](journal/Meetings/2026-10-02.md)), and GameNGen ([GameNGen_diffusion_research.md](journal/Research/GameNGen_diffusion_research.md)).

## Repository layout

| Folder | Contents |
| --- | --- |
| `journal/Profiles/` | Team interests and backgrounds, plus a deeper dive into XGuardian |
| `journal/Meetings/` | Meeting notes showing how we chose and narrowed the topic |
| `journal/Research/` | Notes on the papers we reviewed (XGuardian, SHAP alert review, Minecraft behaviour anti-cheat, YOLO aimbots, GameNGen) |
| `journal/Paper/` | Final paper in LaTeX (`Final-Paper.tex`) |
| `journal/Images/` | Figures used in the notes |
