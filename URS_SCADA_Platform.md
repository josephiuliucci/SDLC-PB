# User Requirements Specification (URS) – Modern SCADA Platform (Ignition 8.3.x)

## Purpose

This document defines the user requirements for a modern Supervisory Control and Data Acquisition (SCADA) platform based on Inductive Automation’s Ignition 8.3.x framework.  The goal is to provide a configurable, modular solution for real‑time monitoring, visualization and data collection across manufacturing equipment, sensors and industrial systems.  Requirements are written to support compliance with 21 CFR Part 11 and follow the FDA’s risk‑based Computer Software Assurance (CSA) guidance.  In this context, the SCADA platform will be treated as GAMP Category 4 (configurable off‑the‑shelf software) because it primarily collects and visualizes data rather than making process control decisions.  Any custom scripts or modules that influence process parameters would be assessed separately as GAMP Category 5.

## Scope and Intended Use

The SCADA platform will serve as a centralized hub for data acquisition, visualization, alarm management, reporting and integration with enterprise systems.  It will acquire data from programmable logic controllers (PLCs), remote I/O devices and industrial sensors via standard protocols such as OPC UA, Modbus, BACnet, Allen‑Bradley, Siemens and other drivers【417854487715557†L22-L29】.  The platform will store real‑time and historical process data, display it through web‑based human–machine interfaces (HMIs), trigger alarms when process conditions deviate from limits, and exchange information with databases and applications.  The system is not the system of record; it does not by itself make final acceptance decisions or maintain batch record integrity.  Pipelines, scripts or models built within the platform that perform calculations, adjustments or process control must undergo separate intended‑use assessment and validation.

## Stakeholders and Roles

| Role | Description |
| --- | --- |
| **Plant Operator** | Monitors process variables via dashboards and HMIs, acknowledges alarms and enters comments. |
| **Control/Automation Engineer** | Configures connectivity to PLCs, develops HMI screens, scripts, alarm logic and data models. |
| **SCADA Administrator** | Installs, configures and maintains the SCADA server, manages user accounts, roles and security certificates. |
| **IT/Infrastructure Team** | Provides and maintains servers, networks, databases and backup systems; manages virtualization and container deployments. |
| **Quality Assurance (QA)** | Reviews compliance with 21 CFR Part 11, ensures audit trail integrity and verifies that electronic records and signatures meet regulatory requirements. |
| **Process Engineer** | Defines dashboards, trending and reports needed for process monitoring and improvement. |
| **Compliance Manager** | Ensures that software categorization, documentation and validation follow CSA principles and GAMP guidance. |

## Functional Requirements and User Stories

### 4.1 Data Acquisition & Connectivity

The following requirements define how the SCADA platform acquires data and connects to industrial devices and databases.  Each requirement is paired with a user story that explains its value.

- **Requirement – PLC and device connectivity:** The system shall connect to major PLCs and industrial devices via OPC UA and vendor drivers such as Modbus, TCP/UDP, BACnet, Allen‑Bradley and Siemens, enabling live data acquisition from plant‑floor equipment【417854487715557†L22-L29】.

  **User Story:** *As a control engineer, I need the SCADA platform to connect to major PLCs and industrial devices via OPC UA and vendor drivers such as Modbus, TCP/UDP, BACnet, Allen‑Bradley and Siemens, so that I can acquire live data from plant‑floor equipment.*

- **Requirement – SQL and data sources:** The system shall connect to any SQL database (MSSQL, Oracle, PostgreSQL, MySQL) and web services to read from and write to enterprise systems【417854487715557†L22-L29】.

  **User Story:** *As a SCADA administrator, I want the platform to connect to any SQL database and web services, so that I can exchange data with enterprise systems.*

- **Requirement – Unlimited tags and connections:** The system shall support unlimited tags, device connections and designer sessions without additional licensing, ensuring scalability【417854487715557†L31-L43】【417854487715557†L66-L70】.

  **User Story:** *As an IT manager, I need the system to support unlimited tags, device connections and designer sessions without extra licensing costs, so that the solution can scale with plant growth.*

