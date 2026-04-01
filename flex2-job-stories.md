# Nova Biomedical Bioprofile FLEX2 — Job Stories

**Companion to:** FLEX2 Publisher Data Contract Specification v0.3  
**Date:** 2026-04-01  
**Format:** Job stories — *"When [situation], I need [capability], so that [outcome]."*

---

## 1. Overarching Story

**When** a Bioprofile FLEX2 analyzer completes any analytical, quality control, or maintenance activity, **I need** that activity to be published as a structured, self-contained event to the site event bus, **so that** downstream systems (MES, LIMS, historian, genealogy graph) can consume instrument data without direct integration to the FLEX2, without manual transcription, and without loss of traceability context.

---

## 2. Data Contract Stories

**When** multiple teams (PS, MSAT, QC, data science) need different subsets of the same FLEX2 measurement, **I need** a single publisher contract that emits the full analytical panel for every analysis, **so that** each consumer can filter to the parameters they care about without requiring the instrument to know who is consuming or what they need.

**When** I am defining what fields belong in a FLEX2 event payload, **I need** the contract to reflect what the instrument actually produces, not what any single consumer has requested, **so that** the contract is stable across changing consumer requirements and new consumers can onboard without modifying the publisher.

**When** the FLEX2 OPC server exposes state tags rather than discrete events, **I need** the data contract to define how state transitions are detected and converted into events by Ignition Event Streams, **so that** the derivation logic is documented, testable, and consistent across instrument instances.

---

## 3. Sample Analyzed Event Stories

### 3.1 Event-Level Stories

**When** a FLEX2 sample analysis completes (manual, load-and-go, 96-well plate, ESM, or OLS), **I need** a sample analyzed event to be published to the MQTT broker, **so that** the result is available to all downstream consumers within seconds of completion without anyone manually exporting or transcribing data.

**When** a sample analyzed event is published, **I need** it to contain the full set of results for every module that was active during the analysis, **so that** consumers receiving the event don't need to make a second request to the instrument for missing parameters.

**When** Ignition detects that the sample time has changed on the OPC server, **I need** it to read the complete result tag tree and assemble a single atomic event, **so that** no consumer receives a partial result set where some analyte values belong to the current sample and others to the previous one.

### 3.2 Sample Identification Payload Stories

**When** a QC analyst reviews a FLEX2 result in the LIMS, **I need** the event to carry the sample ID, **so that** the result can be matched to the LIMS sample record without manual lookup.

**When** a process engineer is trending in-process analytics across a production batch, **I need** the event to carry the batch ID, **so that** results can be grouped and plotted by batch without cross-referencing a separate log.

**When** a facility runs multiple bioreactors in parallel, **I need** the event to carry the vessel ID, **so that** results are attributed to the correct reactor and not conflated across vessels.

**When** a deviation investigation requires identifying who performed a specific analysis, **I need** the event to carry the operator name, **so that** the individual responsible can be identified from the event record without pulling the FLEX2 audit log.

**When** a cell culture process uses multiple cell lines or clones, **I need** the event to carry the cell type, **so that** analytical results can be stratified by cell line for comparative analysis.

### 3.3 Sampling Context Payload Stories

**When** I need to understand how a sample was introduced to the analyzer, **I need** the event to carry the sample source (manual, load-and-go, 96-well, ESM, OLS), **so that** I can distinguish between sampling modes when investigating variability or comparing manual vs. automated workflows.

**When** samples are analyzed from a load-and-go carousel or 96-well plate, **I need** the event to carry the tray location, **so that** I can correlate a result to a specific well or cup position for troubleshooting positional effects.

**When** samples are analyzed via the Online Autosampler from a connected bioreactor, **I need** the event to carry the autosampler port, **so that** I can link the result to the specific RSM port and reactor connection.

**When** a sample type has been pre-configured with specific module selections, dilution ratios, and ranges, **I need** the event to carry the sample type name, **so that** I can verify the correct analytical profile was applied and filter results by sample type.

### 3.4 Module Configuration Payload Stories

