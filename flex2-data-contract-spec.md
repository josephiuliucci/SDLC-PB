# Nova Biomedical Bioprofile FLEX2 — Publisher Data Contract Specification

**Version:** 0.3 DRAFT  
**Date:** 2026-04-01  
**Status:** For review  
**Source documentation:**
- BioProfile FLEX2 IFU (PN 57960 Rev F, March 2023)
- BioProfile FLEX2 OPC Server IFU (PN 60644 Rev B, March 2024)

**Scope:** Defines the canonical event set, OPC tag mappings, payload schemas, and integration contract for the FLEX2 as a publisher in the site-wide traceability and genealogy architecture.

---

## 1. Instrument Overview

The Bioprofile FLEX2 is a multi-module automated cell culture analyzer producing up to 16 parameters per sample analysis. The instrument comprises four analytical modules, each with independent consumables, calibration lifecycles, and QC regimes:

| Module | Parameters | Consumables |
|--------|-----------|-------------|
| pH/Gas | pH, pCO₂, pO₂ | MicroSensor Card, Calibrator Cartridge |
| Chemistry | Gluc, Lac, Gln, Glu, NH₄⁺, Na⁺, K⁺, Ca²⁺ | MicroSensor Card, Calibrator Cartridge, Reagent Cartridge |
| Cell Density/Viability (CDV) | Total cell density, viable cell density, viability, cell diameter | Bottle Pack, Reagent Cartridge |
| Osmometer (Osm20 or Osm48) | Osmolality | Tubes, wiper ring (Osm20 only) |

The FLEX2 maintains five immutable, timestamped log systems: Audit Log, Error Log, Calibration Log, Maintenance Log, and Warranty Log. All logs are exportable as CSV to the Bridge computer (`C:\Export`).

---

## 2. Acquisition Architecture

### 2.1 Primary Interface: OPC UA

The FLEX2 ships with a proprietary OPC server (licensed separately) installed on the Bridge computer. The Bridge is a Windows PC acting as a gateway between the Analytical Unit (AU) and external systems.

| Setting | Value |
|---------|-------|
| OPC UA Port | 59888 |
| OPC DA ProgID | `Nova.Biomedical.OPC.DA.Server` |
| Namespace Index | NS = 2 |
| Identifier Type | String for all tags |
| Tag root | `<-OPCSystemObjects->` (read-only) / `<-OPCSystemCommands->` (writable) |
| Software requirement | FLEX2 Software ≥ 3.2.18138; latest OPC Server recommended |

### 2.2 Critical Architectural Insight: State Tags, Not Event Streams

The OPC server exposes **current state**, not discrete events. Tags represent the instrument's present condition and update in-place when state changes:

- **Consumable status tags** (e.g., `ChemCard->LotNumber`) reflect what is *currently installed*. When a MicroSensor Card is swapped, the `LotNumber`, `InstallationDate`, and `Hydrated` tags update to the new card's values. There is no discrete "card installed" event — the event must be **derived by Ignition via change detection** on the `LotNumber` tag.
- **Calibration status tags** (e.g., `DP_ChemCal->ChemCal->Gluc->CalibrationStatus`) show `Calibrated` or `Uncalibrated`. No slope data, no timestamp of last calibration, no pass/fail history.
- **Parameter alert/warning tags** (e.g., `Parameters->pH->Alert`) are booleans that flag when a parameter is unavailable. QC Lockout must be **inferred** from these flags — there is no explicit lockout event or tag.
- **Sample and QC result tags** are "last value" registers that update when a new analysis completes. The `SampleTime` / `TimeStamp` tag change serves as the trigger.

Ignition Event Streams is the mechanism that converts these OPC state transitions into discrete, publishable events. Ignition subscribes to the FLEX2 OPC UA server as a native OPC UA client, detects value transitions on subscribed tags, and emits events to downstream consumers via the MQTT broker.

### 2.3 Supplementary Interface: CSV Log Export

The following data is **not available via OPC** and requires CSV log export from the Bridge shared folder (`C:\Export`) if needed:

