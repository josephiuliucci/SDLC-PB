# Software Development Life Cycle Plan – MQTT Broker (HiveMQ)

## 1. Purpose and Scope

This plan defines the software development life cycle (SDLC) activities for deploying an MQTT broker based on HiveMQ in a regulated manufacturing environment.  The broker will be used to transport operational data between industrial equipment, sensors and enterprise systems.  It is **not** a system of record or source of truth and does not directly control process parameters.  According to FDA’s Computer Software Assurance guidance, software used only to collect and record data for monitoring or to support quality ‑system processes, without directly impacting production, is considered **not high process risk**.  Therefore, this MQTT broker is treated as a GAMP Category 4 configurable product.  Instances (e.g., specific topic hierarchies) may need separate intended ‑use assessments if they control high‑risk processes.

## 2. SDLC Phases

The following phases apply to the broker’s life cycle.  Each phase produces documentation and evidence commensurate with the low process risk classification.

### 2.1 Concept

* **Define business need and intended use** – Identify why a broker is needed (e.g., to decouple publishers and subscribers and provide reliable, secure message transport).  Document high‑level requirements in a User Requirements Specification (URS).
* **Identify stakeholders** – Include IT/OT integrators, process engineers, data scientists, QA/Compliance, and system administrators.
* **Classify intended use and process risk** – Confirm the broker collects and routes data without controlling process parameters.  Since it does not adjust process conditions or automatically decide product acceptability, it falls under “not high process risk”.
* **Decide GAMP category** – Because HiveMQ is a configurable, off‑the‑shelf product with standard interfaces and functions, classify it as GAMP Category 4.  Custom scripts or extensions would require Category 5 treatment.
* **High‑level risk assessment** – Identify potential hazards (e.g., message loss, security breaches) and planned mitigations.  Define whether separate instances or topics used for critical process control require additional review.

### 2.2 Procure

* **Evaluate vendors** – Perform supplier qualification.  Review HiveMQ’s ISO 27001:2022 and SOC 2 Type 2 certifications, which provide third‑party assurance of security, availability and processing integrity.  Assess vendor maturity, support, and responsiveness.
* **Select product** – Choose the edition (e.g., self‑hosted or cloud) and licensing model that meets business and GxP requirements.  Verify that the product supports TLS/mTLS, role‑based access control and audit logging.
* **Document procurement justification** – Record why HiveMQ was selected, including alignment with URS and risk mitigation.  Keep vendor audit reports and certification evidence for inspectors.

### 2.3 Plan

* **Project planning** – Define project scope, deliverables, schedule, resource assignments and budget.  Establish roles and responsibilities for validation activities.
* **Validation plan** – Develop a Computer Software Assurance (CSA) plan describing assurance activities.  Identify required specifications (URS, configuration specification), testing activities (IQ/OQ/PQ), traceability matrices, and acceptance criteria.  Emphasize risk‑based testing per FDA CSA: focus on higher‑risk features and leverage vendor evidence for low‑risk functions.
* **Training plan** – Plan training for administrators and users on broker configuration, security and monitoring.

### 2.4 Build

* **Infrastructure preparation** – Provision hardware or virtual machines, operating systems and network interfaces.  Ensure segregation of environments (development, test, production) and maintain backups.
* **Installation** – Install HiveMQ software using vendor instructions.  Conduct system hardening and apply security patches.  Document installation steps for Installation Qualification (IQ).
* **Baseline environment** – Record versions of operating system, dependencies and HiveMQ.  Create rollback procedures.

### 2.5 Configure (Category 4)

* **Configuration specification** – Translate URS into a configuration specification.  Document topics, quality of service (QoS) levels, retained message settings, persistent sessions and bridging.  For Category 4 systems, the configuration specification details which functions are enabled and how they are set【796508146311493†L413-L433】.  Link each configuration item back to the URS requirement.
* **Security configuration** – Configure TLS/mTLS, client authentication (certificates or user/password), and role‑based access control.  Ensure audit logging is enabled to capture connections, publishes, subscriptions and admin actions.
* **High availability** – Set up clustering or replication to ensure fail‑over.  Document node roles and heartbeat monitoring.
* **Integration configuration** – Define connections to external systems (MQTT bridges, REST endpoints, analytics platforms) and any transformation or filtering required for message payloads.  Ensure that integration functions merely route or format data; any automation that adjusts process parameters would require additional risk evaluation.

### 2.6 Design (Category 5 not used)

HiveMQ is a configurable product; therefore, no custom software design is expected.  If custom extensions, scripts or plug‑ins are developed (GAMP Category 5), produce a software design specification and module design documents as required.  Each custom component must undergo its own life cycle and risk assessment.

### 2.7 Test

* **Installation Qualification (IQ)** – Verify that the broker and supporting infrastructure are installed according to the installation instructions.  Confirm OS version, file locations, services, and environment variables match the configuration specification.
* **Operational Qualification (OQ)** – Test configured functions against the configuration specification.  Verify that topics are created, QoS levels enforced, TLS works, authentication rejects unauthorized clients, RBAC restricts access appropriately, clustering fails over correctly, and audit logs record actions.  Where applicable, integrate vendor test evidence and perform unscripted/exploratory testing for lower‑risk features to conserve effort.
* **Performance Qualification (PQ)** – Execute end‑to‑end tests demonstrating that the broker meets the URS.  Simulate typical and peak loads, measure message latency and throughput, and ensure reliability.  Validate that the system supports business workflows and meets defined acceptance criteria.
* **Defect management** – Document deviations and corrective actions.  Retest until acceptance criteria are met.

### 2.8 Implement

* **System deployment** – Promote the validated configuration to the production environment.  Execute change control and ensure that the production environment matches the tested configuration.
* **Training and handover** – Train operators and administrators.  Provide documentation on operational procedures, backup/restore and incident response.
* **Go‑live readiness review** – Obtain final approvals from QA/Compliance before activation.

### 2.9 Release and Maintenance

* **Release documentation** – Compile validation summary report, traceability matrix, and sign‑off.  Maintain a baseline of approved configuration and software versions.
* **Change management** – Implement controlled procedures for future changes.  Evaluate the impact of changes on risk classification.  Minor configuration updates may require regression testing; major updates or custom extensions may trigger additional design and test phases.
* **Periodic review** – Conduct periodic reviews of system performance, audit logs and security posture.  Verify continued compliance with GxP regulations and update risk assessments as needed.

## 3. References

* FDA, **Computer Software Assurance for Production and Quality System Software** (2025).  This guidance defines high vs. not high process risk and recommends using risk‑based assurance activities.
* R.D. McDowall, **Understanding and Interpreting the GAMP 5 Life Cycle Models for Software**.  This paper describes the Category 4 life cycle with user requirements, functional and configuration specifications and notes that configurable products are classified as Category 4.
* HiveMQ, **ISO 27001 and SOC 2 Certifications Support GxP Compliance**.  HiveMQ’s ISO 27001:2022 and SOC 2 Type 2 certifications demonstrate security and quality controls necessary for GxP compliance.
