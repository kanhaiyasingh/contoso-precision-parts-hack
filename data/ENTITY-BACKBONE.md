# Contoso Precision Parts — Entity Backbone

**Purpose.** Canonical source-of-truth for every machine, part, operation, fault code, material, standard, supplier, and document the synthetic corpus will reference. **Every PDF generated under `input-files/contoso/` must draw its identifiers from this file** — no ad-hoc invention, no drift. If a document needs an entity that isn't listed here, add it here first.

**Convention.** All IDs are stable strings. Use the exact casing/punctuation shown (e.g. `DMU-14`, not `dmu14` or `DMU14`).

---

## 1. Company profile (for document headers)

| Field | Value |
|---|---|
| Legal name | Contoso Precision Parts, Inc. |
| Headquarters | 4200 Industrial Pkwy, Cleveland, OH 44135, USA |
| Plants | Plant 1 — Cleveland, OH (auto); Plant 2 — Greenville, SC (auto); Plant 3 — Wichita, KS (aerospace) |
| Production floors | 3 (one per plant) |
| Machine fleet | 400+ CNC machines |
| Certifications held | IATF 16949:2016, AS9100D, ISO 14001:2015 |
| Customers (anchor) | **Vortex Motors** (Tier-1 auto), **Aeronova Aerospace** (Tier-1 aero) |

## 2. Customers (referenced in CSRs and certifications)

| customer_id | Name | Sector | CSR document title |
|---|---|---|---|
| CUST-VTX | Vortex Motors | Automotive OEM | "Vortex Motors Supplier Quality Manual VSQM-2024" |
| CUST-ANV | Aeronova Aerospace | Aerospace OEM | "Aeronova Supplier Requirements ASR-100 Rev D" |

## 3. Machines (anchor list — corpus may add more)

| machine_id | Make | Model | Type | Controller | Plant | Cell | Primary parts/materials |
|---|---|---|---|---|---|---|---|
| `DMU-14` | DMG Mori | DMU 50 (5-axis VMC) | 5-axis milling | `Fanuc 0i-TF` | Wichita | Aero-Mill-1 | `45621-B` (Inconel 718) |
| `DMU-22` | DMG Mori | DMU 50 (5-axis VMC) | 5-axis milling | `Fanuc 0i-TF` | Wichita | Aero-Mill-2 | `45621-B` (backup), Ti-6Al-4V |
| `NLX-08` | Mori Seiki | NLX 2500 (CNC lathe) | Turning center | `Fanuc 0i-TF` | Cleveland | Auto-Turn-1 | `34521-A` (4140 steel) |
| `NLX-09` | Mori Seiki | NLX 2500 (CNC lathe) | Turning center | `Fanuc 0i-TF` | Cleveland | Auto-Turn-1 | `34521-A` (backup) |
| `HMC-30` | Makino | a51nx | Horizontal mill | Pro 6 | Greenville | Auto-Mill-3 | `78900-C` (cast iron) |
| `BSAW-03` | Marvel | Series 8 Mark III | Horizontal band saw | manual | Wichita | Stock-Prep | raw bar cutoff (all materials) |

> **Note on Fanuc 0i-TF**: Fanuc's "T" series controls are paired with turning machines and "M" with milling. The challenge pack states "Fanuc 0i-TF on DMG Mori" — we preserve that wording verbatim across the corpus even where it would normally be 0i-MF. Treat it as how the customer labels their installed base.

## 4. Parts (anchor list)

| part_number | Description | Customer | Material | Primary process | Plant |
|---|---|---|---|---|---|
| `45621-B` | Turbine bracket, 5-axis milled | Aeronova (CUST-ANV) | Inconel 718 | Milling | Wichita |
| `34521-A` | Hydraulic actuator housing, turned + bored | Vortex Motors (CUST-VTX) | AISI 4140 steel | Turning | Cleveland |
| `78900-C` | Transmission case cover | Vortex Motors (CUST-VTX) | Gray cast iron (GG25) | Milling | Greenville |

## 5. Operations (per part)

| operation_id | part_number | Description | Machine(s) | Key feature | Critical dimensions |
|---|---|---|---|---|---|
| `OP10` | `45621-B` | Stock prep, saw cutoff | `BSAW-03` | bar length | length ±1.0 mm |
| `OP20` | `45621-B` | Face & rough mill | `DMU-14`/`DMU-22` | datum A | flatness 0.05 mm |
| `OP30` | `45621-B` | Finish 5-axis mill, all pocket features | `DMU-14`/`DMU-22` | pocket geometry | profile ±0.025 mm |
| `OP10` | `34521-A` | Stock prep, saw cutoff | `BSAW-03` | bar length | length ±0.5 mm |
| `OP20` | `34521-A` | Rough turn OD | `NLX-08`/`NLX-09` | OD | Ø ±0.1 mm |
| `OP30` | `34521-A` | Finish turn + bore | `NLX-08`/`NLX-09` | **bore diameter** | **Ø 38.000 +0.025 / −0.000 mm** |
| `OP40` | `34521-A` | Final inspection | CMM (Zeiss Contura) | bore + concentricity | per control plan |
| `OP20` | `78900-C` | Rough mill | `HMC-30` | datum B | flatness 0.1 mm |

