# Nova Biomedical Bioprofile FLEX2 — Publisher Data Contract Specification

**Version:** 0.1 DRAFT  
**Date:** 2026-04-01  
**Status:** For review  
**Source documentation:** BioProfile FLEX2 IFU (PN 57960 Rev F, March 2023)  
**Scope:** Defines the canonical event set, payload schemas, and integration contract for the FLEX2 as a publisher in the site-wide traceability and genealogy architecture.

---

## 1. Instrument Overview

The Bioprofile FLEX2 is a multi-module automated cell culture analyzer producing up to 16 parameters per sample analysis. The instrument comprises four analytical modules, each with independent consumables, calibration lifecycles, and QC regimes:

| Module | Parameters | Consumables |
|--------|-----------|-------------|
| pH/Gas | pH, pCO₂, pO₂ | MicroSensor Card, Calibrator Cartridge |
| Chemistry | Gluc, Lac, Gln, Glu, NH₄⁺, Na⁺, K⁺, Ca²⁺ | MicroSensor Card, Calibrator Cartridge, Reagent Cartridge |
| Cell Density/Viability (CDV) | Total cell density, viable cell density, viability, cell diameter | Bottle Pack, Reagent Cartridge |
| Osmometer (Osm20 or Osm48) | Osmolality | Tubes, wiper ring (Osm20 only) |

The FLEX2 maintains five immutable, timestamped log systems: Audit Log, Error Log, Calibration Log, Maintenance Log, and Warranty Log. All logs are exportable as CSV.

---

## 2. Acquisition Path

**Primary interface:** OPC UA server on the FLEX2 Bridge computer.

The FLEX2 ships with a proprietary OPC server (licensed separately, PN 60644). The Bridge computer is a Windows PC that acts as a gateway between the Analytical Unit (AU) and external systems. OPC UA and DA endpoints are available on port 59888.

**Integration flow:**

```
FLEX2 AU → Bridge Computer (OPC UA Server, port 59888) → HighByte Intelligence Hub → MQTT Broker
```

HighByte subscribes to OPC UA tags, enriches with downstream context (Odoo batch/lot resolution, SENAITE sample linkage), and publishes canonical events to the MQTT broker.

**Dependency:** The OPC tag namespace and available tags are defined in the BioProfile FLEX2 OPC Manual (PN 60644). That document is required to map the events below to specific OPC tag addresses.

**Action required:**
- [ ] Obtain OPC license from Nova Biomedical
- [ ] Obtain OPC Manual (PN 60644) for tag structure documentation
- [ ] Confirm OPC UA endpoint accessibility from HighByte network segment
- [ ] Determine which events below are directly available as OPC tags vs. require derivation

---

## 3. MQTT Topic Structure

Following the site convention: `{org}/{site}/{domain}/{model}/{event}/{id}`

```
{org}/{site}/analytical/flex2-{analyzer_id}/{event_name}/{event_id}
```

Where `{analyzer_id}` is the Analyzer ID configured in FLEX2 Settings, allowing multiple instruments per site.

---

## 4. Canonical Event Set

### 4.1 Sample.Analyzed

**Trigger:** Completion of any sample analysis (manual, load-and-go, 96-well plate, ESM, or OLS autosampler).

**Audit log entry:** `Executed Sample Analysis Sample ID {sample_ID} Sample Time {sample_time} Sample Type {sample_type} Batch ID {batch_ID} Vessel ID {vessel_ID} Cell Type {cell_type}`

**Frequency:** Per sample. Up to 700/day at max throughput.

