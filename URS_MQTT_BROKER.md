# User Requirements Specification (URS) with User Stories for an MQTT Broker

This document combines the original user requirements for deploying the an MQTT broker in a regulated manufacturing environment with corresponding user stories.  Each requirement is paired with a user story that explains, from the stakeholder’s perspective, why the capability is needed and what value it provides.

## 1. Introduction

The MQTT broker will serve as a secure messaging layer to shuttle data between process equipment (sensors, instruments, controllers) and higher‑level systems such as electronic batch record (EBR) systems, historians and analytics platforms.  The MQTT broker is **not** the system of record; it provides reliable, secure message transport and supports regulatory compliance in accordance with FDA 21 CFR Part 11 and EU Annex 11.

## 2. Intended Use and Scope

- **Purpose:** Implement a secure, high‑availability MQTT broker for transporting production and quality data in real time.  The broker does not control process parameters or make product‑quality decisions; it routes messages.

- **Scope:** Selection, configuration and operation of the MQTT broker, including integration with sensors, edge devices, enterprise systems and user interfaces.  Requirements cover functional, performance, security and compliance aspects.  

- **Exclusions:** Process control software (e.g., autoclave or bioreactor control), external EBR, or database systems, and IT infrastructure components (servers, networks) except where integration interfaces are required.

## 3. Stakeholders

| Role | Responsibilities |
|---|---|
| **Manufacturing Operations** | Publish process values and events to the broker; subscribe to instructions and alerts; require reliable and timely message delivery. |
| **Quality Assurance (QA)** | Ensure data integrity, auditability and compliance with regulatory requirements; review audit logs and message trails. |
| **IT/Systems Administrator** | Install, configure and maintain the broker; manage user accounts and certificates; monitor system health and perform backups. |
| **Data Analytics and Reporting** | Consume data from the broker for analytics, dashboards and reporting; require consistent, complete data streams. |
| **Regulatory/Compliance** | Verify that the system meets 21 CFR Part 11 and Annex 11 requirements; oversee validation and periodic reviews. |

## 4. Functional Requirements and User Stories

### 4.1 Messaging and Connectivity

- **Requirement – Publish/Subscribe Support:** The broker shall support MQTT v3.1.1 and v5 protocols with Quality of Service (QoS) levels 0, 1 and 2 to ensure at‑most‑once, at‑least‑once, or exactly‑once delivery depending on device needs.

  **User Story:** *As a device engineer, I need the broker to support MQTT v3.1.1 and v5 with QoS 0–2, so that each sensor or controller can choose the appropriate reliability level for its messages and ensure data is delivered as required.*

- **Requirement – Topic Management:** The broker shall allow creation and management of hierarchical topics representing, but not limited to, business domains, equipment, sensors, batches, and process events.  Wildcard subscriptions shall be supported for groups of devices.

  **User Story:** *As a system administrator, I want to define hierarchical topics with wildcard subscriptions, so that I can organize data streams semantically and let clients subscribe to groups of related topics without maintaining long lists.*

- **Requirement – Retained Messages:** The broker shall support retained messages for topics where the last known value must be immediately available to new subscribers (e.g. current temperature, set point).

  **User Story:** *As an operator, I want the broker to retain the last known value for critical topics, so that when a new subscriber connects it immediately receives the most recent value rather than waiting for the next publish.*  


- **Requirement – Session Persistence:** The broker shall maintain persistent sessions for clients that request it, including queued messages for offline clients.

  **User Story:** *As a field‑device integrator, I need the broker to support persistent sessions and queue messages for clients that reconnect, so that devices in unreliable network conditions can recover without losing important data.*  

- **Requirement – Message Routing:** The broker shall route messages between publishers and subscribers within configurable clusters and support bridging to external brokers or enterprise systems when necessary.

  **User Story:** *As an enterprise architect, I want the broker to route messages across clusters and bridge to external brokers or enterprise systems, so that the messaging layer can scale and integrate with existing IT infrastructure.*

### 4.2 Security and Authentication

- **Requirement – Encrypted Transport:** All client‑broker communication shall use TLS/mTLS (TLS 1.2 or higher) to ensure confidentiality and integrity.

  **User Story:** *As a QA specialist, I need all broker communications to use TLS/mTLS, so that sensitive production and quality data are protected in transit and meet 21 CFR Part 11’s integrity requirements.*  

- **Requirement – Client Authentication:** The broker shall support certificate‑based authentication (mTLS) and/or username/password authentication with configurable policies (minimum password complexity, expiration, lock‑out).

  **User Story:** *As an IT administrator, I want to authenticate clients via certificates or username/password with configurable policies, so that only authorized devices and users can connect and we can enforce password complexity and lock‑out rules.*  