| Data | Log Source | OPC Available? |
|------|-----------|----------------|
| Calibration slope data (2-point) | Calibration Log | No — only `Calibrated`/`Uncalibrated` status string |
| Error descriptions | Error Log | No — only per-parameter `Alert`/`Warning` booleans |
| User login/logout | Audit Log | No |
| Configuration change audit trail | Audit Log | No |
| Maintenance action history | Maintenance Log | No — state tags show current consumable, not history |
| Warranty claims | Warranty Log | No |
| Consecutive calibration failure count | Calibration Log | No |

**Recommendation:** Implement a lightweight file watcher on `C:\Export` for Audit Log and Error Log CSVs to supplement OPC for audit/compliance events. This is lower priority than the OPC path for analytical results.

### 2.4 Integration Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        FLEX2 Bridge Computer                           │
│                                                                        │
│   OPC UA Server (port 59888)          C:\Export\ (CSV logs)           │
│   ├── OPCSystemObjects                ├── Results/                    │
│   │   ├── HistoricalSampleResults     │   ├── SampleResultsYYYY-MM.csv│
│   │   ├── QCResults                   │   └── QCResultsYYYY-MM.csv   │
│   │   ├── *Card / *PackStatus         ├── Logs/                      │
│   │   ├── Parameters->*.Alert         │   └── MaintenanceLog*.csv    │
│   │   └── DP_*Cal->CalibrationStatus  ├── AuditableAction*.csv       │
│   └── OPCSystemCommands               ├── CalibrationLog*.csv        │
│       ├── ChemistryCalibration        └── Errors*.csv                │
│       ├── GasCalibration                                              │
│       └── *QcLevel*                                                   │
└────────────┬──────────────────────────────────┬───────────────────────┘
             │ OPC UA subscription              │ File watch (optional)
             ▼                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                   Ignition (Event Streams)                             │
│   • OPC UA client subscribes to FLEX2 Bridge                          │
│   • Event Streams detect value transitions on state tags              │
│   • Converts state changes into discrete canonical events             │
│   • Enrichment with MES batch/lot context, LIMS sample linkage        │
│   • Consumer-specific filtering and transformation                    │
└────────────────────────────────┬────────────────────────────────────────┘
                                 │ Publish
                                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          MQTT Broker                                   │
│   {org}/{site}/analytical/flex2-{analyzer_id}/{event}/{id}            │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.5 OPC Command Tags (Writable)

The OPC server also exposes writable command tags for remotely triggering instrument operations. These are relevant for automation but not directly for the publisher contract — they're included here for completeness.

| Command | Tag Path | Data Type | Action |
|---------|----------|-----------|--------|
| Chemistry Calibration | `OPCSystemCommands->ChemistryCalibration` | Boolean | Write 1 to trigger 2-point calibration |
| pH/Gas Calibration | `OPCSystemCommands->GasCalibration` | Boolean | Write 1 to trigger 2-point calibration |
| Chemistry QC Level 1 | `OPCSystemCommands->ChemistryQcLevel1` | Boolean | Write 1 to trigger onboard QC |
| Chemistry QC Level 2 | `OPCSystemCommands->ChemistryQcLevel2` | Boolean | Write 1 to trigger onboard QC |
| pH/Gas QC Level 1 | `OPCSystemCommands->GasQcLevel1` | Boolean | Write 1 to trigger onboard QC |
| pH/Gas QC Level 2 | `OPCSystemCommands->GasQcLevel2` | Boolean | Write 1 to trigger onboard QC |
| CDV Adjust Intensity | `OPCSystemCommands->AdjustIntensity` | Boolean | Write 1 to trigger |
| Clear Wells | `OPCSystemCommands->ClearWells` | Boolean | Write 1 to clear |
| Depro Wells | `OPCSystemCommands->DeproWells` | Boolean | Write 1 to bleach clean |

---

## 3. MQTT Topic Structure

Following the site convention: `{org}/{site}/{domain}/{model}/{event}/{id}`

```
{org}/{site}/analytical/flex2-{analyzer_id}/{event_name}/{event_id}
```

The `{analyzer_id}` value is available via OPC tag `OPCSystemObjects->Settings->AnalyzerID`.

---

## 4. Canonical Event Set

### Event Availability Summary