**Full panel payload — all active module results included:**

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| sample_time | ISO 8601 datetime | Yes | Primary key. Instrument-assigned timestamp. |
| sample_id | string | Yes | Operator-entered or auto-indexed. |
| batch_id | string | Yes | Operator-entered. Links to Odoo batch. |
| vessel_id | string | Yes | Operator-entered. Identifies source bioreactor. |
| cell_type | string | No | Operator-entered. |
| sample_type | string | Yes | Pre-configured sample type name. |
| analyzer_id | string | Yes | Instrument identifier. |
| operator | string | Yes | Logged-in user at time of analysis. |
| sampling_mode | enum | Yes | `manual` / `load_and_go` / `96well` / `esm` / `ols` |
| cup_position | integer | Conditional | Required for load-and-go and 96-well. |
| ols_vessel | string | Conditional | Required for OLS autosampler. RSM name. |
| chemistry_dilution | string | No | Dilution ratio applied to chemistry module. |
| cdv_dilution | string | No | Dilution ratio applied to CDV module. |
| cdv_inspection_type | string | No | Cell inspection configuration used. |
| pre_dilution_multiplier | number | No | If operator-entered. |
| vessel_temperature | number | No | °C. If operator-entered. |
| vessel_pressure | number | No | If operator-entered. |
| sparging_o2_pct | number | No | If operator-entered. |
| modules_active | array of string | Yes | Which modules were selected for this analysis. |
| results | array of Result | Yes | See Result object below. |

**Result object (one per measured parameter):**

| Field | Type | Notes |
|-------|------|-------|
| parameter | string | e.g., `pH`, `Gluc`, `VCD`, `Osm` |
| value | number | Measured value. |
| unit | string | Unit of measure (configurable per parameter). |
| status | enum | `normal` / `suppressed` / `qc_lockout` / `dependency_error` / `calibration_error` |
| module | string | `ph_gas` / `chemistry` / `cdv` / `osmometer` |

**Consumer notes:** Parameters may be absent if the module was not selected for this analysis. A status of `qc_lockout` means the parameter was locked out due to a prior QC failure — no result value is reported. A `dependency_error` means a dependent parameter is unavailable (e.g., Gln unavailable because Glu is locked out).

---

### 4.2 QC.Analyzed

**Trigger:** Completion of an onboard or external QC analysis.

**Audit log entry:** `Executed QC Analysis for Lot {lot_number} Level {level} Sample Time {sample_time}`

**Frequency:** Scheduled (user-configurable intervals for onboard), ad-hoc for external.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| sample_time | ISO 8601 datetime | Yes | Primary key. |
| qc_type | enum | Yes | `onboard` / `external` |
| qc_lot | string | Yes | QC material lot number. |
| qc_level | string | Yes | e.g., `high` / `low` |
| module | string | Yes | `ph_gas` / `chemistry` / `osmometer` |
| analyzer_id | string | Yes | |
| operator | string | Yes | Logged-in user. |
| results | array of QCResult | Yes | See QCResult object below. |
| overall_pass | boolean | Yes | True if all parameters in range. |

**QCResult object:**

| Field | Type | Notes |
|-------|------|-------|
| parameter | string | |
| value | number | Measured value. |
| unit | string | |
| lower_limit | number | Configured lower QC range. |
| upper_limit | number | Configured upper QC range. |
| pass | boolean | Within range. |

---

### 4.3 QC.LockoutTriggered

**Trigger:** An onboard QC analysis reports out-of-range results when QC Lockout is enabled. Also triggers when a scheduled QC is missed (e.g., reagent cartridge expired or empty).

**Frequency:** Low — only on QC failures or missed schedules.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| timestamp | ISO 8601 datetime | Yes | |
| analyzer_id | string | Yes | |
| triggering_qc_sample_time | datetime | Conditional | If triggered by a QC failure. |
| triggering_qc_lot | string | Conditional | |
| triggering_qc_level | string | Conditional | |
| trigger_reason | enum | Yes | `qc_failure` / `missed_schedule` |
| locked_parameters | array of string | Yes | Directly locked parameters. |
| cascade_locked_parameters | array of string | No | Parameters locked due to dependency (e.g., Gln locked because Glu failed). |

**Consumer notes:** QC Lockout persists across power cycles. A locked parameter remains locked until a subsequent onboard QC analysis passes for all applicable levels. QC Lockout does not apply to external controls.

**Dependency chain reference (from IFU):**
- If Glu locked → Gln unavailable
- If Na⁺ or K⁺ locked → NH₄⁺ unavailable
- If pH locked → pCO₂ unavailable

---

### 4.4 Calibration.Completed

**Trigger:** Completion of a pH/Gas, Chemistry, CDV, or Osmometer module calibration — automatic or manual.

**Calibration Log entry:** Documents 2-point calibration slope data per parameter.

