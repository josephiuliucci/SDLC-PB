# Software Development Life‑Cycle (SDLC) Plan – Modern SCADA Platform (Ignition 8.3.x)

## Purpose

This Software Development Life‑Cycle (SDLC) plan defines activities and deliverables for procuring, configuring and validating a modern SCADA platform based on Inductive Automation’s Ignition 8.3.x.  It follows the user’s defined phases (Concept, Procure, Plan, Build, Configure, Design, Test, Implement and Release) and aligns with Computer Software Assurance (CSA) principles and GAMP 5 guidance.  The SCADA platform will be classified as GAMP Category 4 (configurable off‑the‑shelf software) because it primarily acquires, stores and displays data without directly controlling critical process parameters; custom scripts that influence process control will be treated as Category 5.

## Scope

This SDLC applies to the acquisition, installation and configuration of the Ignition gateway and core modules (Vision, Perspective, Power Historian, Alarm Notification, Reporting, MQTT, API modules), along with their integration with PLCs, databases and enterprise systems.  It does not cover the validation of custom scripts or MES modules; those will be addressed separately.  The plan ensures that the platform operates as intended, supports regulatory compliance and provides a reliable foundation for plant‑wide monitoring and data management.

## Applicability and Risk Classification

The FDA’s 2025 Computer Software Assurance guidance states that software used for collecting or recording data without directly impacting production is generally **not high process risk**【318686843355289†L768-L795】.  Ignition’s core functionality (data acquisition, visualization and historian) falls into this category.  Therefore, the platform will undergo Category 4 validation, which requires defined user requirements, functional and configuration specifications, installation and operational qualification (IQ/OQ) and limited scripted testing【796508146311493†L400-L420】.  Custom scripting or modules that can adjust process parameters or perform acceptance decisions will be categorized as Category 5 and will require detailed design specifications, code review and more extensive testing.

## SDLC Phases

### 1. Concept

**Objectives:** Define the business need and high‑level scope for the SCADA platform.

- Identify the need to improve visibility, alarm management and data collection across production lines.
- Determine the scope of equipment, sensors and systems to be integrated (e.g., PLCs, SQL databases, historians).
- Assess that the SCADA platform will serve as a data visualization and recording tool, not the system of record.
- Evaluate initial risk classification and note that the platform is Category 4; any custom control logic may become Category 5.

**Deliverables:**

- Business case and project charter.
- High‑level URS outlining key capabilities (referencing the URS document).
- Preliminary risk assessment identifying high‑risk functions (none for base platform).

**Roles:** Project sponsor, control engineer, compliance manager.

### 2. Procure

**Objectives:** Select the appropriate SCADA software and vendor; ensure vendor qualification.

- Evaluate potential SCADA platforms and select Inductive Ignition 8.3.x based on features such as cross‑platform deployment, unlimited clients and tags【417854487715557†L31-L43】【417854487715557†L66-L70】, high‑performance historian【906305396111596†L26-L40】, improved store‑and‑forward【646938752162582†L218-L231】【906305396111596†L53-L90】, and security features (TLS, federated identity, secrets management)【417854487715557†L86-L100】【906305396111596†L192-L243】.
- Request and review vendor documentation and certifications (e.g., ISO 27001, SOC 2) to leverage vendor evidence for CSA.
- Perform vendor audit/qualification to ensure quality systems and support meet GxP requirements.
- Document procurement justification and vendor selection rationale.

**Deliverables:**

- Vendor assessment report and qualification summary.
- Approved purchase order and licensing agreement.
- Vendor documentation and certificates on file.

**Roles:** Procurement specialist, compliance manager, IT manager.

### 3. Plan

**Objectives:** Define detailed tasks, resources and timelines for implementing the SCADA platform.

- Develop a project plan with milestones for installation, configuration, testing and deployment.
- Identify resources (personnel, servers, network infrastructure) required for each phase.
- Define training plan for users and administrators.
- Refine risk assessment; confirm Category 4 classification for base platform and identify any modules or scripts requiring Category 5 treatment.
- Define validation strategy, including which functions will be tested via scripted versus unscripted testing per CSA.

**Deliverables:**

- Detailed project schedule and resource matrix.
- Training plan and user roles/responsibilities matrix.
- Risk management plan and validation strategy.

**Roles:** Project manager, SCADA administrator, QA lead, IT infrastructure lead.

### 4. Build

**Objectives:** Assemble and configure the hardware and software infrastructure for the SCADA platform.

- Provision and qualify servers (physical or virtual) or containers to host the Ignition gateway.  Ensure cross‑platform requirements (Windows/Linux/macOS) are met【417854487715557†L31-L43】.
- Install operating systems, databases and support services according to IT standards.
- Set up network segmentation, firewalls and VPNs to secure communications.
- Prepare development, test and production environments.  If using Docker or Kubernetes, obtain pre‑built Ignition containers and confirm quick launch options【987429409847858†L107-L174】.
- Document hardware and operating system installation, including serial numbers and versions.

**Deliverables:**

- Hardware and OS installation qualification records (part of IQ).
- Network architecture diagram.
- Environment specifications for development, QA and production.

**Roles:** IT/infrastructure team, SCADA administrator.

### 5. Configure (Category 4 functions)

**Objectives:** Configure the SCADA platform according to the URS and functional specifications.