| Event | OPC Source | Derivation Method | Priority |
|-------|-----------|-------------------|----------|
| `Sample.Analyzed` | `HistoricalSampleResults` tags | Trigger on `SampleTime` change | **MVP** |
| `QC.Analyzed` | `QCResults` tags | Trigger on `SampleTime` change | **MVP** |
| `Consumable.Changed` | `*Card` / `*PackStatus` tags | Poll + detect `LotNumber` change | **Phase 2** |
| `Calibration.StatusChanged` | `DP_*Cal->CalibrationStatus` tags | Poll + detect value change | **Phase 2** |
| `Parameter.AlertChanged` | `Parameters->*->Alert/Warning` | Poll + detect boolean change | **Phase 2** |
| `Instrument.Error` | Not in OPC | CSV Error Log export | **Phase 3** |
| `User.Session` | Not in OPC | CSV Audit Log export | **Phase 3** |

---

### 4.1 Sample.Analyzed

**OPC source:** `HistoricalSampleResults` object tags (recommended by Nova over `SampleResults`).

**Trigger:** Ignition Event Streams detects a change in `HistoricalSampleResults->SampleTime`. On change, read all tags in the `HistoricalSampleResults` tree and assemble the canonical event payload.

**Note:** Nova's OPC manual recommends using `HistoricalSampleResults` over `SampleResults` for gathering analysis data. Both contain equivalent fields; `HistoricalSampleResults` retrieves all results regardless of how the sample was initiated.

**OPC Tag Mapping:**

| Payload Field | OPC Tag Path | Data Type |
|--------------|-------------|-----------|
| sample_time | `HistoricalSampleResults->SampleTime` | DateTime |
| modified_time | `HistoricalSampleResults->ModifiedTime` | DateTime |
| timestamp | `HistoricalSampleResults->TimeStamp` | DateTime |
| time_in_tray | `HistoricalSampleResults->TimeInTray` | String |
| operator | `HistoricalSampleResults->StartTags->Operator` | String |
| sample_source | `HistoricalSampleResults->StartTags->SampleSource` | String |
| sample_type | `HistoricalSampleResults->StartTags->SampleType` | String |
| tray_location | `HistoricalSampleResults->StartTags->TrayLocation` | Int32 |
| autosampler_port | `HistoricalSampleResults->StartTags->AutosamplerPort` | String |
| dispense_volume | `HistoricalSampleResults->StartTags->DispenseVolume` | Int32 |
| follow_with_retain | `HistoricalSampleResults->StartTags->FollowWithRetain` | Boolean |
| retain_volume | `HistoricalSampleResults->StartTags->RetainVolume` | Double |
| retain_count | `HistoricalSampleResults->RetainCount` | Int32 |

**Sample Information Tags:**

| Payload Field | OPC Tag Path | Data Type |
|--------------|-------------|-----------|
| sample_id | `HistoricalSampleResults->StartTags->SampleInformation->SampleID` | String |
| batch_id | `HistoricalSampleResults->StartTags->SampleInformation->BatchID` | String |
| vessel_id | `HistoricalSampleResults->StartTags->SampleInformation->VesselID` | String |
| cell_type | `HistoricalSampleResults->StartTags->SampleInformation->CellType` | String |
| pre_dilution_multiplier | `HistoricalSampleResults->StartTags->SampleInformation->PreDilutionMultiplier` | Double |
| vessel_temperature | `HistoricalSampleResults->StartTags->SampleInformation->VesselTemperature` | Double |
| vessel_pressure | `HistoricalSampleResults->StartTags->SampleInformation->VesselPressure` | Double |
| sparging_o2_pct | `HistoricalSampleResults->StartTags->SampleInformation->SpargingO2` | Double |

**Module Selection Tags:**

| Payload Field | OPC Tag Path | Data Type |
|--------------|-------------|-----------|
| module_cdv_active | `HistoricalSampleResults->StartTags->ModuleInformation->Modules->CDV` | Boolean |
| module_chemistry_active | `HistoricalSampleResults->StartTags->ModuleInformation->Modules->Chemistry` | Boolean |
| module_gas_active | `HistoricalSampleResults->StartTags->ModuleInformation->Modules->Gas` | Boolean |
| module_osmo_active | `HistoricalSampleResults->StartTags->ModuleInformation->Modules->Osmo` | Boolean |
| chemistry_dilution | `HistoricalSampleResults->StartTags->ModuleInformation->ChemistryDilutionRatio` | String |
| cdv_dilution | `HistoricalSampleResults->StartTags->ModuleInformation->CellDensityDilutionRatio` | String |
| cdv_inspection | `HistoricalSampleResults->StartTags->ModuleInformation->CellInspection` | String |