- **Requirement – Role‑Based Access Control (RBAC):** The system shall allow assignment of roles/permissions to clients and users specifying which topics can be published or subscribed to.  Unauthorized publish/subscribe attempts shall be rejected and logged.

  **User Story:** *As a security officer, I need to assign publish/subscribe permissions by topic and role, so that clients can only access data they are entitled to and unauthorized attempts are blocked and logged.*  

- **Requirement – Audit Logging:** The broker shall record a tamper‑evident audit trail of all connection attempts, authentications, topic subscriptions, message publications, configuration changes and administrative actions with timestamps and user identity.  Logs shall be retained in accordance with data‑retention policies and accessible for regulatory inspection.

  **User Story:** *As a compliance manager, I want the broker to record a tamper‑evident audit trail of connections, authentications, subscriptions, publications, configuration changes and administrative actions with timestamps and user identity, so that we can reconstruct who did what and when in accordance with Part 11 and Annex 11.*  

- **Requirement – Access to Configuration:** Administrative interfaces (e.g. Web UI or REST API) shall require multi‑factor authentication and role‑based permissions to prevent unauthorized changes.

  **User Story:** *As a systems administrator, I need administrative interfaces to require multi‑factor authentication and role‑based permissions, so that only authorized personnel can change the broker’s configuration.*  

### 4.3 Availability and Performance

- **Requirement – High Availability:** The broker shall operate as a cluster with automatic fail‑over.  In the event of a node failure, connected clients shall reconnect automatically without data loss.

  **User Story:** *As an operations manager, I want the broker to run as a cluster with automatic fail‑over, so that if a node fails the clients reconnect without data loss and production is not interrupted.*  

- **Requirement – Scalability:** The system shall support horizontal scaling to handle increasing numbers of clients and messages without degradation.  Target throughput shall be defined during design (e.g. messages/second and number of concurrent connections).

  **User Story:** *As a planner, I need the messaging layer to scale horizontally to handle more clients and higher message rates, so that the system can grow with our manufacturing operations without degradation.*  

- **Requirement – Latency:** Message latency from publisher to subscriber under normal load shall meet process requirements (e.g. < 500 ms for control signals, < 2 s for monitoring data).  Latency metrics shall be monitored.

  **User Story:** *As a process engineer, I need message latency to remain below defined thresholds (< 500 ms for control signals, < 2 s for monitoring data), so that real‑time decisions and alerts remain timely.*  

- **Requirement – Monitoring and Alerting:** The broker shall expose health metrics (CPU, memory, message rates, queue sizes) and integrate with the enterprise monitoring solution to generate alerts when thresholds are exceeded or nodes become unavailable.

  **User Story:** *As an IT operator, I want the broker to expose health metrics and integrate with our monitoring tools, so that we receive alerts when resource utilization is high or nodes become unavailable.*  

- **Requirement – Backup and Recovery:** There shall be mechanisms to back up broker configuration, persistent session data and audit logs.  Recovery procedures shall be documented and tested.

  **User Story:** *As an administrator, I need the ability to back up broker configuration, persistent session data and audit logs, and to test recovery procedures, so that we can restore service quickly after a failure.*  

### 4.4 Compliance and Data Integrity

- **Requirement – 21 CFR Part 11 Compliance:** The system shall provide features necessary to support Part 11 and Annex 11 compliance: role‑based authentication, timestamped audit trails, tamper‑evident logs, and the ability to export records in human‑readable and electronic form.

  **User Story:** *As a quality‑systems lead, I need the broker to provide role‑based authentication, timestamped audit trails, tamper‑evident logs and the ability to export records, so that electronic records handled by the broker meet Part 11 and Annex 11 requirements.*  

- **Requirement – Data Integrity (ALCOA Principles):** Messages transmitted via the broker shall be attributable (capturing identity of publisher), legible, contemporaneous, original and accurate.  Each sensor value and event shall include a timestamp from the source device and be logged by the broker.

  **User Story:** *As a QA reviewer, I need each message to include a publisher identity and source timestamp and be logged by the broker, so that our data is attributable, legible, contemporaneous, original and accurate.*  

- **Requirement – Time Synchronization:** The broker shall maintain accurate system time and support Network Time Protocol (NTP) synchronization to ensure consistency of timestamps across audit logs and messages.

  **User Story:** *As a systems integrator, I want the broker to synchronize its clock via NTP, so that timestamps across audit logs and messages are consistent and auditable.*  