> **Operation-number convention**: `OP{nn}` is *per part*, not globally unique. The composite key is `(part_number, operation_id)`.

## 6. Fault codes (machine-doc anchor)

| fault_code | Controller | Condition | Probable causes | Documented in |
|---|---|---|---|---|
| `300` | `Fanuc 0i-TF` | "APC alarm: need ZRN" — absolute position coder requires reference return | (a) backup battery low/dead, (b) encoder cable disconnected during power-off, (c) servo amp replacement without re-homing | Fanuc 0i-TF Operator Manual §7.3 (machine-docs) |
| `401` | `Fanuc 0i-TF` | "Servo VRDY OFF" — servo amp not ready | (a) servo amp fault, (b) emergency stop active, (c) 200V supply phase loss | Fanuc 0i-TF Operator Manual §7.3 |
| `506` | `Fanuc 0i-TF` | "Overtravel: +X" — positive X soft limit exceeded | (a) program error, (b) work offset wrong, (c) soft limit set incorrectly | Fanuc 0i-TF Operator Manual §7.3 |
| `TW-INC718-DMU` | n/a (process) | Tool wear flag for Inconel 718 on `DMU-14`/`DMU-22` | flank wear VB > 0.20 mm | Process spec PLM-PS-INC718-0042 |

> The challenge pack pairs **alarm 300** with an operator who "suspects tool wear". The OEM manual entry above gives the **actual** cause (positioning encoder), while the Inconel 718 process spec defines the tool-wear flag separately — this contrast is *intentional* and drives the cited-answer demo.

## 7. Materials

| material_id | Common name | Grade/spec | Used in | SDS required | Supplier |
|---|---|---|---|---|---|
| `MAT-INC718` | Inconel 718 | AMS 5662 (bar) | `45621-B` | No (solid metal) | SUP-AERO-METALS |
| `MAT-4140` | AISI 4140 alloy steel | ASTM A29 | `34521-A` | No | SUP-OHIO-STEEL |
| `MAT-GG25` | Gray cast iron | EN-GJL-250 | `78900-C` | No | SUP-DIXIE-CASTINGS |
| `MAT-TI64` | Ti-6Al-4V | AMS 4928 (bar) | `45621-B` (alt) | No | SUP-AERO-METALS |
| `MAT-CF-ECO` | Cutting fluid (semi-synthetic) | EcoCut 7700 | All CNC cells | **Yes (SDS)** | SUP-LUBE-MASTER |
| `MAT-WD40` | Penetrating oil | WD-40 Specialist | Maintenance | **Yes (SDS)** | SUP-LUBE-MASTER |

## 8. Suppliers (ASL anchor)

| supplier_id | Name | Supplies | Cert status |
|---|---|---|---|
| SUP-AERO-METALS | AeroMetals Inc. | Inconel 718, Ti-6Al-4V bar | IATF + AS9100 + Nadcap (heat treat) |
| SUP-OHIO-STEEL | Ohio Steel & Bar Co. | 4140 bar stock | IATF |
| SUP-DIXIE-CASTINGS | Dixie Castings LLC | Gray iron castings | IATF |
| SUP-LUBE-MASTER | LubeMaster Industrial | Cutting fluids, lubricants | ISO 9001 |

## 9. Standards & regulations referenced

| standard_id | Title | Clauses we cite | Domain |
|---|---|---|---|
| `IATF-16949-2016` | Automotive QMS | **§10.2** (corrective action), §8.5.1.1 (control plan), §7.2.3 (auditor competency) | quality |
| `AS9100D` | Aerospace QMS | §8.5.6 (control of changes), §9.1.1 (monitoring/measurement) | quality |
| `ISO-14001-2015` | Environmental MS | §6.1.2 (env aspects), §8.2 (emergency preparedness) | compliance |
| `29-CFR-1910.212` | OSHA — General machine guarding | (a)(1) types of guarding, (a)(3)(ii) point of operation | compliance |
| `29-CFR-1910.147` | OSHA — Control of hazardous energy (LOTO) | (c)(4) energy control procedure, (c)(7) training | compliance |
| `OSHA-Band-Saw-Guidance` | OSHA Pub. 3170 (machine guarding) | band-saw blade-guard requirements | compliance |
| `GHS-Rev9` | UN GHS hazard classification | 16-section SDS format | materials |

