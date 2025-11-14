# Software Development Life‑Cycle (SDLC) and Computer Software Assurance (CSA) for HiveMQ in a 21 CFR regulated environment

## Context

- **Intended use:** The team intends to deploy **HiveMQ** as an MQTT broker to shuttle messages between equipment, sensors and cloud systems.  It will **not** be the system of record (data persists in other systems such as EBR/DB) and does not determine process parameters or acceptance decisions.
- **Regulatory framework:** Under FDA’s 21 CFR Part 11 and EU Annex 11, electronic records and signatures must be trustworthy.  FDA’s 2025 **Computer Software Assurance for Production and Quality System Software** guidance introduces a risk‑based approach.  Software used **directly** in production or the quality system (e.g., controlling process parameters, recording critical data) must be validated, whereas software used as development tools or general record‑keeping is considered **supporting** (indirect) and typically carries lower risk【318686843355289†L515-L545】.  Software used for general business processes (e‑mail, accounting) is outside the scope【318686843355289†L551-L565】.
- **Process risk classification:** The guidance defines **high process risk** functions as those where failure may compromise product safety—examples include maintaining critical process parameters, automatically determining product acceptability, or adjusting parameters based on sensor data【318686843355289†L732-L767】.  Functions that simply collect or transmit data for monitoring and review without controlling the process are considered **not high process risk**【318686843355289†L768-L795】.  The MQTT broker fits this lower‑risk category because it provides data‑transport services and does not decide or adjust process parameters.
- **Vendor evidence:** HiveMQ’s ISO 27001:2022 certification and SOC 2 Type 2 report indicate that the broker is built under an information security management system, with controls over change management, access control, processing integrity and availability【838680406521625†L88-L98】.  HiveMQ’s own compliance framework includes encryption, authentication, audit logging and redundancy; thus its broker provides **encrypted, authenticated messaging with data integrity and availability**【28586329347191†screenshot】.
- **Use in regulated settings:** HiveMQ’s blog presents use cases: 
  - **Real‑time environmental monitoring in GMP manufacturing** – sensors send environmental readings through the secure MQTT broker, which guarantees timestamped, authenticated messages and retains messages for auditability【28586329347191†screenshot】.  Alerts can be triggered when values deviate, enabling immediate action.
  - **Electronic batch record (EBR) collection and audit trails** – equipment publishes process values and events through the broker to the EBR system; built‑in security and audit logging provide a **complete audit trail of all messages**【685230322548853†screenshot】.
  - **Clinical trial IoT data gathering** – patient devices send data via the broker to the sponsor’s database; end‑to‑end encryption, client authentication and SOC 2 privacy controls ensure data privacy and integrity【882851453500482†screenshot】.

Given this context, HiveMQ will be treated as **supporting software, not high process risk**; however, a documented life‑cycle and assurance activities are still necessary to demonstrate control over its deployment and operation.

## High‑level SDLC plan for HiveMQ deployment

The SDLC documentation must demonstrate that the system is selected, installed, configured, tested and maintained in a controlled manner.  The process should follow **gated phases**, where each phase’s outputs are reviewed and approved before proceeding【881540794594959†L80-L83】.  The following plan assumes a mixture of waterfall and agile practices tailored for a commercial off‑the‑shelf (COTS) broker.

### 1 – Initiation and Planning

| Document/Activity | Purpose | Considerations |
|---|---|---|
| **Project Charter & Scope** | Define objectives, intended use (data transport only), high‑level architecture, and scope (e.g., number of topics, security requirements). | Identify that the broker is not a system of record.  Clarify boundaries with EBR/DB systems. |
| **System Assessment / Classification** | Classify the broker as supporting software used to transfer information.  Document that it does not control process parameters, hence the process risk is **not high**【318686843355289†L768-L795】. | Use a risk assessment worksheet to show why the broker’s failure would not directly compromise product safety (e.g., process parameters remain controlled elsewhere; data are independently verified). |
| **Validation Plan / CSA Plan** | Outline the lifecycle activities, roles, documentation deliverables, and risk‑based assurance approach.  Describe use of vendor evidence (ISO 27001, SOC 2) and define testing strategy. | Indicate that unscripted or ad‑hoc testing is appropriate for low‑risk functions【914170562103472†L260-L317】; however, key functions (e.g., security configuration, message routing) will be verified. |
| **Vendor Qualification** | Evaluate HiveMQ as a vendor: review certifications (ISO 27001, SOC 2), security posture, change control, support, and audit reports. | Include results in the qualification report; maintain vendor certificates as evidence. |
| **User Requirements Specification (URS)** | Collect requirements from manufacturing, quality and IT stakeholders.  Requirements should cover message throughput, latency, availability, authentication, encryption, auditing, scalability, retention time, alerting, backup & recovery, integration to existing systems, and compliance features (tamper‑evident logs, role‑based access). | URS must be traceable to functional requirements and tests【881540794594959†L116-L119】.