- **Requirement – Store and forward:** The system shall buffer data locally when network connectivity is lost and forward it when connectivity is restored, preventing data loss【646938752162582†L218-L231】【906305396111596†L53-L90】.

  **User Story:** *As a network engineer, I want buffered store‑and‑forward functionality that caches data when network connectivity is lost and forwards it once connectivity is restored, so that no data is lost during outages.*

- **Requirement – Event streaming:** The system shall publish data events to external brokers via Kafka or HTTP and provide subscription‑based updates【906305396111596†L53-L90】.

  **User Story:** *As a developer, I need the platform to publish data events to external brokers via Kafka or HTTP, so that other services can consume real‑time event streams.*

### 4.2 Visualization & HMI

This section describes the visualization and human–machine interface capabilities required of the SCADA platform and pairs each requirement with a user story.

- **Requirement – Web‑based responsive HMI:** The system shall provide a mobile‑responsive HMI accessible from modern web browsers on desktop, tablet or mobile devices, enabling remote monitoring【417854487715557†L31-L43】【646938752162582†L257-L269】.

  **User Story:** *As a plant operator, I want to access dashboards from any modern web browser on desktop, tablet or mobile devices, so that I can monitor the process remotely.*

- **Requirement – Desktop HMI:** The system shall support a desktop‑based HMI for legacy systems with full‑screen operator interfaces【646938752162582†L236-L249】.

  **User Story:** *As a control engineer, I need a desktop‑based HMI for legacy systems, so that I can run full‑screen operator interfaces on dedicated workstations.*

- **Requirement – Drawing and animation:** The system shall include drawing tools for creating custom graphics and animating shapes bound to tag values【906305396111596†L95-L117】.

  **User Story:** *As an HMI developer, I want to create custom graphics and animate shapes bound to tag values, so that the operator interface reflects the process accurately.*

- **Requirement – Forms and offline mode:** The system shall allow operators to enter data via forms and work offline when network connectivity is intermittent, synchronizing data when reconnected【906305396111596†L130-L142】.

  **User Story:** *As an operator in the field, I want to enter data via forms and work offline if network connectivity is intermittent, so that I can capture observations and synchronize them later.*

- **Requirement – Report designer:** The system shall provide a report designer with charts and the ability to convert bitmap images to vector graphics for scalable reports【906305396111596†L118-L126】.

  **User Story:** *As a process engineer, I need a report designer with charts and the ability to convert bitmap images to vector for scalable graphics, so that I can create professional reports.*

### 4.3 Historian & Data Storage

This section defines the historian and data‑storage requirements and pairs them with user stories.

- **Requirement – High‑performance historian:** The system shall include a built‑in time‑series historian with zero configuration and high‑performance queries, along with a time‑series API for custom historians【906305396111596†L26-L40】.

  **User Story:** *As a process engineer, I want a built‑in time‑series historian with zero configuration and high‑performance queries, so that I can store and analyze large volumes of process data efficiently.*

- **Requirement – Metadata and modeling:** The system shall capture metadata about equipment and measurements and automatically model data to provide context for historical queries【906305396111596†L26-L40】.

  **User Story:** *As a data modeler, I need the historian to capture metadata about equipment and measurements, so that I can organize and query historical data by context.*

### 4.4 Alarming & Notifications

This section covers requirements for alarm configuration, notification and analysis.

- **Requirement – Alarm configuration and escalation:** The system shall allow configuration of alarms with priorities, setpoints and escalation rules to ensure timely notifications.

  **User Story:** *As a control engineer, I want to configure alarms with priorities, setpoints and escalation rules, so that operators receive timely notifications when process limits are exceeded.*

- **Requirement – Advanced notifications:** The system shall support sending alarm notifications via SMS, email, voice and messaging services (e.g., WhatsApp)【906305396111596†L225-L233】.

  **User Story:** *As an operator, I want to receive alarm notifications via SMS, email, voice and messaging services (e.g., WhatsApp), so that I can respond quickly.*