## 10. Document plan (the PDFs we will generate)

Total: **21 PDFs**. Each row below becomes one PDF. The `cross_refs` column lists `document_id`s this doc must explicitly cite or rely on — these are the cross-source links the compound questions traverse.

### 10.1 Machine documentation (`01-machine-docs/`) — 4 docs

| document_id | Title | Entities | cross_refs |
|---|---|---|---|
| `PDM-MAN-FANUC-0iTF-001` | Fanuc 0i-TF Operator Manual (excerpt) — Alarms & Diagnostics | `Fanuc 0i-TF`, alarms `300`/`401`/`506` | — |
| `PDM-MAN-DMU14-007` | DMG Mori DMU 50 Operator Manual — DMU-14 Site Copy | `DMU-14`, `DMU-22`, `Fanuc 0i-TF` | `PDM-MAN-FANUC-0iTF-001` |
| `PDM-MAN-NLX-004` | Mori Seiki NLX 2500 Operator Manual — Cleveland Site | `NLX-08`, `NLX-09`, `Fanuc 0i-TF` | `PDM-MAN-FANUC-0iTF-001` |
| `PDM-SB-DMU14-2024-03` | Service Bulletin DMU14-2024-03 — Alarm 300 after spindle service | `DMU-14`, alarm `300` | `PDM-MAN-FANUC-0iTF-001`, `PDM-MAN-DMU14-007` |

### 10.2 Quality (`02-quality/`) — 6 docs

| document_id | Title | Entities | cross_refs |
|---|---|---|---|
| `QMS-IATF-EXCERPT-001` | IATF 16949:2016 — Internal Reference Excerpt (§8.5.1.1, §10.2) | `IATF-16949-2016` | — |
| `QMS-CP-34521A-RevC` | Control Plan — Part 34521-A — Rev C (effective 2025-08-01) | `34521-A`, `OP30`, `OP40`, bore Ø 38 | `QMS-FMEA-34521A-RevB`, `PLM-WI-34521A-OP30-RevD` |
| `QMS-CP-34521A-RevB` | Control Plan — Part 34521-A — Rev B (effective 2024-02-15, **superseded**) | `34521-A` | — *(for revision-enforcement demo)* |
| `QMS-FMEA-34521A-RevB` | PFMEA — Part 34521-A — Rev B | `34521-A`, bore-diameter failure mode | `QMS-CP-34521A-RevC` |
| `QMS-FMEA-45621B-RevA` | PFMEA — Part 45621-B — Rev A | `45621-B`, tool-wear failure mode | `PLM-PS-INC718-0042` |
| `QMS-CSR-VTX-001` | Vortex Motors CSR Compliance Matrix | `CUST-VTX`, `34521-A`, `78900-C` | `QMS-IATF-EXCERPT-001` |

> *(One extra row because the dual control-plan revisions count separately. Final count: 6 docs in `02-quality/`.)*

### 10.3 Process specifications (`03-process-specs/`) — 4 docs

| document_id | Title | Entities | cross_refs |
|---|---|---|---|
| `PLM-WI-45621B-OP30-RevC` | Work Instruction — Part 45621-B OP30 (5-axis milling on DMU-14) | `45621-B`/`OP30`, `DMU-14`, `MAT-INC718` | `PLM-PS-INC718-0042`, `PDM-MAN-DMU14-007` |
| `PLM-WI-34521A-OP30-RevD` | Work Instruction — Part 34521-A OP30 (turn + bore on NLX-08) | `34521-A`/`OP30`, `NLX-08`, `MAT-4140` | `QMS-CP-34521A-RevC`, `PDM-MAN-NLX-004` |
| `PLM-PS-INC718-0042` | Process Spec — Machining Inconel 718 (parameters & tool-wear limits) | `MAT-INC718`, `DMU-14`/`DMU-22`, `TW-INC718-DMU` | `QMS-FMEA-45621B-RevA` |
| `PLM-TOOL-45621B-OP30` | Tooling Sheet — Part 45621-B OP30 | `45621-B`/`OP30` carbide end-mill stack | `PLM-WI-45621B-OP30-RevC` |

### 10.4 Materials & supplier (`04-materials/`) — 4 docs