### 2 – Design and Configuration

| Document/Activity | Purpose | Considerations |
|---|---|---|
| **Functional Specification (FS)** | Translate URS into specific broker functions and configuration: number of clusters, topics and QoS levels; TLS/mTLS configuration; authentication (client certificates, username/password); retention policies; monitoring; logging and audit settings; data‑bridge configuration; disaster recovery (DR). | Document features for **role‑based authentication and tamper‑evident audit trails** because these are required by 21 CFR Part 11 and EU Annex 11【838680406521625†L88-L98】. |
| **Design Specification (DS)** | Provide a design of the infrastructure: network diagrams, cluster sizing, load balancing, firewall rules, certificate management, integration points (e.g., EBR, historian, analytics).  Include fail‑over and high‑availability design. | Because the broker will be part of the regulated architecture, the design must show segregation of environments (DEV/TEST/PROD) and adherence to corporate IT controls. |
| **Requirements Traceability Matrix (RTM)** | Map each URS to the FS/DS and to planned tests.  This document will be used during validation to ensure every requirement is tested【881540794594959†L116-L120】. | For a COTS product, many functional requirements will be satisfied by configuration rather than code changes. |
| **Configuration Specification** | Record actual settings applied during installation (cluster name, port numbers, user roles, access control lists, SSL certificates, bridging rules).  This becomes the baseline for audits. | Documented configuration is necessary for traceability and repeatability during updates or disaster recovery. |
| **Risk Assessment** | Assess each requirement’s failure mode and classify as high/not high process risk.  For instance, a failure to deliver sensor data in real time may delay monitoring but does not change process parameters; thus it is not high process risk【318686843355289†L768-L795】. | The risk assessment determines the level of testing.  High‑risk requirements (e.g., encryption) may warrant more scripted testing. |

### 3 – Implementation and Verification

| Document/Activity | Purpose | Considerations |
|---|---|---|
| **Installation Qualification (IQ)** | Verify installation against the DS and configuration specification.  Activities include verifying OS version, prerequisites (Java runtime, network ports), correct installation of HiveMQ software, applying licenses, and loading TLS certificates.  Results recorded in IQ report. | Use vendor installation guide as reference; attach logs and screenshots as evidence. |
| **Operational Qualification (OQ)** / **CSA Testing** | Demonstrate that the configured broker meets functional requirements under normal and boundary conditions.  Because the broker is supporting and not high process risk, combine **unscripted exploratory testing** with targeted scripted tests:  
  - **Connectivity and Authentication** – attempt connections with valid and invalid credentials; verify certificate handling and rejection of unauthorized clients.  
  - **Encryption** – verify TLS negotiation and that plaintext connections are refused.  
  - **Message Routing and QoS** – publish/subscribe to topics, validate message ordering, duplication, and delivery to multiple subscribers at different QoS levels.  
  - **Retained Messages & Persistence** – confirm messages are retained per policy and recover after node restart.  
  - **Audit Logging** – check that all connection attempts, subscriptions, publications and administrative actions are logged with timestamps and user identity, forming a **complete audit trail of all messages**【685230322548853†screenshot】.  
  - **High Availability & Failover** – simulate node failure, verify cluster fails over and client reconnects automatically.  
  - **Alerting & Monitoring** – verify integration with monitoring tools and that alerts trigger on threshold conditions. | Use risk assessment to decide depth of each test: e.g., encryption and authentication are high‑risk to data integrity and must be scripted; message throughput performance can be exploratory.  Record all test results; unresolved deviations require corrective actions. |
| **Performance Qualification (PQ)** / **User‑acceptance testing** | For regulated systems, PQ ensures the software performs as expected in the actual production environment with typical load.  Perform an end‑to‑end test by sending data from sensors through the broker to the EBR/historian and verifying that data are recorded, alerts trigger correctly, and audit logs capture the events. | Document test cases and results.  PQ may be combined with OQ for low‑risk systems. |
| **Validation Summary Report (VSR)** | Summarize all validation activities, results, deviations and conclusions.  Confirm that the broker meets user requirements and is fit for intended use. | The VSR should refer to vendor certificates (ISO 27001, SOC 2) as supporting evidence for security and change management controls.  Note any limitations or open issues.

### 4 – Release and Maintenance

