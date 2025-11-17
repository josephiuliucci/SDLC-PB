# User Requirements Specification (URS)

## 1. Introduction

This User Requirements Specification (URS) defines the user‑level requirements for the deployment of **HiveMQ**, an MQTT broker, in a regulated manufacturing environment.  The broker will serve as a secure messaging layer to shuttle data between process equipment (sensors, instruments, controllers) and higher‑level systems such as electronic batch record (EBR) systems, historians and analytics platforms.  HiveMQ will not be the system of record; it will provide reliable, secure message transport and support regulatory compliance in accordance with FDA 21 CFR Part 11 and EU Annex 11.

## 2. Intended Use and Scope

- **Purpose**: To implement a secure, high‑availability MQTT broker for transporting production and quality data in real time.  The broker will not control process parameters or make product‑quality decisions; it will merely route messages.
- **Scope**: This URS covers requirements for selection, configuration and operation of the HiveMQ broker, including integration with sensors, edge devices, enterprise systems and user interfaces.  It includes functional, performance, security and compliance requirements.
- **Exclusions**: Process control software (e.g., autoclave or bioreactor control), external EBR or database systems, and IT infrastructure components (servers, networks) are not covered by this URS except where integration interfaces are required.

## 3. Stakeholders

| Role | Responsibilities |
|---|---|
| **Manufacturing Operations** | Publish process values and events to the broker; subscribe to instructions and alerts; require reliable and timely message delivery. |
| **Quality Assurance (QA)** | Ensure data integrity, auditability and compliance with regulatory requirements; review audit logs and message trails. |
| **IT/Systems Administrator** | Install, configure and maintain the broker; manage user accounts and certificates; monitor system health and perform backups. |
| **Data Analytics and Reporting** | Consume data from the broker for analytics, dashboards and reporting; require consistent, complete data streams. |
| **Regulatory/Compliance** | Verify that the system meets 21 CFR Part 11 and Annex 11 requirements; oversee validation and periodic reviews. |

## 4. Functional Requirements

### 4.1 Messaging and Connectivity

1. **Publish/Subscribe Support** – The broker shall support MQTT v3.1.1 and v5 protocols with Quality of Service (QoS) levels 0, 1 and 2 to ensure at‑most‑once, at‑least‑once or exactly‑once delivery depending on device needs.
2. **Topic Management** – It shall allow creation and management of hierarchical topics representing equipment, sensors, batches and process events.  Wildcard subscriptions shall be supported for groups of devices.
3. **Retained Messages** – The broker shall support retained messages for topics where the last known value must be immediately available to new subscribers (e.g., current temperature, set point).
4. **Session Persistence** – It shall maintain persistent sessions for clients that request it, including queued messages for offline clients.
5. **Message Routing** – The broker shall route messages between publishers and subscribers within configurable clusters and support bridging to external brokers or enterprise systems when necessary.

### 4.2 Security and Authentication

1. **Encrypted Transport** – All client‑broker communication shall use TLS/mTLS (TLS 1.2 or higher) to ensure confidentiality and integrity.
2. **Client Authentication** – The broker shall support certificate‑based authentication (mTLS) and/or username/password authentication with configurable policies (minimum password complexity, expiration, lockout).
3. **Role‑Based Access Control (RBAC)** – The system shall allow assignment of roles/permissions to clients and users specifying which topics can be published or subscribed to.  Unauthorized publish/subscribe attempts shall be rejected and logged.
4. **Audit Logging** – The broker shall record a tamper‑evident audit trail of all connection attempts, authentications, topic subscriptions, message publications, configuration changes and administrative actions with timestamps and user identity.  Logs shall be retained in accordance with data‑retention policies and accessible for regulatory inspection.
5. **Access to Configuration** – Administrative interfaces (e.g., Web UI or REST API) shall require multi‑factor authentication and role‑based permissions to prevent unauthorized changes.

### 4.3 Availability and Performance