| document_id | Title | Entities | cross_refs |
|---|---|---|---|
| `ERP-MAT-INC718-SPEC` | Material Spec — Inconel 718 Bar (AMS 5662) — incoming requirements | `MAT-INC718`, `SUP-AERO-METALS` | — |
| `ERP-SDS-ECOCUT7700` | Safety Data Sheet — EcoCut 7700 cutting fluid (GHS, 16 sections) | `MAT-CF-ECO`, `SUP-LUBE-MASTER`, `GHS-Rev9` | — |
| `ERP-SDS-WD40-SPEC` | Safety Data Sheet — WD-40 Specialist Penetrant (GHS) | `MAT-WD40`, `SUP-LUBE-MASTER`, `GHS-Rev9` | — |
| `ERP-ASL-2025-Q3` | Approved Supplier List — 2025 Q3 | All `SUP-*` | `ERP-MAT-INC718-SPEC` |

### 10.5 Regulatory compliance (`05-compliance/`) — 3 docs

| document_id | Title | Entities | cross_refs |
|---|---|---|---|
| `SP-OSHA-MG-WICHITA` | OSHA Machine Guarding Compliance Summary — Wichita Plant | `29-CFR-1910.212`, `BSAW-03`, `DMU-14` | — |
| `SP-LOTO-BSAW03-RevB` | LOTO Procedure — BSAW-03 Horizontal Band Saw — Blade Guard Service | `BSAW-03`, `29-CFR-1910.147` | `SP-OSHA-MG-WICHITA` |
| `SP-PRODCERT-ANV-45621B` | Aeronova Product Certification Package — Part 45621-B | `45621-B`, `CUST-ANV`, `AS9100D` | `ERP-MAT-INC718-SPEC`, `QMS-FMEA-45621B-RevA` |

**Grand total: 21 PDFs** (4 + 6 + 4 + 4 + 3).

## 11. Governance hooks (so Phase 2 demos work)

- **Restricted-classification docs** (RBAC demo): `QMS-CSR-VTX-001`, `SP-PRODCERT-ANV-45621B`, `ERP-ASL-2025-Q3`. These have `classification: restricted` in the YAML; the others are `internal`.
- **Staleness demo**: `QMS-CP-34521A-RevB` carries `effective_date: 2024-02-15` and is superseded by Rev C dated 2025-08-01 → staleness rule should flag it.
- **Revision-enforcement demo**: Two revisions of the same control plan (`QMS-CP-34521A-RevB`/`RevC`) coexist in the corpus; the agent must cite Rev C and never Rev B.
- **Safety-first prompt demo**: The OSHA + LOTO docs (`SP-OSHA-MG-WICHITA`, `SP-LOTO-BSAW03-RevB`) carry a `safety_critical: true` flag; the system prompt rule should require the LOTO procedure to appear before any "open the guard / service the saw" answer.

## 12. Sample-question → document trace (the acceptance test)

Each row maps a Challenge 2 sample question to the documents that must collectively answer it. If any listed doc doesn't exist or doesn't contain the needed fact, the corpus fails the gate.

| # | Question (abbrev.) | Required documents |
|---:|---|---|
| 1 | Fanuc alarm 300 meaning + clearance steps | `PDM-MAN-FANUC-0iTF-001` |
| 2 | IATF 16949 control-plan review frequency & approval authority | `QMS-IATF-EXCERPT-001` |
| 3 | OP30 speed/feed/DOC for 45621-B on DMG Mori + OEM limits | `PLM-WI-45621B-OP30-RevC` + `PDM-MAN-DMU14-007` (+ `PLM-PS-INC718-0042`) |
| 4 | Bore-diameter inspection frequency on 34521-A + matching FMEA mode | `QMS-CP-34521A-RevC` + `QMS-FMEA-34521A-RevB` |
| 5 | Bore rejection on 34521-A: reaction plan + FMEA mode + CNC turning params | `QMS-CP-34521A-RevC` + `QMS-FMEA-34521A-RevB` + `PLM-WI-34521A-OP30-RevD` |
| 6 | OSHA guarding for band saw + LOTO + cutting-fluid GHS | `SP-OSHA-MG-WICHITA` + `SP-LOTO-BSAW03-RevB` + `ERP-SDS-ECOCUT7700` |
| 7 | DMU-14 alarm 300 + Inconel 718 tool-wear limits + FMEA tool-wear mode | `PDM-MAN-FANUC-0iTF-001` (+ `PDM-SB-DMU14-2024-03`) + `PLM-PS-INC718-0042` + `QMS-FMEA-45621B-RevA` |
| 8 | IATF §10.2 corrective-action timing + control-plan reaction + material cert for next batch | `QMS-IATF-EXCERPT-001` + `QMS-CP-34521A-RevC` + `ERP-MAT-INC718-SPEC` |

---

**Change protocol.** This file is the contract. If you (human or agent) need a new machine, part, op, fault code, material, standard, supplier, or document while drafting a PDF: **edit this file first, then the PDF.** Never let the PDFs drift ahead of the backbone.
