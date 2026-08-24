# CASE Ontology Extensions (AI-Generated)

## Overview

This repository contains AI-generated extension ontologies for the [CASE (Cyber-investigation Analysis Standard Expression)](https://caseontology.org/) Investigation namespace. These extensions were created for **Project VIC's Autopsy-Rust project** to improve explainability of digital forensic investigation outputs.

All extensions in this repository are scoped to the `case/investigation` namespace domain. UCO-level extensions are maintained separately in the [Unified Cyber Ontology Extensions (AI-Generated)](https://github.com/vulnmaster/Unifed-Cyber-Ontology-Extensions-AI-Generated) repository.

## Extension: `investigation-ai-ext`

**Namespace URI:** `https://ontology.caseontology.org/case/investigation-ai-ext/`
**Prefix:** `investigation_ai_ext`
**File:** [`ontology/investigation-ai-ext.ttl`](ontology/investigation-ai-ext.ttl)

### Purpose

Digital forensics examiners operate under search constraints (warrants, scope limitations, time windows) and must explain not only what evidence they found, but also what they did not find and why. This extension adds concepts for:

- **Hypothesis testing** -- formulating and testing statements about digital account activity
- **Structured findings** -- documenting inculpatory, exculpatory, or neutral outcomes
- **Negative findings** -- explicitly recording "no evidence found under stated constraints"
- **Search constraints** -- making the boundaries of evidence searches explicit and auditable

### Non-Assertion Policy

Autopsy-Rust does **not** assert account-to-human ownership within its CASE/UCO exports. OS accounts, service accounts, and other digital identities are exported as `uco-observable:DigitalAccount` objects with machine-derived identifiers (SIDs, usernames, profile paths). The mapping of digital accounts to human individuals is left to post-processing hypothesis testing workflows, which may use the `Hypothesis`, `HypothesisTestAction`, and `Finding` classes defined here.

### Classes

| Class | Parent | Description |
|-------|--------|-------------|
| `investigation_ai_ext:Hypothesis` | `uco-core:UcoObject` | A testable statement about digital account activity or evidence patterns |
| `investigation_ai_ext:HypothesisTestAction` | `case-investigation:InvestigativeAction` | An action performed to test a hypothesis against evidence |
| `investigation_ai_ext:Finding` | `uco-core:UcoObject` | A structured outcome of a test, with polarity and confidence |
| `investigation_ai_ext:NegativeFinding` | `investigation_ai_ext:Finding` | Explicit documentation that expected evidence was not found |
| `investigation_ai_ext:SearchConstraint` | `uco-core:UcoObject` | Documented constraints on an evidence search (scope, time, query, config) |

### Properties

| Property | Type | Domain | Range | Description |
|----------|------|--------|-------|-------------|
| `testsHypothesis` | Object | `HypothesisTestAction` | `Hypothesis` | Links a test action to the hypothesis it tests |
| `producedFinding` | Object | `InvestigativeAction` | `Finding` | Links an action to its produced finding (subPropertyOf `uco-action:result`) |
| `supportedBy` | Object | `Finding` | `uco-core:UcoObject` | Links a finding to supporting evidence |
| `underConstraint` | Object | `UcoObject` | `SearchConstraint` | Links a finding or test to its constraints |
| `polarity` | Datatype | `Finding` | `xsd:string` | `"inculpatory"`, `"exculpatory"`, or `"neutral"` |
| `confidenceNote` | Datatype | `Finding` | `xsd:string` | Narrative confidence/limitations statement |
| `hypothesisStatement` | Datatype | `Hypothesis` | `xsd:string` | The textual hypothesis |
| `hypothesisScope` | Datatype | `Hypothesis` | `xsd:string` | Scope/context summary |
| `constraintScope` | Datatype | `SearchConstraint` | `xsd:string` | Data sources/paths included or excluded |
| `constraintStartTime` | Datatype | `SearchConstraint` | `xsd:dateTime` | Time window start |
| `constraintEndTime` | Datatype | `SearchConstraint` | `xsd:dateTime` | Time window end |
| `constraintQuery` | Datatype | `SearchConstraint` | `xsd:string` | Query/pattern used |
| `constraintConfiguration` | Datatype | `SearchConstraint` | `xsd:string` | Tool/module configuration |

## Connection to Autopsy-Rust Artifact Types

Several Autopsy-Rust artifact types map naturally to the investigation extension concepts:

| Autopsy-Rust Artifact | Maps to | Notes |
|-----------------------|---------|-------|
| `AnalysisResult` | `investigation_ai_ext:Finding` | Module analysis outcomes can be wrapped as Findings with polarity. From **Plaso** (TimelineEvent), **Malware Scan** (MalwareHitFacet), **Object Detection** (ObjectDetectionFacet), **Central Repo**, etc. |
| `VerificationFailed` | `investigation_ai_ext:NegativeFinding` | Data integrity failures are negative findings documenting what was checked and failed |
| `InterestingFileHit` / `InterestingArtifactHit` | `investigation_ai_ext:Finding` (via `producedFinding`) | Rule-based matches can be linked to the action that produced them |
| Keyword search results | `investigation_ai_ext:SearchConstraint` + `Finding` | Search queries, time windows, and data source scope are captured as constraints |
| **TimelineEvent** (Plaso) | Observable with `observable_ai_ext:TimelineEventFacet`; action → `producedFinding` | Timeline events can be linked to the Plaso InvestigativeAction as results/findings |
| **AnalysisResult** (Malware Scan, Object Detection) | Observable with `MalwareHitFacet` / `ObjectDetectionFacet` (UCO); action → `producedFinding` | Module-specific facets in UCO; the producing action is typed (e.g. Malware Scanning, Object Detection) in `action_ai_ext` |

The `producedFinding` property (subPropertyOf `uco-action:result`) links `InvestigativeAction` nodes to their `Finding` outputs. The UCO `action-ai-ext` vocabulary terms (see the [UCO Extensions](https://github.com/vulnmaster/Unifed-Cyber-Ontology-Extensions-AI-Generated) repository) define the specific action types whose results can flow into Findings (e.g. `PlasoTimelineExtraction`, `MalwareScanning`, `ObjectDetection`, `iOSArtifactExtraction`).

## Compatibility

- **Base ontology:** CASE v1.4.0 / UCO v1.4.0
- **Import chain:** `investigation-ai-ext` imports `case/investigation/1.4.0`, `uco/action/1.4.0`, and `uco/core/1.4.0`
- Live official CASE/UCO is 1.5.0. This repository still documents and imports 1.4.0. It has not been retargeted.
- These extensions are additive and do not modify or conflict with base CASE/UCO terms

## Origin

These extension ontologies were **AI-generated** for Project VIC's Autopsy-Rust project to increase explainability of exported CASE/UCO provenance graphs in digital forensic investigations.

## CDO Community Playground

These extensions have been aligned with the [CDO Community Playground](https://github.com/casework/CASE-Profile-Example) requirements (see project root `CDO-Playground-Instructions.md` and `CDO-Playground-Testing.md`):

- All classes use `owl:Class` with explicit `rdfs:subClassOf` (no `owl:NamedIndividual` for concept categories).
- OWL and SHACL are separated: ontology in `ontology/investigation-ai-ext.ttl`, shapes in `ontology/investigation-ai-ext-shapes.ttl`.
- Exemplar A-Box with UUID-based IRIs: `exemplars/investigation-ai-ext-exemplar.ttl`.

## Validation

**Validated with case_validate** (pass the extension ontology so extension classes are recognized):

```bash
# investigation-ai-ext exemplar (run from this repo root)
case_validate --built-version case-1.4.0 \
  --ontology-graph ontology/investigation-ai-ext.ttl \
  exemplars/investigation-ai-ext-exemplar.ttl
```

The CASE typo-checker looks for IRIs under `https://ontology.unifiedcyberontology.org/`; the investigation extension uses `https://ontology.caseontology.org/case/investigation-ai-ext/`, so it is not treated as UCO. For full SHACL validation, use the CASE-Profile-Example workflow (`make -j check` after injecting ontology, shapes, and exemplar per `CDO-Playground-Testing.md`).

## License

Apache-2.0