- **Requirement – Alarm aggregation and analysis:** The system shall provide ISA‑18.2 compliant alarm statistics (count, duration, chattering) and aggregated overviews to evaluate system health【987429409847858†L204-L210】.

  **User Story:** *As a compliance manager, I need ISA‑18.2 compliant alarm statistics (count, duration, chattering) and aggregated overviews, so that I can evaluate system health and performance.*

### 4.5 Reporting & Analytics

This section describes reporting and analytics capabilities.

- **Requirement – Custom reports:** The system shall allow users to define and schedule reports that include trends, tables and images for documenting process performance.

  **User Story:** *As a process engineer, I need to define and schedule reports that include trends, tables and images, so that I can document process performance and share insights.*

- **Requirement – Data export:** The system shall provide the ability to export historical and real‑time data in standard formats (CSV, JSON).

  **User Story:** *As a data analyst, I want to export historical and real‑time data in standard formats (CSV, JSON), so that I can analyze it using external tools.*

- **Requirement – Real‑time dashboards:** The system shall provide configurable dashboards with widgets (gauges, charts, bar graphs) that update in near‑real time.

  **User Story:** *As an operator, I need dashboards with configurable widgets (gauges, charts, bar graphs) that update in near‑real time, so that I can make informed decisions during production.*

### 4.6 Integration & Interoperability

This section outlines integration and interoperability requirements.

- **Requirement – APIs and scripting:** The system shall provide a Python‑based scripting environment and a REST API for automating tasks, integrating with external systems and extending functionality【417854487715557†L46-L63】.

  **User Story:** *As a developer, I want a Python‑based scripting environment and a REST API, so that I can automate tasks, integrate with external systems and extend functionality.*

- **Requirement – MQTT and messaging:** The system shall support publishing and subscribing to MQTT topics and Sparkplug B to integrate with IIoT ecosystems.

  **User Story:** *As an integration specialist, I want the platform to publish and subscribe to MQTT topics and support Sparkplug B, so that I can integrate with IIoT ecosystems.*

- **Requirement – Enterprise system integration:** The system shall provide connectors for ERP, MES, historians and cloud services (AWS, Azure, SAP) to ensure seamless data flow between OT and IT.

  **User Story:** *As an enterprise architect, I need connectors for ERP, MES, historians and cloud services, so that data flows seamlessly between OT and IT.*

- **Requirement – File‑based configuration and source control:** The system shall store project configuration in JSON files and integrate with Git for source control【987429409847858†L107-L174】.

  **User Story:** *As a DevOps engineer, I want project configuration stored in JSON files and accessible via Git, so that I can manage changes through version control.*

### 4.7 Security & Access Control

This section specifies security and access control requirements.

- **Requirement – Encrypted transport:** The system shall use TLS 1.2 or higher to encrypt all communications and ensure data integrity and confidentiality【417854487715557†L86-L100】.

  **User Story:** *As a QA specialist, I need all communications to be encrypted using TLS 1.2 or higher, so that data integrity and confidentiality are preserved.*

- **Requirement – Authentication and SSO:** The system shall integrate with Active Directory and other federated identity providers, supporting multi‑factor authentication and single sign‑on【417854487715557†L86-L100】.

  **User Story:** *As an IT administrator, I want to integrate with Active Directory and other federated identity providers with multi‑factor authentication and SSO, so that user management aligns with corporate policies.*

- **Requirement – Role‑based access control (RBAC):** The system shall allow assignment of roles and permissions at the project, tag and resource levels and enforce them, rejecting unauthorized actions【417854487715557†L86-L100】.

  **User Story:** *As a security officer, I need to assign permissions at the project, tag and resource levels, so that users only access functions and data they are authorized to use.*

- **Requirement – Secrets management:** The system shall manage passwords, certificates and API keys outside configuration files and integrate with external secret vaults【906305396111596†L192-L243】.

  **User Story:** *As a security engineer, I want passwords, certificates and API keys managed outside configuration files and integrated with external secret vaults, so that credentials are protected.*