- Install the Ignition gateway and required modules (OPC UA, Perspective, Vision, Power Historian, Event Streams, Alarm Notification, Reporting, MQTT modules).
- Create and configure device connections to PLCs and databases using built‑in drivers【417854487715557†L22-L29】.
- Configure store‑and‑forward settings for buffering and load management【646938752162582†L218-L231】【906305396111596†L53-L90】.
- Set up projects, tags, and tag providers.  Utilize the Power Historian for time‑series data storage and metadata modeling【906305396111596†L26-L40】.
- Design HMI screens and dashboards in Perspective and/or Vision using drag‑and‑drop components and drawing tools【906305396111596†L95-L117】.
- Configure alarm setpoints, priorities, notification pipelines (e.g., SMS, email, WhatsApp, Twilio Voice)【906305396111596†L225-L233】 and alarm aggregation settings【987429409847858†L204-L210】.
- Establish security settings: enable TLS 1.2/1.3, integrate with Active Directory or single sign‑on providers, configure role‑based access control and audit logging【417854487715557†L86-L100】.
- Set up secrets management to external vault if available【906305396111596†L192-L243】.
- Define backup and redundancy settings, enabling automatic failover【417854487715557†L146-L152】.
- Leverage file‑based configuration and integrate with Git for source control【987429409847858†L107-L174】.

**Deliverables:**

- Configuration specification capturing settings for connections, tags, alarms, security and modules.
- Change management records for configuration revisions.
- Updated risk assessment to ensure no new high‑risk functions introduced.

**Roles:** SCADA administrator, control engineer, security officer.

### 6. Design (Category 5 functions only)

**Note:** This phase applies only if the project includes custom scripts, modules or templates that perform control logic or decision‑making.  Otherwise, it may be skipped.

**Objectives:** Develop detailed design specifications for custom functionality.

- Identify scripts or modules that will adjust process parameters or perform acceptance decisions; classify them as high process risk【318686843355289†L768-L795】.
- Document functional and design specifications, including pseudocode, algorithms, flow diagrams and acceptance criteria.
- Review and approve design documents with stakeholders.

**Deliverables:**

- Functional specification and design specification for each custom component.
- Design review and approval records.

**Roles:** Control engineer, software developer, QA lead.

### 7. Test

**Objectives:** Verify that the installed and configured SCADA platform meets the user requirements through a combination of scripted and unscripted testing.

- **Installation Qualification (IQ):** Verify that hardware, OS, databases and Ignition software are installed as specified.  Capture software versions, module versions and license details.
- **Operational Qualification (OQ):** Test core functions against the URS and configuration specification.  Sample unscripted tests can be used for low‑risk features (e.g., navigation, dashboard creation) while scripted tests will confirm critical functions such as alarm notification, historian performance, security authentication and audit trail.
- **Performance Qualification (PQ):** Challenge the system under anticipated load (devices, tags, users) to confirm performance, reliability and failover capabilities.
- Validate alarm functions including escalation and notifications (SMS, email, voice)【906305396111596†L225-L233】.
- Validate historian operation: data capture, querying, metadata association【906305396111596†L26-L40】.
- Verify store‑and‑forward during network interruptions【646938752162582†L218-L231】【906305396111596†L53-L90】.
- Test the Event Streams module for publishing and subscribing event data【906305396111596†L53-L90】.
- Test security settings: TLS connections, user authentication, RBAC enforcement, secrets management【417854487715557†L86-L100】【906305396111596†L192-L243】.
- Confirm backup, redundancy and restore procedures【417854487715557†L146-L152】.

**Deliverables:**

- IQ, OQ and PQ protocols and executed test reports with evidence.
- Test summary report documenting any deviations and resolutions.
- Updated traceability matrix linking URS requirements to test cases.

**Roles:** QA test lead, control engineer, SCADA administrator.

### 8. Implement

**Objectives:** Deploy the validated SCADA system into the production environment, migrate configuration and provide training.

- Promote configuration and projects from the test environment to production using controlled change management processes.
- Conduct user training for operators, engineers, administrators and support staff.
- Migrate historical data if necessary and verify data integrity post‑migration.
- Ensure disaster recovery procedures (backup/restore) are operational.
- Establish monitoring and logging; integrate with enterprise monitoring tools for event alerts and system health.

**Deliverables:**

- Production deployment records and change control approvals.
- Training attendance and competency records.
- Go‑live readiness checklist and sign‑off.

**Roles:** Project manager, SCADA administrator, QA lead, IT support.

### 9. Release and Maintenance

**Objectives:** Manage ongoing support, changes and continuous improvement.

- Transition to operational support; maintain a log of support issues and resolutions.
- Conduct periodic review of audit trails and security settings to ensure compliance.
- Plan and implement software updates, patch management and module upgrades through formal change control.
- Periodically review risk assessment; if new high‑risk features are added, perform additional validation.
- Monitor vendor communications for security advisories and best practices.

**Deliverables:**

- Maintenance and support plan.
- Change control records for updates and patches.
- Periodic review reports and audit logs.

**Roles:** SCADA administrator, IT support, compliance manager.

## Integration with CSA Practices

This SDLC plan incorporates risk‑based CSA principles.  Activities focus on critical functions that could impact product quality or compliance.  For low‑risk features (navigation, dashboards, simple reports), exploratory testing and vendor evidence may suffice.  High‑risk features such as alarms, historian, store‑and‑forward, security and custom scripts require scripted testing and detailed documentation.  All electronic records and signatures must comply with 21 CFR Part 11, and audit trails must never be overwritten【318686843355289†L768-L795】.  The project team will document intended use, risk assessment, testing rationale and results to demonstrate that the system operates as intended.