**When** not every module is run for every sample (e.g., chemistry only, no CDV), **I need** the event to indicate which modules were active during the analysis, **so that** consumers can distinguish between "this parameter was not measured" and "this parameter was measured and returned no result."

**When** a sample is run with a non-default dilution ratio, **I need** the event to carry the chemistry and CDV dilution ratios, **so that** the dilution context travels with the result and downstream calculations can account for it.

**When** different CDV analysis configurations are used for different cell types, **I need** the event to carry the cell inspection type, **so that** I know which cell inspection profile was applied and can compare results only across analyses using the same configuration.

### 3.5 Process Condition Payload Stories

**When** a process engineer needs to correlate analytical results with bioreactor operating conditions, **I need** the event to carry the vessel temperature, vessel pressure, and sparging oxygen percentage, **so that** the conditions reported by the operator at sampling time are captured alongside the measurement.

**When** a sample has been pre-diluted before presentation to the analyzer, **I need** the event to carry the pre-dilution multiplier, **so that** the reported result can be interpreted in the context of the actual sample concentration.

### 3.6 Analyte Result Payload Stories

**When** a QC analyst needs to verify that a pH measurement is within the process specification, **I need** the event to carry the pH result with its unit of measure and error status, **so that** the value, its unit, and whether it was measured successfully are all present in a single record.

**When** the manufacturing record requires a pH value for the batch record, **I need** the event to carry the pH result even when other parameters in the same analysis are in error, **so that** a chemistry module error doesn't suppress a valid pH/gas module result.

**When** a parameter is locked out due to QC failure, **I need** the event to carry an error status indicating the lockout rather than silently omitting the parameter, **so that** a missing value is distinguishable from a value that could not be reported due to instrument state.

*The following stories apply to each analyte individually (pH, pCO₂, pO₂, Na⁺, K⁺, Ca²⁺, NH₄⁺, Gln, Glu, Gluc, Lac, osmolality) and to the CDV results (total density, viable density, viability, cell diameter):*

**When** a measured value is reported, **I need** it to carry the configured unit of measure, **so that** downstream systems can handle unit conversion if the instrument is configured differently than the consumer expects.

**When** a measured value is reported, **I need** it to carry any applied correlation offset (intercept and multiplier), **so that** I can determine whether the raw or corrected value was reported and reproduce the correction if needed.

**When** a measured value is reported, **I need** it to carry the lower and upper measurement range limits that were active for this analysis, **so that** I can assess whether the result was within the instrument's validated operating range.

### 3.7 CDV-Specific Payload Stories

**When** a cell density analysis is reported, **I need** the event to carry total density, viable density, viability percentage, and average live cell diameter, **so that** I have the complete picture of culture health in a single event.

**When** I need to assess the confidence of a CDV measurement, **I need** the event to carry the good image count, total cell count, total live count, and live standard deviation, **so that** I can evaluate measurement quality and flag results with low image counts or high variability.

### 3.8 Calculated Result Payload Stories

**When** a process scientist needs temperature-corrected gas values, **I need** the event to carry the corrected pH, corrected pCO₂, and corrected pO₂, **so that** the instrument-calculated corrections are available without downstream recalculation.

**When** dissolved gas analysis is used for process monitoring, **I need** the event to carry CO₂ saturation, O₂ saturation, and bicarbonate, **so that** the derived saturation and bicarbonate values travel with the direct measurements.

### 3.9 Diagnostic Payload Stories

**When** I need to investigate whether an analytical module was performing normally during a specific analysis, **I need** the event to carry per-module flow time data, **so that** deviations in fluidic behavior can be correlated with suspect results.

**When** a sample analysis encounters an error, **I need** the event to carry the error description, **so that** the nature of the failure is recorded alongside the partial result set.

### 3.10 Sample Retain Payload Stories

**When** a sample retain is collected alongside an analysis, **I need** the event to indicate that a retain was collected and to carry the retain volume and count, **so that** the existence and volume of retained material is linked to the analytical result for traceability.

---

## 4. QC Analyzed Event Stories

### 4.1 Event-Level Stories

**When** an onboard or external QC analysis completes on the FLEX2, **I need** a QC analyzed event to be published, **so that** the QC result is available to the LIMS and genealogy graph without manual data entry.