- **Requirement – Audit logging:** The system shall record a tamper‑evident audit trail of user actions, configuration changes, alarms and report generation and retain logs for regulatory inspection【417854487715557†L86-L100】.

  **User Story:** *As a compliance manager, I need a tamper‑evident audit trail of user actions, configuration changes, alarms and report generation, so that I can reconstruct events and demonstrate compliance.*

### 4.8 Reliability & Availability

This section covers reliability and availability requirements.

- **Requirement – Redundancy and failover:** The system shall support redundant clustering and automatic failover so that operations continue during hardware or software failures【417854487715557†L146-L152】.

  **User Story:** *As an IT manager, I need the SCADA server to support redundant clustering and automatic failover, so that operations continue during hardware or software failures.*

- **Requirement – Store‑and‑forward reliability:** The system shall maintain data integrity during network outages through store‑and‑forward and resume synchronization upon recovery【646938752162582†L218-L231】【906305396111596†L53-L90】.

  **User Story:** *As a network engineer, I need store‑and‑forward to maintain data integrity during network outages and resume synchronization upon recovery, so that no data is lost.*

- **Requirement – Deployment modes:** The system shall allow separate development, test, staging and production environments with distinct resources【987429409847858†L167-L175】.

  **User Story:** *As a DevOps engineer, I want the ability to run separate development, test, staging and production environments with different resources, so that we can safely test changes before release.*

### 4.9 Deployment & Configuration

This section defines deployment and configuration requirements.

- **Requirement – Cross‑platform support:** The system shall run on Windows, Linux and macOS with minimal hardware【417854487715557†L31-L43】.

  **User Story:** *As an infrastructure engineer, I want the platform to run on Windows, Linux and macOS with minimal hardware, so that I can deploy it on any industrial PC, virtual machine or container.*

- **Requirement – Containerization and quick launch:** The system shall provide pre‑built Docker containers and quick‑launch installers for simple installation and upgrades【987429409847858†L107-L174】.

  **User Story:** *As a systems administrator, I need pre‑built Docker containers and quick‑launch installers, so that installation and upgrades are simple and reproducible.*

- **Requirement – Gateway configuration UI:** The system shall provide a user‑friendly gateway interface with integrated search for configuring devices, projects, security and modules【987429409847858†L107-L174】.

  **User Story:** *As an administrator, I want a user‑friendly gateway interface with integrated search, so that I can configure devices, projects, security and modules efficiently.*

- **Requirement – File‑based backup & restore:** The system shall provide the ability to back up configuration and projects via files and restore them across systems for quick recovery and replication.

  **User Story:** *As an IT manager, I want the ability to back up configuration and projects via files and restore them across systems, so that I can quickly recover from failures and replicate environments.*

### 4.10 Scalability & Performance

This section defines scalability and performance requirements.

- **Requirement – High throughput:** The system shall handle thousands of devices and millions of tags while maintaining low latency (e.g. < 500 ms for control updates).

  **User Story:** *As a system architect, I want the SCADA platform to handle thousands of devices and millions of tags while maintaining low latency (< 500 ms for control updates), so that production scales without degradation.*

- **Requirement – Elastic load distribution:** The system shall distribute workloads across multiple servers and containers to support horizontal scaling.

  **User Story:** *As an IT manager, I want to distribute workloads across multiple servers and containers, so that we can scale horizontally as demand increases.*

### 4.11 Compliance & Data Integrity

This section lists compliance and data integrity requirements.

- **Requirement – Electronic records:** The system shall handle electronic records and signatures in compliance with 21 CFR Part 11, ensuring completeness, accuracy and retrievability.

  **User Story:** *As a QA specialist, I need the platform to handle electronic records and signatures in compliance with 21 CFR Part 11, ensuring that records are complete, accurate and readily retrievable.*

