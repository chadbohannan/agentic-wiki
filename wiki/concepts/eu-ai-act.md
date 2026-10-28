# EU AI Act

The EU AI Act, Regulation (EU) 2024/1689, regulates AI systems by risk tier and places separate duties on providers of general-purpose AI (GPAI) models. Its application dates were amended in 2026 by the Digital Omnibus on AI. This page covers exam objective 9.5. Here the study guide is out of date: it links the 2021 Commission proposal (COM(2021) 206), which the adopted Regulation superseded when it entered into force on 2024-08-01 **[web, Commission AI Act Service Desk]**.

The Act's oversight and transparency duties connect to [Human Oversight and Accountability](human-oversight-and-accountability.md) and [Agent Transparency and Explainability](agent-transparency-and-traceability.md). Its logging duty is detailed in [Audit Trails and Traceability](audit-trails-and-traceability.md). The frameworks it sits beside are compared in [AI Risk Frameworks and Standards](ai-risk-frameworks-and-standards.md), sector regimes such as the FDA's are in [FDA AI-Enabled Medical Devices](fda-ai-enabled-medical-devices.md), and the licence terms that govern model use are in [Model and Dataset Licensing](model-and-dataset-licensing.md).

## Risk tiers

The Future of Life Institute's unofficial summary describes four tiers **[web, unofficial; consistent with background knowledge]**:
- **Unacceptable risk** is prohibited under Article 5. This covers:
  - manipulation, and exploiting vulnerabilities;
  - social scoring;
  - untargeted scraping of facial images;
  - inferring emotions at work or school, with exceptions;
  - certain biometric categorisation, and real-time remote biometric identification for law enforcement.
- **High risk** covers safety components of products under Annex I, and the use cases in Annex III: employment, education, essential services, law enforcement, migration and justice.
- **Limited risk** carries transparency duties, for example for chatbots and deepfakes.
- **Minimal risk** carries no specific duties.

Providers of high-risk systems must provide:
- risk management and data governance;
- technical documentation;
- logging, which Article 12 requires to be automatic over the system's lifetime;
- human oversight;
- accuracy, robustness and cybersecurity;
- a quality management system;
- conformity assessment, CE marking and registration in the EU database.

## General-purpose AI models

All GPAI model providers must keep technical documentation, inform downstream providers, have a copyright policy and publish a summary of training content. Models presumed to pose systemic risk, by exceeding 10^25 training FLOPs or by designation, carry extra duties: evaluations including adversarial testing, risk mitigation, serious-incident reporting and cybersecurity. A voluntary GPAI Code of Practice was published on 2025-07-10 **[web, unofficial summary]**. According to the Commission's Service Desk, GPAI rules have applied since 2025-08-02. The AI Office's enforcement powers apply from 2026-08-02: information requests, model access for evaluations, mitigation, restriction or recall, and fines of up to 3% of global annual turnover **[web, official]**.

## Timeline (verified 2026-10-28)

The Commission's AI Act Service Desk timeline reflects the Digital Omnibus amendments **[web, official, read directly]**:

| Date | What applies |
| ---- | ------------ |
| 2024-08-01 | Entry into force |
| 2025-02-02 | Definitions, AI literacy, prohibitions |
| 2025-08-02 | GPAI rules and governance |
| 2026-08-02 | Most remaining rules, including Article 50 transparency and enforcement |
| 2026-12-02 | New prohibitions on AI-generated non-consensual sexual deepfakes and child sexual abuse material; transitional deadline for some synthetic-content providers to meet Article 50(2) |
| 2027-08-02 | At least one regulatory sandbox per Member State |
| 2027-12-02 | Annex III high-risk rules |
| 2028-08-02 | Annex I high-risk rules (AI in regulated products) |

The amending act is Regulation (EU) 2026/1744. It was published in the Official Journal on 2026-07-24 and entered into force on 2026-07-27, three days later, so that it would settle the rules before the 2026-08-02 general application date **[web; consistent across several law-firm summaries; EUR-Lex text not opened]**. Its history:
1. The Commission proposed it on 2025-11-19.
2. A provisional agreement followed on 2026-05-07.
3. Parliament approved it on 2026-06-16, by 423 to 57 with 174 abstentions.
4. The Council gave final approval in late June 2026.

It also amends the EASA and Machinery regulations, expands the AI Office's oversight, and creates a legal basis for processing special-category data to detect bias **[web, secondary]**. Sources disagree on whether the Article 50 labelling duty for systems already on the market moved to 2026-12-02, so check the Official Journal text.

The consequence for the study guide: the high-risk obligations it would have dated to 2026-08-02 were deferred to 2027-12-02 and 2028-08-02. The prohibitions and GPAI duties were not delayed **[web]**.

## Reading it for agents

An agent is not a category of its own. Classification depends on the use case. An HR-screening agent is high-risk under Annex III, while a customer-service chatbot owes transparency. Providers that build on GPAI models inherit upstream documentation, and an integrator that substantially modifies a model can become a provider. Tooling such as NeMo Guardrails supports controls, but does not by itself make a system compliant **[inferred]**.

## Interview Q&A

**What are the risk tiers?** Unacceptable (banned), high, limited (transparency duties) and minimal **[web]**.

**What changed from the 2021 proposal?** It was adopted as Regulation (EU) 2024/1689, with a GPAI chapter added that sets the systemic-risk threshold at 10^25 FLOPs **[web, inferred]**.

**When do the high-risk rules apply now?** Annex III from 2027-12-02, and Annex I from 2028-08-02, after the 2026 Digital Omnibus **[web, official]**.

**What applies to a RAG chatbot?** Article 50 transparency from 2026-08-02. It is high-risk only if used in an Annex III domain **[web, inferred]**.

**Were the GPAI obligations delayed?** No. They have applied since 2025-08-02, with enforcement powers from 2026-08-02 **[web, official]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Proposal for a Regulation on AI, COM(2021) 206 (EUR-Lex)](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:52021PC0206) |
| 2026-10-28 | [AI Act Service Desk: implementation timeline](https://ai-act-service-desk.ec.europa.eu/en/ai-act/timeline/timeline-implementation-eu-ai-act) |
| 2026-10-28 | [AI Act high-level summary (Future of Life Institute)](https://artificialintelligenceact.eu/high-level-summary/) |
| 2026-10-28 | [EU AI Act Article 12: record-keeping](https://artificialintelligenceact.eu/article/12/) |
| 2026-10-28 | [EU Digital Omnibus on AI enters into force (Hunton)](https://www.hunton.com/privacy-and-cybersecurity-law-blog/eu-digital-omnibus-on-ai-enters-into-force) |
| 2026-10-28 | [Digital AI Omnibus delays key deadlines (Cooley)](https://cdp.cooley.com/digital-ai-omnibus-delays-key-deadlines-introduces-new-rules/) |