**When** a QC result is reported, **I need** the event to carry the QC material lot number, level, and expiration date, **so that** the QC result is traceable to a specific lot of control material and can be assessed for material validity.

**When** a QC result is reported, **I need** the event to carry the operator who initiated or was logged in during the analysis, **so that** QC accountability is maintained.

### 4.2 QC Result Payload Stories

**When** a QC result is reported, **I need** each analyte to carry the measured value, the configured lower limit, and the configured upper limit, **so that** pass/fail status can be determined and QC ranges are captured at the time of analysis rather than looked up after the fact.

**When** a QC analysis completes, **I need** the event to include whether all parameters passed overall, **so that** downstream systems can quickly determine whether the instrument passed QC without evaluating each parameter individually.

**When** a QC failure occurs and QC Lockout is enabled, **I need** the QC result event to precede any resulting parameter alert event in the event stream, **so that** consumers can establish causality between the QC failure and the parameter lockout.

---

## 5. Consumable Changed Event Stories

### 5.1 Event-Level Stories

**When** a MicroSensor Card, Calibrator Cartridge, Reagent Cartridge, QC Cartridge, or CDV pack is installed or replaced on the FLEX2, **I need** a consumable changed event to be published, **so that** the genealogy graph can link all subsequent analytical results to the correct consumable lot.

**When** an auditor asks "which MicroSensor Card was installed when this sample was analyzed?", **I need** the consumable installation to be recorded as an event with the lot number and installation date, **so that** the answer can be retrieved from the event history without inspecting the instrument's maintenance log.

### 5.2 Consumable Payload Stories

**When** a consumable is changed, **I need** the event to carry the lot number and expiration date, **so that** the specific lot of material in use is captured and consumable expiration can be monitored.

**When** a MicroSensor Card is installed, **I need** the event to indicate whether the card has completed hydration, **so that** I know whether the card is ready for use.

**When** a consumable is installed, **I need** the event to carry the number of samples remaining, **so that** consumable depletion can be monitored and replacements planned before the instrument goes offline.

**When** a consumable event is derived from an OPC state tag change, **I need** the previous lot number to be captured from the last known value, **so that** the transition from one lot to another is explicit in the event record.

---

## 6. Calibration Status Changed Event Stories

**When** the calibration status of any FLEX2 parameter changes (uncalibrated to calibrated or calibrated to uncalibrated), **I need** a calibration status changed event to be published, **so that** the genealogy graph can record whether the instrument was calibrated at the time of each sample analysis.

**When** a parameter transitions from calibrated to uncalibrated, **I need** the event to identify which specific parameters lost calibration, **so that** downstream systems can flag any samples analyzed after the calibration loss as potentially suspect.

**When** a calibration event occurs, **I need** to know it happened even though the OPC server does not expose slope data or calibration details, **so that** I can at minimum record that calibration state changed and direct investigators to the calibration log for root cause detail.

---

## 7. Parameter Alert Changed Event Stories

**When** any FLEX2 parameter becomes unavailable (alert activates), **I need** a parameter alert changed event to be published, **so that** downstream systems are immediately aware that subsequent sample results will not include this parameter.

**When** a parameter alert clears, **I need** the event to record the recovery, **so that** the window of unavailability is bounded and auditable.

**When** a parameter alert activates and I cannot determine the cause from the OPC tags alone, **I need** the event to include a snapshot of related state (calibration status, recent QC results, consumable status), **so that** the most likely cause can be inferred without querying multiple data sources.

**When** QC Lockout causes a cascade of parameter dependencies (e.g., glutamate locked out causes glutamine to become unavailable), **I need** alerts for both the directly failed parameter and the dependent parameters to be captured, **so that** the cascade is visible in the event record.

---

## 8. Instrument Error Event Stories

**When** the FLEX2 logs an error, **I need** an instrument error event to be published (via CSV log export), **so that** equipment errors are captured in the site event bus and can trigger maintenance workflows or deviation investigations.

