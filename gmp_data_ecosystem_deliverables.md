# GMP Data Communication Ecosystem – Categorized Deliverables (Assets & Evidence)

This document lists the **deliverables required to bring an integrated data and information communication ecosystem** (Ignition SCADA, HighByte Intelligence Hub, HiveMQ) into **GMP operation**.

A deliberate distinction is made throughout between:

- **Assets** – tangible systems, configurations, capabilities, or states that must *exist* in the facility
- **Evidence** – records that *demonstrate* those assets were created, controlled, tested, and maintained per approved procedures

Both are required. Evidence alone is never sufficient.

---

## 1. Governance & Scope Definition

### 1.1 Governance Assets

- Defined project and system scope
- Agreed GMP boundary (what is and is not GxP-relevant)
- Defined regulatory posture for the ecosystem

### 1.2 Governance Evidence

- Project Charter
- GMP Scope Definition
- Intended Use Statements (platform-level and use-case-level)
- Regulatory applicability mapping (21 CFR 210/211, Part 11, Annex 11, GAMP 5)
- Validation Strategy / Validation Master Plan

---

## 2. Risk Management & Quality Planning

### 2.1 Risk & Quality Assets

- Risk-based validation approach
- Defined quality oversight model
- Explicit acceptance of residual risk

### 2.2 Risk & Quality Evidence

- GxP Risk Assessment (patient safety, product quality, data integrity)
- Data Integrity Risk Assessment (ALCOA+)
- Cybersecurity risk assessment
- Validation approach justification (CSA-aligned)
- Quality Plan (document control, change control, review cadence)

---

## 3. Requirements & Design

### 3.1 Requirements & Design Assets

- Defined business and quality requirements
- Designed system behaviors and data flows
- Agreed information lifecycle

### 3.2 Requirements & Design Evidence

- User Requirements Specification (URS)
- Functional Specification (FS)
- Configuration / Design Specification (CS/DS)
- Interface specifications (SCADA ↔ HighByte ↔ MQTT consumers)
- MQTT namespace and naming conventions
- HighByte data models and contextualization rules

---

## 4. Architecture & Infrastructure

### 4.1 Infrastructure Assets

- Provisioned servers (physical or virtual) per environment (DEV / TEST / PROD)
- Installed and supported operating systems
- Network connectivity in approved zones
- Storage volumes and file systems
- Time synchronization services
- Backup and restore mechanisms
- Monitoring and alerting services

### 4.2 Infrastructure Evidence

- Logical and physical architecture diagrams
- Server build or provisioning records
- OS installation and version records
- Network and firewall configuration records
- Security hardening confirmation
- Backup configuration and restore test evidence
- Monitoring enablement confirmation

---

## 5. Software Platforms

### 5.1 Software Assets

- Installed Ignition SCADA platform
- Installed HighByte Intelligence Hub
- Installed HiveMQ broker
- Required databases and supporting services

### 5.2 Software Evidence

- Installation records and version inventories
- Module and feature enablement lists
- Configuration exports / backups
- Environment separation confirmation

---

## 6. Data & Integration

### 6.1 Data & Integration Assets

- Live data connections from source systems
- Contextualized and modeled data streams
- Published MQTT topics
- Downstream consumers receiving data

### 6.2 Data & Integration Evidence

- Data flow diagrams
- HighByte pipeline definitions
- MQTT topic hierarchies and QoS definitions
- Interface verification records

---

## 7. Security & Access Control

### 7.1 Security Assets

- Role-based access control implemented
- Secure authentication mechanisms
- Encrypted communications (TLS)
- Centralized identity integration (where applicable)

### 7.2 Security Evidence

- Access control matrices
- User and service account registers
- Certificate and key management records
- Security configuration verification

---

## 8. Audit Trails & Data Integrity

### 8.1 Data Integrity Assets

- Enabled and functioning audit trails
- Tamper-evident data storage
- Time-stamped, attributable records

### 8.2 Data Integrity Evidence

- Audit trail configuration documentation
- Audit trail verification test results
- Data integrity review records

---

## 9. Testing & Qualification

### 9.1 Qualified System Assets

- Installed and configured systems performing as intended
- Use cases executing correctly in the GMP context

### 9.2 Qualification Evidence

- Installation Qualification (IQ) protocols and reports
- Operational Qualification (OQ) protocols and reports
- Performance Qualification (PQ) or use-case verification
- Traceability Matrix (URS → tests)

---

## 10. Procedures & Operations

### 10.1 Operational Assets

- Defined operational and administrative practices
- Supported system lifecycle

### 10.2 Operational Evidence

- SOPs (operation, administration, security, backup)
- Incident and deviation procedures
- Change control procedures

---

## 11. Training & Readiness

### 11.1 Organizational Assets

- Trained and competent personnel
- Defined support and escalation model

### 11.2 Training Evidence

- Training materials
- Training execution records
- Competency confirmation (where required)

---

## 12. GMP Release & Go-Live

### 12.1 GMP-Ready Assets

- System approved for GMP use
- Controlled production environment

### 12.2 GMP Release Evidence

- Validation Summary Report
- Quality approval for GMP release
- Go-live checklist

---

## 13. Operations & Lifecycle Management

### 13.1 Lifecycle Assets

- Monitored and maintained systems
- Sustainable operational posture

### 13.2 Lifecycle Evidence

- Monitoring and alerting records
- Periodic review plans and reports
- Patch and upgrade documentation

---

## 14. Change & Decommissioning

### 14.1 Change & Retirement Assets

- Controlled evolution or retirement of systems

### 14.2 Change & Retirement Evidence

- Change requests and impact assessments
- Revalidation decisions
- Decommissioning plans and execution records

---

## Closing Principle

Every GMP-relevant deliverable exists in **two forms**:

1. **The thing itself** (system, configuration, capability)
2. **The proof** that it was created and controlled correctly

A compliant system requires both. Evidence without assets is fiction; assets without evidence are indefensible.