**Result Tags (per analyte):**

| Analyte | Result Tag | Unit Tag | Error Status Tag |
|---------|-----------|----------|-----------------|
| pH | `...->Gas->pH->Result` | `...->Gas->pH->Units` | `...->Gas->pH->ErrorStatus` |
| pCO₂ | `...->Gas->pCO2->Result` | `...->Gas->pCO2->Units` | `...->Gas->pCO2->ErrorStatus` |
| pO₂ | `...->Gas->pO2->Result` | `...->Gas->pO2->Units` | `...->Gas->pO2->ErrorStatus` |
| Na⁺ | `...->Chem->Na->Result` | `...->Chem->Na->Units` | `...->Chem->Na->ErrorStatus` |
| K⁺ | `...->Chem->K->Result` | `...->Chem->K->Units` | `...->Chem->K->ErrorStatus` |
| Ca²⁺ | `...->Chem->Ca->Result` | `...->Chem->Ca->Units` | `...->Chem->Ca->ErrorStatus` |
| NH₄⁺ | `...->Chem->NH4->Result` | `...->Chem->NH4->Units` | `...->Chem->NH4->ErrorStatus` |
| Gln | `...->Chem->Gln->Result` | `...->Chem->Gln->Units` | `...->Chem->Gln->ErrorStatus` |
| Glu | `...->Chem->Glu->Result` | `...->Chem->Glu->Units` | `...->Chem->Glu->ErrorStatus` |
| Gluc | `...->Chem->Gluc->Result` | `...->Chem->Gluc->Units` | `...->Chem->Gluc->ErrorStatus` |
| Lac | `...->Chem->Lac->Result` | `...->Chem->Lac->Units` | `...->Chem->Lac->ErrorStatus` |
| Osmo | `...->Osmo->Result` | `...->Osmo->Units` | `...->Osmo->ErrorStatus` |

All result tag paths above are prefixed with `HistoricalSampleResults`. Result data types are Double; Unit types are String; ErrorStatus types are String.

**CDV Result Tags:**

| Field | OPC Tag Path | Data Type |
|-------|-------------|-----------|
| total_density | `HistoricalSampleResults->CellDensity->TotalDensity` | Double |
| total_density_units | `HistoricalSampleResults->CellDensity->TotalDensityUnits` | String |
| viable_density | `HistoricalSampleResults->CellDensity->ViableDensity` | Double |
| viable_density_units | `HistoricalSampleResults->CellDensity->ViableDensityUnits` | String |
| viability | `HistoricalSampleResults->CellDensity->Viability` | Double |
| avg_live_diameter | `HistoricalSampleResults->CellDensity->AvgLiveDiameter` | Double |
| live_std_deviation | `HistoricalSampleResults->CellDensity->LiveStdDeviation` | Double |
| total_cell_count | `HistoricalSampleResults->CellDensity->TotalCellCount` | Int32 |
| total_live_count | `HistoricalSampleResults->CellDensity->TotalLiveCount` | Int32 |
| good_image_count | `HistoricalSampleResults->CellDensity->GoodImageCount` | Int32 |

**Calculated Result Tags:**

| Field | OPC Tag Path | Data Type |
|-------|-------------|-----------|
| co2_saturation | `HistoricalSampleResults->CalculatedResults->CO2Saturation` | Double |
| o2_saturation | `HistoricalSampleResults->CalculatedResults->O2Saturation` | Double |
| hco3 | `HistoricalSampleResults->CalculatedResults->HCO3` | Double |
| ph_corrected | `HistoricalSampleResults->CalculatedResults->pHCorrected` | Double |
| pco2_corrected | `HistoricalSampleResults->CalculatedResults->pCO2Corrected` | Double |
| po2_corrected | `HistoricalSampleResults->CalculatedResults->pO2Corrected` | Double |

**Per-Analyte Range Tags (available for each analyte):**