**When** a sample analysis encounters an error, **I need** the per-analyte error status to be included in the sample analyzed event, **so that** analysis-time errors are associated with the specific result they affected.

---

## 9. User Session Event Stories

**When** an operator logs into or out of the FLEX2, **I need** a user session event to be published (via audit log CSV export), **so that** the operator responsible for instrument activity during a time window can be identified independently of per-sample operator attribution.

**When** a sample result carries the operator name, **I need** that name to reflect the user logged in at the time of analysis, **so that** per-sample operator attribution is available even before the user session event pipeline is implemented.

---

## 10. Genealogy and Traceability Stories

**When** an auditor or quality reviewer needs to assess the trustworthiness of a specific FLEX2 result, **I need** the sample analyzed event to carry a snapshot of the instrument state at the time of analysis (active consumable lots, calibration status, parameter alerts), **so that** the question "was this instrument qualified to produce this result at this time?" can be answered from a single event record.

**When** a batch disposition decision depends on FLEX2 analytical results, **I need** the genealogy graph to link each result to the bracketing QC events (most recent passing QC per module and level before the sample), **so that** the validity of the result is supported by QC evidence.

**When** a consumable lot is later found to be defective (e.g., a MicroSensor Card lot recall), **I need** to query the genealogy graph for all sample results produced while that consumable lot was active, **so that** the scope of impact can be determined and affected batches identified.

**When** a QC failure is detected and QC Lockout locks a parameter, **I need** the genealogy graph to identify all samples analyzed between the last passing QC and the lockout event, **so that** the results produced during the window of uncertainty can be reviewed.

**When** the calibration status of a parameter changes to uncalibrated, **I need** to identify the last valid calibration event and all samples analyzed between that calibration and the failure, **so that** the potential impact window is bounded.

---

## 11. Consumer Filtering Stories

**When** the PS team needs only pH for the manufacturing batch record, **I need** a consumer flow that subscribes to sample analyzed events and extracts only the pH result and batch identification fields, **so that** the batch record receives only the data it needs without being overwhelmed by the full 16-parameter panel.

**When** the MSAT team needs viable cell density and viability for process trending, **I need** a consumer flow that subscribes to sample analyzed events and extracts CDV results with vessel and batch context, **so that** trend data is available in near-real-time without manual data pulls.

**When** the QC/QA team needs the complete instrument picture for disposition, **I need** a consumer flow that subscribes to sample analyzed, QC analyzed, parameter alert changed, and calibration status changed events, **so that** they have full visibility into both the analytical result and the instrument state that produced it.

**When** the data science or engineering team needs all data for exploratory analysis, **I need** a consumer flow that subscribes to all events with no filtering, **so that** they can build models and discover patterns without being limited by pre-defined views.

**When** the LIMS needs analytical results for review and verification workflows, **I need** a consumer flow that subscribes to sample analyzed and QC analyzed events and routes them into the appropriate LIMS workflow, **so that** results arrive automatically for analyst review rather than being manually entered.

---

## 12. Architecture and Integration Stories

**When** the FLEX2 OPC server provides state tags that update in-place rather than an event stream, **I need** Ignition Event Streams to detect value transitions and emit discrete events, **so that** the rest of the architecture can consume events without awareness of the underlying OPC polling or subscription mechanics.

**When** the sample result tag tree updates upon sample completion, **I need** Ignition to read all related tags atomically (or to verify atomicity), **so that** no event is published with a mix of current and previous sample data.

**When** multiple FLEX2 instruments are deployed across the site, **I need** each instrument's events to carry its analyzer ID and follow a consistent MQTT topic structure, **so that** consumers can subscribe to a specific instrument or to all instruments via topic wildcards.

**When** the FLEX2 Bridge computer is restarted or loses network connectivity, **I need** the integration to recover gracefully without losing events that occurred during the outage, **so that** the event record remains complete even through infrastructure disruptions.

**When** data that is not available via OPC (calibration slope data, error descriptions, user sessions, maintenance history) is needed for compliance, **I need** a supplementary CSV log export path from the Bridge computer, **so that** the full audit record can be assembled from a combination of OPC-sourced events and log-sourced events.