- **Requirement – Configuration Change Control:** All configuration changes (topic creation, ACL updates, cluster adjustments) shall be made under a formal change control process, with approval and documented testing before promotion to production.

  **User Story:** *As a compliance auditor, I need all configuration changes to follow formal change control with approvals and testing, so that unapproved changes cannot be promoted to production.*  

### 4.5 Integration

- **Requirement – External System Interfaces:** The broker shall integrate with downstream systems (EBR, historians, analytics) through standard MQTT bridging, REST APIs or connector plugins.  Integration points must be documented and version‑controlled.

  **User Story:** *As an architect, I want the broker to integrate with EBR systems, historians and analytics platforms via standard MQTT bridging, REST APIs or connectors, so that production and quality data flows seamlessly into downstream systems.*  

- **Requirement – Client Libraries:** Client libraries compatible with target languages (e.g. Java, Python, C#, Node.js) shall be available to device manufacturers and application developers.

  **User Story:** *As a device manufacturer, I need official client libraries for Java, Python, C#, Node.js, etc., so that our developers can quickly integrate sensors and controllers with the broker.*  

- **Requirement – Message Transformation (Optional):** If necessary, the broker shall support transformation or enrichment of messages (e.g. adding metadata) via plugin or middleware without altering original payloads for regulatory traceability.

  **User Story:** *As an integration developer, I may need to enrich or transform messages via plugins or middleware, without altering the original payload, so that metadata can be added for downstream processing while preserving raw data for audit purposes.*  

### 4.6 Usability and Support

- **Requirement – Management Interface:** A user‑friendly administrative interface shall allow authorized personnel to configure topics, users, roles and monitor system status.

  **User Story:** *As an administrator, I want a user‑friendly interface to configure topics, users and roles and to monitor system status, so that routine tasks do not require specialist knowledge.*  

- **Requirement – Documentation and Training:** Vendor documentation for installation, configuration, API usage and troubleshooting shall be provided.  Training shall be delivered to administrators and users as part of system deployment.

  **User Story:** *As a project manager, I need comprehensive vendor documentation and training for installation, configuration, API usage and troubleshooting, so that our team can deploy and maintain the system correctly.*  

- **Requirement – Support and Maintenance:** Vendor shall provide support services (e.g. updates, patches, security advisories) and a mechanism to report incidents or request assistance.

  **User Story:** *As a support lead, I need access to vendor updates, patches, security advisories and a support channel for incidents, so that we can keep the broker secure and up to date.*  

## 5. Non‑Functional Requirements and User Stories

- **Maintainability:** The system shall be maintainable with minimal downtime.  Configuration and version upgrades shall be performed in controlled environments and rolled out according to change control procedures.

  **User Story:** *As an IT manager, I need the broker to be maintainable with minimal downtime and allow controlled upgrades, so that maintenance activities don’t disrupt manufacturing.*  

- **Portability:** The broker shall be deployable on both on‑premise servers and in cloud environments consistent with corporate IT policies.

  **User Story:** *As an infrastructure architect, I want the broker to run on both on‑premise and cloud environments, so that we can deploy it according to corporate IT policies.*  

- **Interoperability:** The broker shall support interoperability standards (MQTT, TLS, REST) to avoid vendor lock‑in and ensure compatibility with a range of devices and software.

  **User Story:** *As a strategist, I need the broker to support open standards (MQTT, TLS, REST), so that we are not locked into a single vendor and can integrate a variety of devices and applications.*  

- **Privacy:** If any personal data are transmitted (e.g. patient‑generated data in clinical trials), data privacy controls compliant with relevant regulations (HIPAA, GDPR) shall be implemented.  Data shall be encrypted in transit and, where applicable, pseudonymized or anonymized.

  **User Story:** *As a data‑privacy officer, I need personal data transmitted through the broker to be encrypted and, where applicable, pseudonymized or anonymized, so that we comply with regulations such as GDPR.*  

## 6. Regulatory References

- FDA 21 CFR Part 11 and EU GMP Annex 11 (electronic records and signatures).
- FDA **Computer Software Assurance for Production and Quality System Software** guidance (2025) for risk‑based assurance and classification of software.
- ISO/IEC 27001:2022 and SOC 2 Type 2 controls for information security, processing integrity, availability, confidentiality and privacy.

## 7. Acceptance Criteria

The system will be accepted when:

- All functional and non‑functional requirements specified herein are satisfied and traced through validation testing.
- Validation documentation (Validation Plan, Risk Assessment, IQ/OQ/PQ protocols, Validation Summary Report) demonstrates that the broker is fit for intended use.
- SOPs for operation, maintenance, backup and incident response are in place and personnel are trained.
- Regulatory and quality assurance teams approve the system based on review of documentation and testing.