**Frequency:** Automatic 2-point calibration every 2 hours for pH/Gas and Chemistry. 1-point calibration during every sample analysis. Manual calibrations operator-initiated.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| timestamp | ISO 8601 datetime | Yes | |
| analyzer_id | string | Yes | |
| module | string | Yes | `ph_gas` / `chemistry` / `cdv` / `osmometer` |
| calibration_type | enum | Yes | `auto_2pt` / `auto_1pt` / `manual` / `smart_maintenance` |
| operator | string | Conditional | For manual calibrations. |
| parameter_results | array of CalResult | Yes | |
| overall_pass | boolean | Yes | |

**CalResult object:**

| Field | Type | Notes |
|-------|------|-------|
| parameter | string | |
| pass | boolean | |
| slope | number | From 2-point calibration. |
| consecutive_failures | integer | Tracks toward warranty threshold (3). |

---

### 4.5 Calibration.Failed

**Trigger:** Any parameter fails to calibrate. Status icon turns red; parameter unavailable for sampling.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| timestamp | ISO 8601 datetime | Yes | |
| analyzer_id | string | Yes | |
| module | string | Yes | |
| failed_parameters | array of string | Yes | |
| consecutive_failure_count | integer | Yes | Per parameter. At 3, warranty support triggered. |
| warranty_eligible | boolean | Yes | True if count ≥ 3. |

---

### 4.6 MicroSensorCard.Installed

**Trigger:** Smart Maintenance detects a new MicroSensor Card via RFID. Initiates hydration sequence followed by calibration.

**Maintenance Log entry:** Captures date/time, user, description, lot/part number.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| timestamp | ISO 8601 datetime | Yes | |
| analyzer_id | string | Yes | |
| module | enum | Yes | `ph_gas` / `chemistry` |
| card_lot | string | Yes | From RFID tag. |
| card_part_number | string | Yes | From RFID tag. |
| previous_card_lot | string | No | If available from prior maintenance log. |
| operator | string | Yes | Logged-in user. |
| hydration_initiated | boolean | Yes | Should always be true on successful install. |

**Consumer notes:** The instrument is not ready for sampling until hydration completes and subsequent calibration passes. A `Calibration.Completed` event with `calibration_type: smart_maintenance` will follow.

---

### 4.7 CalibratorCartridge.Installed

**Trigger:** Smart Maintenance detects a new Calibrator Cartridge via RFID. Initiates priming and calibration.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| timestamp | ISO 8601 datetime | Yes | |
| analyzer_id | string | Yes | |
| module | enum | Yes | `ph_gas` / `chemistry` |
| cartridge_lot | string | Yes | From RFID. |
| cartridge_expiration | ISO 8601 date | Yes | From RFID. |
| cartridge_part_number | string | Yes | |
| operator | string | Yes | |

---

### 4.8 ReagentCartridge.Installed

**Trigger:** Smart Maintenance detects new Reagent Cartridge or CDV Bottle Pack/Reagent Cartridge via RFID. CDV installation triggers Adjust Intensity sequence.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| timestamp | ISO 8601 datetime | Yes | |
| analyzer_id | string | Yes | |
| module | enum | Yes | `chemistry` / `cdv` |
| cartridge_type | enum | Yes | `reagent` / `bottle_pack` |
| cartridge_lot | string | Yes | From RFID. |
| cartridge_expiration | ISO 8601 date | Yes | From RFID. |
| cartridge_part_number | string | Yes | |
| operator | string | Yes | |

---

### 4.9 QCCartridge.Installed

**Trigger:** Installation of onboard QC cartridge (pH/Gas/Osmo or Chemistry QC).

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| timestamp | ISO 8601 datetime | Yes | |
| analyzer_id | string | Yes | |
| module | enum | Yes | `ph_gas_osmo` / `chemistry` |
| qc_lot | string | Yes | From RFID. |
| qc_expiration | ISO 8601 date | Yes | From RFID. |
| samples_remaining | integer | Yes | From RFID. ~30-day supply. |
| operator | string | Yes | |

---

### 4.10 OsmometerTubes.Changed

**Trigger:** Operator initiates Change Tubes sequence. Triggers osmometer module calibration upon completion.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| timestamp | ISO 8601 datetime | Yes | |
| analyzer_id | string | Yes | |
| osmometer_type | enum | Yes | `osm20` / `osm48` |
| tubes_installed | integer | Yes | 10 (Osm20) or 49 (Osm48). |
| wiper_ring_replaced | boolean | Conditional | Osm20 only. |
| operator | string | Yes | |