| Range Field | Tag Suffix | Data Type | Notes |
|------------|-----------|-----------|-------|
| Lower measurement limit | `Ranges->{analyte}->LowerLimit` | Double | |
| Upper measurement limit | `Ranges->{analyte}->UpperLimit` | Double | |
| Correlation offset intercept | `Ranges->{analyte}->OffsetIntercept` | Double | 0 = no offset applied |
| Correlation offset multiplier | `Ranges->{analyte}->OffsetMultiplier` | Double | 1 = no offset applied |

**Flowtime Tags:**

| Module | OPC Tag Path | Data Type |
|--------|-------------|-----------|
| pH/Gas | `HistoricalSampleResults->Gas->FlowTimeData->FlowTime` | Double |
| Chemistry | `HistoricalSampleResults->Chem->FlowTimeData->FlowTime` | Double |
| CDV | `HistoricalSampleResults->CellDensity->FlowTimeData->FlowTime` | Double |

**General Errors Tag:**

| Field | OPC Tag Path | Data Type | Notes |
|-------|-------------|-----------|-------|
| errors | `HistoricalSampleResults->Errors` | String | General error string if sample error occurred |

---

### 4.2 QC.Analyzed

**OPC source:** `QCResults` object tags.

**Trigger:** Ignition Event Streams detects a change in `QCResults->SampleTime`.

**OPC Tag Mapping — Start Tags:**

| Payload Field | OPC Tag Path | Data Type |
|--------------|-------------|-----------|
| sample_time | `QCResults->SampleTime` | DateTime |
| timestamp | `QCResults->TimeStamp` | DateTime |
| operator | `QCResults->StartTags->Operator` | String |
| qc_lot | `QCResults->StartTags->LotNumber` | String |
| qc_level | `QCResults->StartTags->Level` | String |
| qc_expiration | `QCResults->StartTags->ExpirationDate` | DateTime |

**QC Result Tags:** Same per-analyte pattern as `HistoricalSampleResults` — `QCResults->Gas->pH->Result`, `QCResults->Chem->Gluc->Result`, etc. (Double). Unit tags use `Single` type (note: different from sample results which use `String`). Error status tags are `String`.

**QC Range Tags:** Same per-analyte `LowerLimit`, `UpperLimit`, `OffsetIntercept`, `OffsetMultiplier` pattern under `QCResults->StartTags->Ranges->`.

**CDV QC Tags:** `QCResults->CellDensity->TotalDensity` (Double), `QCResults->CellDensity->GoodImageCount` (Int32), `QCResults->CellDensity->Units` (Single), `QCResults->CellDensity->ErrorStatus` (String).

**Derived field — `overall_pass`:** Not available as an OPC tag. Ignition must compute this by comparing each result value against its corresponding `LowerLimit` and `UpperLimit` range tags, and checking each `ErrorStatus` tag.

**Derived field — `qc_type`:** Not directly available. May be inferred from the lot number format or from the source of initiation (command tag vs. schedule vs. manual).

---

### 4.3 Consumable.Changed (State-Derived Event)

**OPC source:** Consumable status object tags. Ignition derives these events by polling and detecting changes in `LotNumber` or `Installed` tags.

**Consolidation note:** Rather than separate events per consumable type, the canonical event is `Consumable.Changed` with a `consumable_type` discriminator, since the derivation mechanism is identical across all consumable types.

**Consumable Status Tag Sets:**

| Consumable | Tag Root | Fields Available |
|-----------|----------|-----------------|
| Chemistry MicroSensor Card | `ChemCard` | ExpirationDate, Expired, Hydrated, InstallationDate, Installed, LotNumber, SamplesRemaining |
| pH/Gas MicroSensor Card | `GasCard` | ExpirationDate, Expired, Hydrated, InstallationDate, Installed, LotNumber, SamplesRemaining |
| Chemistry Calibrator Pack | `ChemPackStatus` | Empty, ExpirationDate, Expired, FluidRemaining, InstallationDate, Installed, LotNumber, SamplesRemaining, SamplesRemainingPercent |
| pH/Gas Calibrator Pack | `GasPackStatus` | Empty, ExpirationDate, Expired, FluidRemaining, InstallationDate, Installed, LotNumber, SamplesRemaining, SamplesRemainingPercent |
| CDV Pack | `CDVPackStatus` | Empty, ExpirationDate, Expired, FluidRemaining, InstallationDate, Installed, LotNumber, SamplesRemaining, SamplesRemainingPercent |
| Chemistry QC Pack | `ChemQCPackStatus` | Empty, ExpirationDate, Expired, FluidRemaining, InstallationDate, Installed, LotNumber, SamplesRemaining, SamplesRemainingPercent |
| pH/Gas QC Pack | `GasQCPackStatus` | Empty, ExpirationDate, Expired, FluidRemaining, InstallationDate, Installed, LotNumber, SamplesRemaining, SamplesRemainingPercent |
| ESM Pack | `ESMPackStatus` | Empty, ExpirationDate, Expired, FluidRemaining, InstallationDate, Installed, LotNumber, SamplesRemaining, SamplesRemainingPercent |

