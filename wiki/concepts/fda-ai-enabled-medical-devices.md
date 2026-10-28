# FDA AI-Enabled Medical Devices

The US Food and Drug Administration regulates AI that performs a medical-device function. This page covers exam objective 9.5 as a sector-specific example of AI regulation. It describes the FDA's list of authorised AI-enabled devices, its guiding principles, and the status of its AI guidance documents. The study guide's FDA link (the "AI and ML in Software as a Medical Device" page) is now dead, returning 404.

FDA oversight sits alongside the horizontal [EU AI Act](eu-ai-act.md) and the voluntary frameworks in [AI Risk Frameworks and Standards](ai-risk-frameworks-and-standards.md).

## What the FDA publishes

The FDA maintains a "List of Artificial Intelligence-Enabled Medical Devices". It is updated periodically, is explicitly not comprehensive, and lists decisions up to at least 2026-06-29. The FDA says it will explore tagging devices that incorporate foundation models, including LLMs **[web, FDA list page]**. Together with Health Canada and the UK's MHRA, it has published guiding principles on three topics **[web, titles on the FDA page]**:
- Good Machine Learning Practice;
- Predetermined Change Control Plans (PCCPs);
- transparency.

## Guidance status (checked 2026-10-28)

The guidance on marketing-submission recommendations for a PCCP for AI-enabled device software functions is final. It was posted in December 2024 and sets out what a submission should say about planned model changes **[web, secondary reports]**.

The broader guidance on lifecycle management and marketing submissions for AI-enabled device software functions was issued as a draft in January 2025, with comments closing on 2025-04-07. The FDA's guidance page still listed it as a draft in the latest sources found. One 2026 headline reports a finalisation, but whether it refers to this guidance or to the PCCP guidance could not be established **[web, snippet-level; check the FDA guidance search before stating its status]**.

## Reading it for agents

An agent that makes diagnostic or treatment recommendations is performing a device function and falls under device regulation. An agent that only handles administrative work generally does not. Models that change after deployment are why PCCPs exist: they let planned, validated updates proceed without a new clearance for each one **[inferred]**.

## Interview Q&A

**What is a PCCP?** A predetermined change control plan. It is a plan agreed in advance that describes the modifications a model may undergo and how each will be validated, so updates do not each need a new clearance **[web for the term; definition inferred]**.

**Is the FDA's list of AI-enabled devices exhaustive?** No, and the FDA says so **[web]**.

**What is the status of the FDA's AI lifecycle guidance?** It was a draft in January 2025. The related PCCP guidance is final. Check the FDA's site for any later finalisation **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [AI and ML in Software as a Medical Device (FDA, dead link)](https://www.fda.gov/medical-devices/software-medical-device-samd/artificial-intelligence-and-machine-learning-software-medical-device) |
| 2026-10-28 | [List of AI-enabled medical devices (FDA)](https://www.fda.gov/medical-devices/artificial-intelligence-enabled-medical-devices/list-artificial-intelligence-enabled-medical-devices) |
| 2026-10-28 | [AI-enabled device software functions: lifecycle management guidance (FDA)](https://fda.gov/regulatory-information/search-fda-guidance-documents/artificial-intelligence-enabled-device-software-functions-lifecycle-management-and-marketing) |
| 2026-10-28 | [FDA guidances on AI-enabled devices (Greenlight Guru)](https://www.greenlight.guru/blog/fda-guidance-ai-enabled-devices) |