| Document/Activity | Purpose | Considerations |
|---|---|---|
| **Configuration Management** | Maintain controlled change process for broker configuration.  Document each change request, risk assessment, approval and testing performed.  Ensure that configuration files are version‑controlled and restricted. | Because HiveMQ will be updated periodically, follow vendor’s release notes.  For low‑risk patches (e.g., bug fixes), rely on vendor evidence; for major releases or changes affecting high‑risk functions, execute regression tests. |
| **Standard Operating Procedures (SOPs)** | Develop SOPs for routine activities such as managing user accounts, adding topics, monitoring broker health, reviewing logs, performing backup and restore, and responding to incidents. | SOPs should include steps for capturing audit trail, performing periodic review and escalation procedures. |
| **Training Records** | Provide training to administrators and users on system functionality, security practices and SOPs.  Maintain records to demonstrate that individuals are qualified to perform their tasks. | Include training on recognising and reporting data‑integrity issues (ALCOA principles). |
| **Periodic Review** | Schedule periodic reviews of the broker to ensure continued compliance.  Review logs, access rights, backup procedures, disaster recovery tests and vendor certificates. | Update risk assessment when new functionality or regulatory guidance emerges. |
| **Incident Management** | Define processes for investigating, correcting and documenting incidents (e.g., unexpected downtime, unauthorised access attempts, data‑loss events). | Determine whether an incident triggers re‑validation or a CAPA. |
| **Decommissioning Plan** | When the broker is retired or replaced, define procedures for migrating topics, preserving historical data and audit logs, revoking certificates, and archiving documentation. | Ensure that data retention requirements are met and that no data is lost during migration.

## CSA considerations and justification

FDA’s CSA guidance encourages applying **critical thinking** to focus assurance activities on areas that impact product quality【914170562103472†L260-L317】.  For HiveMQ:

- **Intended use and classification** – The broker is a **supporting** component used for information exchange【318686843355289†L515-L545】.  It is **not high process risk** because failure would not directly compromise process parameters or product safety; instead, failures delay data delivery or cause manual intervention【318686843355289†L768-L795】.
- **Vendor assurance** – HiveMQ’s ISO 27001 and SOC 2 certifications indicate robust security management, change control and processing integrity.  The broker provides encrypted, authenticated messaging and complete audit trails【28586329347191†screenshot】【685230322548853†screenshot】.  Vendor evidence should be leveraged to reduce duplicative testing.
- **Risk‑based testing** – Use a blend of scripted and unscripted testing.  High‑risk requirements (authentication, encryption, audit logging) require scripted tests; lower‑risk features (performance, high throughput) can be explored through ad‑hoc tests【914170562103472†L260-L317】.
- **Documentation** – Maintain traceability from user requirements to test results; keep records (validation plan, risk assessment, IQ/OQ/PQ protocols, VSR) for inspection.  Provide clear evidence that the broker can reliably transfer data, maintain audit trails and support data integrity.

## Actionable examples

### Autoclave scenario (for comparison)

An autoclave’s control software directly maintains sterilization temperature and pressure—its failure could allow unsterile tools into production.  The software is **direct** and **high process risk**【318686843355289†L732-L767】.  Validation would require rigorous scripted testing and may involve review of calibration functions, cycle parameters, alarms, and electronic signatures.  Unlike the broker, the autoclave’s control system is a system of record for sterilization cycles.

### Middleware (HiveMQ broker) example

HiveMQ acts as **middleware** connecting sensors (temperature, humidity, mixer status) to the EBR or data historian.  It **does not generate or alter data**; its failure would delay data transmission but would not change process parameters.  It is therefore supporting software with **not high process risk**【318686843355289†L768-L795】.  CSA activities focus on verifying secure message transport, audit logs, and integration to the EBR.  Vendor certifications provide significant assurance and reduce the extent of in‑house testing.

### Bioreactor control with independent CQA measurement

A bioreactor’s control software maintains pH, dissolved oxygen and temperature.  Even if an independent instrument measures the critical quality attribute (CQA), the control functions still impact product quality; therefore, the software is **direct** and **high process risk**【318686843355289†L744-L766】.  Validation should include scripted tests for control loops and alarms, with unscripted testing for user interface and monitoring functions.  Independent CQA measurement may allow risk‑based reduction in test depth but does not change the classification.

## Conclusion

Deploying HiveMQ as an MQTT broker in a regulated environment requires a documented SDLC and risk‑based computer software assurance to demonstrate control.  By classifying the broker as supporting, not high process risk software, the team can leverage vendor certifications and focus assurance on security, authentication, and message integrity.  Following the outlined lifecycle—initiation, design/configuration, implementation/verification, release and maintenance—provides auditors with a clear trail of evidence that the broker is fit for its intended use and that changes are managed in compliance with FDA’s 21 CFR Part 11 and CSA guidance.