All tag paths are prefixed with `OPCSystemObjects->`.

**Derivation logic:** Subscribe to or poll each `LotNumber` tag. When the value changes, read the full tag set for that consumable and publish:

| Payload Field | Source |
|--------------|--------|
| consumable_type | Derived from tag root (e.g., `ChemCard` → `chemistry_microsensor_card`) |
| lot_number | `{root}->LotNumber` |
| installation_date | `{root}->InstallationDate` |
| expiration_date | `{root}->ExpirationDate` |
| installed | `{root}->Installed` |
| hydrated | `{root}->Hydrated` (MicroSensor Cards only) |
| samples_remaining | `{root}->SamplesRemaining` |
| analyzer_id | `Settings->AnalyzerID` |

**Not available via OPC:** Operator who performed the change. Available only in the Maintenance Log CSV.

**Osmometer status:** `OPCSystemObjects->OsmoState->CleanTubes` (Int32) tracks available clean tubes. `OPCSystemObjects->OsmoState->CalibrationStatus` (String) tracks calibration state. Tube change can be inferred when `CleanTubes` increases.

---

### 4.4 Calibration.StatusChanged (State-Derived Event)

**OPC source:** Per-parameter calibration status tags.

**Available tags:**

| Module | Tag Path Pattern | Data Type |
|--------|-----------------|-----------|
| pH/Gas | `DP_GasCal->GasCal->GasCal->{param}->CalibrationStatus` | String |
| Chemistry | `DP_ChemCal->ChemCal->ChemCal->{param}->CalibrationStatus` | String |
| CDV | `DP_CdvCal->CdvCal->CdvCal->CalibrationStatus` | String |
| Osmometer | `DP_OsmoCal->OsmoCal->OsmoCal->CalibrationStatus` | String |
| Osmometer (alt) | `OsmoState->CalibrationStatus` | String |

Values: `Calibrated` or `Uncalibrated`.

**Derivation logic:** Subscribe to calibration status tags. When any value changes (e.g., `Uncalibrated` → `Calibrated` or vice versa), publish event with affected parameters and new status.

**Limitations:** The OPC server does not expose calibration slope data, 2-point calibration details, 1-point per-sample calibration results, auto vs. manual distinction, or consecutive failure counts. These are only available in the Calibration Log CSV. The OPC tag tells you *if* calibration succeeded, not *how*.

---

### 4.5 Parameter.AlertChanged (State-Derived Event)

**OPC source:** Per-parameter Alert and Warning boolean tags.

**Available tags (per parameter):**

| Field | Tag Path | Data Type | Notes |
|-------|----------|-----------|-------|
| Alert | `Parameters->{param}->Alert` | Boolean | False = available, True = unavailable |
| Warning | `Parameters->{param}->Warning` | Boolean | False = no warning, True = warning |

Parameters: pH, pCO2, pO2, Na, K, Ca, NH4, Gln, Glu, Gluc, Lac, Osmo, CDV.

**This subsumes QC Lockout, Calibration Failure, and Consumable Expiration alerts.** The OPC server doesn't distinguish *why* a parameter alert is active. It only reports the boolean state. The reason must be inferred from context (recent QC failure, calibration status, consumable status) or obtained from the Error Log CSV.

**Derivation logic:** Subscribe to Alert/Warning tags. When any boolean transitions (False → True or True → False), publish event with the affected parameter, new alert state, and a snapshot of potentially causal state (current calibration status, recent QC results, consumable status).