---

### 4.11 Instrument.Error

**Trigger:** Any error reported by the FLEX2 system.

**Error Log entry:** Date/time and description. Immutable.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| timestamp | ISO 8601 datetime | Yes | |
| analyzer_id | string | Yes | |
| error_description | string | Yes | As reported by instrument. |
| affected_module | string | No | If determinable from error context. |
| affected_parameters | array of string | No | If determinable. |

---

### 4.12 User.Session

**Trigger:** Operator login or logout.

**Audit log entry:** `User {user_name} logged in` / `User {user_name} logged out`

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| timestamp | ISO 8601 datetime | Yes | |
| analyzer_id | string | Yes | |
| action | enum | Yes | `login` / `logout` / `auto_logoff` |
| username | string | Yes | |
| privilege_level | string | No | If available via OPC. |

---

## 5. Consumer Contract Summary

The publisher contract defines the full canonical payload for each event. Consumer systems subscribe and filter as follows:

| Consumer | Events Subscribed | Filter / Transformation |
|----------|-------------------|------------------------|
| Manufacturing Record (PS) | `Sample.Analyzed` | Filter to pH (± pCO₂). Enrich with Odoo Batch ID resolution. Write to batch record. |
| MSAT / Process Development | `Sample.Analyzed` | Filter to VCD, viability, cell diameter. Enrich with process step context. Feed trend analysis. |
| QC / QA | `Sample.Analyzed`, `QC.Analyzed`, `QC.LockoutTriggered`, `Calibration.Completed`, `Calibration.Failed` | Full panel. Full instrument state context. Supports disposition and audit readiness. |
| Data Lake / Historian | All events | No filter. Full payload. Historization and ad-hoc analysis. |
| Genealogy Graph | All events | Builds traceability links: Sample → active consumable lots, bracketing QC, calibration state, operator. |

All filtering and enrichment occurs in HighByte, not at the publisher.

---

## 6. Genealogy Relationships

A single `Sample.Analyzed` event connects to the following instrument-state facts, established by the non-sample events:

- **Active MicroSensor Cards** — which pH/Gas and Chemistry card lots were installed (from most recent `MicroSensorCard.Installed` per module)
- **Active Calibrator Cartridges** — which calibrator lots were installed (from most recent `CalibratorCartridge.Installed` per module)
- **Active Reagent Cartridges** — which reagent lots were installed (from most recent `ReagentCartridge.Installed` per module)
- **Bracketing QC** — the most recent `QC.Analyzed` result per module/level before this sample
- **Calibration state** — the most recent `Calibration.Completed` before this sample
- **Operator** — the active `User.Session` at the time of analysis
- **Instrument health** — absence of unresolved `Instrument.Error` or `Calibration.Failed` events

These relationships enable the auditor's question: "Was this instrument qualified to produce this result at this time?"

---

## 7. Open Items and Dependencies

| # | Item | Owner | Status |
|---|------|-------|--------|
| 1 | Obtain OPC license from Nova Biomedical | Procurement | Not started |
| 2 | Obtain OPC Manual (PN 60644) for tag mapping | Engineering | Not started |
| 3 | Map each event in Section 4 to specific OPC UA tags | Engineering | Blocked by #2 |
| 4 | Confirm which non-sample events (calibration, maintenance, errors) are exposed via OPC vs. only in CSV logs | Engineering | Blocked by #2 |
| 5 | Define HighByte flow configurations per consumer | Integration | Blocked by #3 |
| 6 | Validate parameter dependency chains in QC Lockout against live instrument behavior | QC | Not started |
| 7 | Determine if 1-point (per-sample) calibrations are available as discrete OPC events or only logged internally | Engineering | Blocked by #2 |
| 8 | Establish Analyzer ID naming convention for multi-instrument sites | All | Not started |
| 9 | Define SENAITE inbound interface for FLEX2 results (manual entry, CSV import, or API) | LIMS | Not started |

---

## 8. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1 | 2026-04-01 | — | Initial draft. Event set derived from IFU PN 57960 Rev F. OPC tag mapping pending PN 60644. |
