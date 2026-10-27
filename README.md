Risk Factor Gap Finder
A web app and Claude skill that screens SEC periodic filings for risk factors that speak only of what could happen on a topic where public, dated evidence suggests the risk had already come to pass.

It covers Fortune 500 companies in two separate pools, technology and energy. It is a research aid, not a compliance or liability tool. Every flag is a "candidate for counsel review."

Status: Draft. Spec v0.4 written, build not started.

The idea: two questions
Layer 1, peer comparison. How do the target's risk factors compare with peers you select? Do peers on a topic acknowledge that the risk has occurred, while the target speaks only of what could happen? Which topics do most peers disclose that the target does not?
Layer 2, event comparison. What do public, dated events show that the target's filings do not reflect? The app matches 8-K items, SEC releases, and later filing text to the target's risk factors, then computes a review window from the event to the company's first acknowledging disclosure. Periodic filings inside the window that still use hypothetical language are surfaced.

flowchart LR

    T["Target company<br/>chosen from a drop-down"] --> MODE{"Run mode"}

    MODE -->|"Peers or Both"| L1["Layer 1<br/>Peer comparison<br/>how do selected peers<br/>frame the same topics?"]

    MODE -->|"Events or Both"| L2["Layer 2<br/>Event comparison<br/>what do public events show<br/>that the filings do not?"]

    L1 --> F1["Peer framing contrast<br/>Framing shift<br/>Coverage gap"]

    L2 --> W["Review window<br/>event date to<br/>first acknowledgment"]

    W --> F2["Hypothetical-after-event<br/>flag"]

    F1 --> REP["Report with<br/>not-assessed box"]

    F2 --> REP

Why Layer 2?
A risk factor can be the misstatement. In several circuits, a cautionary statement that portrays as hypothetical a threat that has already occurred can support a securities fraud claim. The circuits disagree on how far that goes, and the Supreme Court dismissed its 2024 case on the issue without ruling. A petition in a related Section 11 case, In re ON24, is pending.

Approach
Circuits
Test, per Cahill's Oct. 23, 2026 alert
Majority
1st, 2d, 3d, 5th, 10th, D.C.
Risk had materialized and harm was near-certain
Ninth Circuit
9th
Actionable if the risk portrayed as hypothetical had already occurred
Minority
4th, 6th
Never actionable under § 10(b)


Recent decisions: Funko (9th Cir., 2026) found risk disclosures actionable. Smith v. Gap (2d Cir., 2026) did not. The tool does not predict outcomes. It finds the statements counsel would want to look at.

Run modes
Mode
Needs
Produces
Peer comparison only
Target and at least 5 peers
Layer 1 flags
Event comparison only
Target
Layer 2 flags and review windows
Both
Target and peers
All flags, with peer framing shown beside each event flag

What it can and cannot assess
Assessed
Not assessed
Whether a risk factor speaks only of what could happen (framing tag, with a quoted phrase)
Whether harm was near-certain
Whether a dated public event matches the topic
What the company knew
How selected peers frame the same topic
Whether the problem was still fixable
Whether the text changed from the prior filing
Whether cautionary language was meaningful
The review window, computed by script
Materiality, reliance, loss causation, or liability


Public events are a conservative start for the window. The real onset is often internal and earlier. Where a later disclosure states when a problem began, the tool uses that stated onset and labels it.

The live part
An EDGAR watcher follows the universe for new 10-Ks, 10-Qs, and 8-Ks. When a relevant 8-K arrives, the app extracts the event, matches it to the company's risk factors, and recomputes the review window. A replay mode plays back recorded events so demos do not depend on the network.

Broad news skimming is not in v1. Headlines may come later as an on-demand lookup for one target.

How it works
flowchart LR

    subgraph SRC["Sources"]

        EDGAR["SEC EDGAR<br/>10-K, 10-Q, 8-K"]

        REL["SEC press and<br/>litigation releases"]

    end

    subgraph BE["Backend"]

        W["EDGAR watcher"]

        T["Extract and tag<br/>topic, framing, specificity"]

        E["Event extractor<br/>8-K items, releases"]

        M["Match events to<br/>risk factors"]

        DB[("Database and<br/>vector store")]

        C["Compare and compute<br/>review windows<br/>scripts"]

        API["REST API"]

        W --> T

        W --> E

        T --> M

        E --> M

        T --> DB

        M --> DB

        DB --> C

        C --> API

    end

    CL["Claude API<br/>server side only"]

    UI["HTML front end<br/>report and timeline"]

    EDGAR --> W

    REL --> E

    CL -.-> BE

    API --> UI

Scripts count and compute dates. Claude reads, tags, matches, and explains.

Using the app (planned)
Pick a pool, technology or energy.
Pick the target company from the drop-down.
Choose a run mode: peers, events, or both.
For the peer modes, choose peers from the multi-select drop-down. It lists only companies already ingested in that pool, so every peer has a processed filing. No free text. Five peers is the minimum.
Read the report. Each flag type has its own section with verbatim quotes and citations. The peer set, run mode, and doctrine lens print in the header.
Open the timeline for any topic: filings marked by framing tag, events as vertical lines, the review window shaded.

The app works on filed, public documents only. Draft filings are excluded in v1 because they can contain material nonpublic information.

Layout (planned)
risk-factor-gap-finder/

├── SKILL.md                 Claude skill instructions (offline and notebook use)

├── Spec.md                  full specification

├── references/              taxonomy, universe, config, case notes

├── scripts/                 extract, verify quotes, compare

├── templates/gap-report.md  fixed report format

├── backend/                 API, watcher, jobs

├── web/                     static HTML front end

├── docs/                    diagrams and archived spec versions

├── data/                    raw, extracted, records, events, compare

└── eval/                    test cases, scored runs, error log

Rules the system follows
Every flag quotes the source and cites it. A script checks that quotes are verbatim.
Never "misleading," "actionable," or "required." Always "a candidate for counsel review."
Never "the risk materialized." Always "a public event dated X concerning topic Y."
Each flag type is reported in its own section.
The not-assessed box prints in every report.
Risk factors that fit no topic are listed as taxonomy candidates, never forced into one.

Known limits
The tool sees public evidence only. It cannot see what a company knew or when a problem began internally.
Peer disclosure is not a legal standard, and peer choice drives Layer 1 results. The app shows the peer set and how sensitive the flags are to it.
Circuit context is informational. Headquarters is a rough proxy for venue, and counsel decides what law governs.
Framing and specificity tags are judgments by a model. They carry quotes and get checked against human samples.
Several doctrinal points are unresolved between sources (Spec Section 2.6). The legal reviewer reads the opinions before any of it is relied on.
Extraction can fail on unusual filings. Failures go to a review queue, never guessed.

Testing
Test design is in active development. Spec.md Section 11 holds a working proposal built around litigated cases, scored on whether the tool surfaces the challenged statements and not on case outcomes. It will change.

Build priorities
Priority
Scope
P0
Universe with headquarters circuit, extraction, tagging, quote verification, target and peer drop-downs, Layer 1 report, case notes
P1
EDGAR watcher, review queue, replay mode, 8-K events, matching, review windows, timeline, Layer 2 flag
P2
SEC releases, stated-onset retrieval, staleness metric, 10-Q handling, doctrine lens, commentary
P3
News on demand, embedding map, trend flag, law-watch panel, local-only draft review

Build status
Nothing is built yet. The spec is the only artifact. Its open questions (deadline, legal reviewer, hosting, embedding model, pools, name, draft review, ownership) are in Section 14.

Disclaimer
Research aid only. Not legal advice. Output should be reviewed by qualified counsel before anyone relies on it.