**Dependency chain reference (from IFU — relevant when interpreting cascading alerts):**
- If Glu locked → Gln unavailable
- If Na⁺ or K⁺ locked → NH₄⁺ unavailable
- If pH locked → pCO₂ unavailable

---

### 4.6 Instrument.Error

**OPC source:** Not available as discrete event tags. Per-parameter `ErrorStatus` strings appear in `HistoricalSampleResults` and `QCResults` only in the context of a specific analysis.

**General errors within an analysis** are available via `HistoricalSampleResults->Errors` (String), but this is limited to sample-analysis-time errors.

**For comprehensive error logging:** Requires CSV Error Log export from `C:\Export\Errors*.csv`.

---

### 4.7 User.Session

**OPC source:** Not available. Login/logout events are recorded only in the Audit Log.

**Partial workaround:** The `HistoricalSampleResults->StartTags->Operator` and `QCResults->StartTags->Operator` tags identify the logged-in operator at the time of each analysis. This provides per-sample operator attribution without discrete session events.

---

## 5. Instrument State Tags (Non-Event, Reference Data)

These tags provide static or slow-changing reference information useful for enrichment.

| Tag Path | Data Type | Description |
|----------|-----------|-------------|
| `Settings->AnalyzerID` | String | Instrument identifier |
| `Settings->Location` | String | Instrument location |
| `SoftwareVersion->SoftwareVersion` | String | Current firmware |
| `CoreHeartbeat->UpTime` | String | Time since last restart |
| `DateTime->DateTime` | DateTime | Current instrument UTC time |
| `TimeSync->LastSync->LocalTimeZone` | String | Local time zone name |
| `TimeSync->LastSync->LocalTZOffset` | String | UTC offset |
| `ActiveTasks->Task` | String | Currently active tasks |
| `ScheduledTasks->Task` | String | Scheduled tasks with due times |
| `SampleTypeNames->SampleTypeNames` | String | Configured sample type names |
| `SampleTypes->SampleTypes` | String | Configured sample type details |
| `ParametersConfiguration->{param}->Units` | String | Configured unit per parameter |
| `Modules->InstalledUnits->CDV` | String | Module connection status (`Ready`) |
| `Modules->InstalledUnits->Osmo` | String | Module connection status |
| `Modules->InstalledUnits->Autosampler` | String | Module connection status |
| `Modules->InstalledUnits->ESM` | String | Module connection status |
| `Modules->InstalledUnits->RetainCollector` | String | Module connection status |
| `Resources->Wells->CDVWell` | String | Well state (`WellState.Clear`) |
| `Resources->Wells->ChemistryWell` | String | Well state |
| `Resources->Wells->WasteWell` | String | Well state |

---

## 6. Consumer Contract Summary

| Consumer | Events Subscribed | Filter / Transformation |
|----------|-------------------|------------------------|
| Manufacturing Record (PS) | `Sample.Analyzed` | Filter to pH (± pCO₂). Enrich with MES batch context. Write to batch record. |
| MSAT / Process Development | `Sample.Analyzed` | Filter to VCD, viability, cell diameter. Enrich with process step context. Feed trend analysis. |
| QC / QA | `Sample.Analyzed`, `QC.Analyzed`, `Parameter.AlertChanged`, `Calibration.StatusChanged` | Full panel. Full instrument state context. Supports disposition and audit readiness. |
| Data Lake / Historian | All events | No filter. Full payload. Historization and ad-hoc analysis. |
| Genealogy Graph | All events | Builds traceability links: Sample → active consumable lots, bracketing QC, calibration state, operator. |
| LIMS | `Sample.Analyzed`, `QC.Analyzed` | Routes analytical results for review, verification, and release workflows. |

All filtering and enrichment occurs in Ignition, not at the publisher.

---

## 7. Genealogy Relationships

A single `Sample.Analyzed` event connects to the following instrument-state context. With the OPC state-tag model, these relationships are established by snapshotting consumable state tags *at the time of each sample analysis* rather than relying on discrete installation events:

- **Active MicroSensor Cards** — snapshot `ChemCard->LotNumber` + `GasCard->LotNumber` at sample time
- **Active Calibrator Packs** — snapshot `ChemPackStatus->LotNumber` + `GasPackStatus->LotNumber`
- **Active Reagent/CDV Packs** — snapshot `CDVPackStatus->LotNumber`
- **Active QC Packs** — snapshot `ChemQCPackStatus->LotNumber` + `GasQCPackStatus->LotNumber`
- **Calibration state** — snapshot all `CalibrationStatus` tags at sample time
- **Parameter availability** — snapshot all `Alert`/`Warning` booleans at sample time
- **Operator** — from `HistoricalSampleResults->StartTags->Operator`
- **Bracketing QC** — most recent `QC.Analyzed` event per module/level before this sample

**Implementation note:** Ignition should enrich each `Sample.Analyzed` event with a snapshot of consumable and calibration state tags at publish time. This creates a self-contained traceability record without requiring the consumer to reconstruct instrument state from a sequence of change events.

---

## 8. Implementation Phases

### Phase 1 (MVP): Analytical Results via OPC

- Ignition OPC UA client subscribes to FLEX2 Bridge OPC server
- Event Streams trigger on `HistoricalSampleResults->SampleTime` and `QCResults->SampleTime` changes
- On change, read full result tag trees, assemble and publish `Sample.Analyzed` and `QC.Analyzed`
- Include consumable state snapshot (lot numbers, calibration status) in each event for genealogy
- Configure consumer flows for PS (pH filter), MSAT (VCD filter), LIMS, and full-panel consumers

### Phase 2: Instrument State Events via OPC Change Detection

- Subscribe to consumable `LotNumber` tags; publish `Consumable.Changed` on transition
- Subscribe to `CalibrationStatus` tags; publish `Calibration.StatusChanged` on transition
- Subscribe to `Alert`/`Warning` booleans; publish `Parameter.AlertChanged` on transition

### Phase 3: Audit and Error Events via CSV Log Export

- File watcher on `C:\Export` for Audit Log and Error Log CSVs
- Parse and publish `User.Session` and `Instrument.Error` events
- Enrich calibration events with slope data from Calibration Log CSV

---

## 9. Open Items

| # | Item | Owner | Status |
|---|------|-------|--------|
| 1 | Obtain OPC license from Nova Biomedical | Procurement | Not started |
| 2 | Confirm OPC UA endpoint accessibility from Ignition server network segment | Engineering | Not started |
| 3 | Validate Ignition Event Streams tag-change subscription behavior against FLEX2 OPC UA server — confirm that Event Streams can reliably detect `SampleTime` updates and trigger event assembly | Engineering | Not started |
| 4 | Determine optimal subscription/polling configuration for state-derived events (consumable lots, calibration status, alert booleans) — balance between responsiveness and OPC server load | Engineering | Not started |
| 5 | Confirm whether the `HistoricalSampleResults` tags update atomically (all tags update together) or if there's a race condition risk where Ignition reads a partial result set during tag updates | Engineering | Not started |
| 6 | Validate parameter dependency chains in QC Lockout against live instrument behavior | QC | Not started |
| 7 | Establish Analyzer ID naming convention for multi-instrument sites | All | Not started |
| 8 | Define LIMS inbound interface for FLEX2 results | LIMS | Not started |
| 9 | Determine if Phase 1 consumable state snapshot at sample time provides sufficient genealogy fidelity, or if Phase 2 discrete change events are needed for audit | QC / QA | Not started |
| 10 | Evaluate OPC Server Version 4.1 Calculation Items feature — can custom calculated tags (Section 7 of PN 60644) simplify Ignition logic for derived fields like `overall_pass`? | Engineering | Not started |

---

## 10. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1 | 2026-04-01 | — | Initial draft. Event set derived from IFU PN 57960 Rev F. OPC tag mapping pending. |
| 0.2 | 2026-04-01 | — | Incorporated OPC Manual PN 60644 Rev B. Complete OPC tag mapping added. Restructured events around state-vs-event architecture. Identified CSV log supplementary path. Added implementation phases. Consolidated consumable events. |
| 0.3 | 2026-04-01 | — | Replaced HighByte with Ignition Event Streams as integration layer. Removed ERP/LIMS-specific system references; uses generic MES and LIMS designations. Added LIMS as explicit consumer. |