- **Requirement – Data integrity (ALCOA principles):** The system shall ensure data is attributable, legible, contemporaneous, original and accurate, with controlled timestamps and versioning.

  **User Story:** *As a QA reviewer, I need data to be attributable, legible, contemporaneous, original and accurate, with controlled timestamps and versioning, so that auditability is maintained.*

- **Requirement – Regulatory audit trail:** The system shall provide a tamper‑evident audit trail that records who performed each action and when, supporting event reconstruction and inspections.

  **User Story:** *As a compliance manager, I want the audit trail to be tamper‑evident and to record who performed each action and when, so that we can reconstruct events for inspections.*

### 4.12 Usability & Support

This section describes usability and support requirements.

- **Requirement – Documentation and training:** The vendor shall provide comprehensive documentation, tutorials and examples for installation, configuration and development.

  **User Story:** *As a project manager, I need comprehensive vendor documentation, tutorials and examples, so that my team can develop and maintain projects efficiently.*

- **Requirement – Vendor support and maintenance:** The vendor shall provide updates, security patches and support to ensure the platform remains secure and operational.

  **User Story:** *As a support lead, I want access to vendor support, updates and security patches, so that the SCADA platform remains secure and operational.*

- **Requirement – Community and third‑party modules:** The platform shall offer a marketplace or repository of third‑party modules and provide a means to share modules with other users.

  **User Story:** *As a developer, I want to be able to leverage a marketplace of third‑party modules and share modules with other users, so that we can extend functionality without reinventing the wheel.*

## Non‑Functional Requirements

- **Performance** – The system should process real‑time data streams with minimal latency and maintain responsive HMIs for hundreds of concurrent users.
- **Reliability** – Mean time between failures (MTBF) shall meet corporate availability targets; redundancy and store‑and‑forward must recover automatically after failures【417854487715557†L146-L152】.
- **Maintainability** – Configuration files and projects should be version‑controlled, and system updates must be applied through a formal change‑control process.
- **Portability** – The platform must run on different operating systems and hardware, including edge devices and virtualized environments【417854487715557†L31-L43】.
- **Scalability** – Architecture must support horizontal scaling of the gateway and modules to handle increasing data volume and user count.
- **Security** – Encryption, authentication, RBAC and secrets management are mandatory【417854487715557†L86-L100】【906305396111596†L192-L243】.
- **Usability** – Interfaces should be intuitive, with consistent navigation and search capabilities; the design environment should support drag‑and‑drop development.

## Regulatory References

- **FDA Computer Software Assurance Guidance (2025)** – This guidance explains that software used to collect and record data without directly impacting production is considered **not high process risk**【318686843355289†L768-L795】.  Ignition will be categorized accordingly (GAMP Category 4).  Custom code that controls or adjusts process parameters is high process risk and must follow Category 5 validation.
- **GAMP 5, 2nd Edition** – Provides life‑cycle and risk‑based approaches for software classification and validation.  Category 4 configurable systems require user requirements, functional and configuration specifications, installation qualification (IQ), operational qualification (OQ) and performance qualification (PQ)【796508146311493†L400-L420】【796508146311493†L447-L456】.
- **21 CFR Part 11** – Governs electronic records and electronic signatures.  The platform must support audit trails, access controls, system security and record integrity.
- **ISA‑18.2 & ISA 101** – Standards for alarm management and HMI design, respectively.  Alarm aggregation and analysis features in Ignition 8.3 help comply with these standards【987429409847858†L204-L210】.

## Acceptance Criteria

1. All functional and non‑functional requirements listed in this URS are addressed in system design, configuration and testing.
2. User stories are mapped to specifications, test cases and traceability matrices to demonstrate coverage.
3. Validation deliverables (IQ, OQ, PQ) verify that the system meets requirements under normal and worst‑case conditions.
4. The system is installed on qualified hardware, configured with approved settings and documented in the configuration specification.
5. Audit trail and security features are verified to comply with 21 CFR Part 11 and GAMP 5 guidelines.
6. Any custom scripts or modules are identified and assessed separately for intended use; appropriate design and testing are performed for Category 5 components.