1. **High Availability** – The broker shall operate as a cluster with automatic fail‑over.  In the event of a node failure, connected clients shall reconnect automatically without data loss.
2. **Scalability** – The system shall support horizontal scaling to handle increasing numbers of clients and messages without degradation.  Target throughput shall be defined during design (e.g., messages/second and number of concurrent connections).
3. **Latency** – Message latency from publisher to subscriber under normal load shall meet process requirements (e.g., < 500 ms for control signals, < 2 s for monitoring data).  Latency metrics shall be monitored.
4. **Monitoring and Alerting** – The broker shall expose health metrics (CPU, memory, message rates, queue sizes) and integrate with the enterprise monitoring solution to generate alerts when thresholds are exceeded or nodes become unavailable.
5. **Backup and Recovery** – There shall be mechanisms to back up broker configuration, persistent session data and audit logs.  Recovery procedures shall be documented and tested.

### 4.4 Compliance and Data Integrity

1. **21 CFR Part 11 Compliance** – The system shall provide features necessary to support Part 11 and Annex 11 compliance: role‑based authentication, timestamped audit trails, tamper‑evident logs, and the ability to export records in human‑readable and electronic form.
2. **Data Integrity (ALCOA Principles)** – Messages transmitted via the broker shall be attributable (capturing identity of publisher), legible, contemporaneous, original and accurate.  Each sensor value and event shall include a timestamp from the source device and be logged by the broker.
3. **Time Synchronization** – The broker shall maintain accurate system time and support Network Time Protocol (NTP) synchronization to ensure consistency of timestamps across audit logs and messages.
4. **Configuration Change Control** – All configuration changes (topic creation, ACL updates, cluster adjustments) shall be made under a formal change control process, with approval and documented testing before promotion to production.

### 4.5 Integration

1. **External System Interfaces** – The broker shall integrate with downstream systems (EBR, historians, analytics) through standard MQTT bridging, REST APIs or connector plugins.  Integration points must be documented and version‑controlled.
2. **Client Libraries** – Client libraries compatible with target languages (e.g., Java, Python, C#, Node.js) shall be available to device manufacturers and application developers.
3. **Message Transformation (Optional)** – If necessary, the broker shall support transformation or enrichment of messages (e.g., adding metadata) via plugin or middleware without altering original payloads for regulatory traceability.

### 4.6 Usability and Support

1. **Management Interface** – A user‑friendly administrative interface shall allow authorized personnel to configure topics, users, roles and monitor system status.
2. **Documentation and Training** – Vendor documentation for installation, configuration, API usage and troubleshooting shall be provided.  Training shall be delivered to administrators and users as part of system deployment.
3. **Support and Maintenance** – Vendor shall provide support services (e.g., updates, patches, security advisories) and a mechanism to report incidents or request assistance.

## 5. Non‑Functional Requirements

- **Maintainability** – The system shall be maintainable with minimal downtime.  Configuration and version upgrades shall be performed in controlled environments and rolled out according to change control procedures.
- **Portability** – The broker shall be deployable on both on‑premise servers and in cloud environments consistent with corporate IT policies.
- **Interoperability** – The broker shall support interoperability standards (MQTT, TLS, REST) to avoid vendor lock‑in and ensure compatibility with a range of devices and software.
- **Privacy** – If any personal data are transmitted (e.g., patient‑generated data in clinical trials), data privacy controls compliant with relevant regulations (HIPAA, GDPR) shall be implemented.  Data shall be encrypted in transit and, where applicable, pseudonymized or anonymized.

## 6. Regulatory References

- FDA 21 CFR Part 11 and EU GMP Annex 11 (electronic records and signatures).
- FDA **Computer Software Assurance for Production and Quality System Software** guidance (2025) for risk‑based assurance and classification of software【318686843355289†L768-L795】.
- ISO/IEC 27001:2022 and SOC 2 Type 2 controls for information security, processing integrity, availability, confidentiality and privacy【28586329347191†screenshot】.

## 7. Acceptance Criteria

The system will be accepted when:

- All functional and non‑functional requirements specified herein are satisfied and traced through validation testing.
- Validation documentation (Validation Plan, Risk Assessment, IQ/OQ/PQ protocols, Validation Summary Report) demonstrates that the broker is fit for intended use.
- SOPs for operation, maintenance, backup and incident response are in place and personnel are trained.
- Regulatory and quality assurance teams approve the system based on review of documentation and testing.
