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

1. **PLC and device connectivity** – *As a control engineer, I need the SCADA platform to connect to major PLCs and industrial devices via OPC UA and vendor drivers such as Modbus, TCP/UDP, BACnet, Allen‑Bradley and Siemens, so that I can acquire live data from plant‑floor equipment.*  Ignition provides drivers for Modbus, UDP & TCP, BACnet, Allen‑Bradley, Siemens and other PLCs【417854487715557†L22-L29】, enabling seamless data collection.

2. **SQL and data sources** – *As a SCADA administrator, I want the platform to connect to any SQL database (MSSQL, Oracle, PostgreSQL, MySQL) and web services, so that I can read from or write to enterprise systems.*  The Ignition platform includes an OPC UA server and can talk to SQL databases【417854487715557†L22-L29】.

3. **Unlimited tags and connections** – *As an IT manager, I need the system to support unlimited tags, device connections and designer sessions without extra licensing costs, so that the solution can scale with plant growth.*  Ignition’s server‑centric model allows unlimited clients and tags【417854487715557†L31-L43】【417854487715557†L66-L70】.

4. **Store and forward** – *As a network engineer, I want buffered store‑and‑forward functionality that caches data when network connectivity is lost and forwards it once connectivity is restored, so that no data is lost during outages.*  Ignition includes a store‑and‑forward engine that buffers data locally before writing to SQL databases【646938752162582†L218-L231】.  In version 8.3, store‑and‑forward includes improved caching and diagnostics【906305396111596†L53-L90】.

5. **Event streaming** – *As a developer, I need the platform to publish data events to external brokers via Kafka or HTTP, so that other services can consume real‑time event streams.*  Ignition 8.3 introduces an Event Streams module for event‑driven data transfer, supporting Kafka and HTTP and providing subscription‑based updates【906305396111596†L53-L90】.

### 4.2 Visualization & HMI

1. **Web‑based responsive HMI** – *As a plant operator, I want to access dashboards from any modern web browser on desktop, tablet or mobile devices, so that I can monitor the process remotely.*  The Perspective module provides mobile‑responsive industrial applications that run in a browser【417854487715557†L31-L43】【646938752162582†L257-L269】.

2. **Desktop HMI** – *As a control engineer, I need a desktop‑based HMI for legacy systems, so that I can run full‑screen operator interfaces on dedicated workstations.*  The Vision module offers a Java‑based HMI and supports full‑screen mode【646938752162582†L236-L249】.

3. **Drawing and animation** – *As an HMI developer, I want to create custom graphics and animate shapes bound to tag values, so that the operator interface reflects the process accurately.*  Ignition 8.3 adds a native drawing editor for creating diagrams and animations and binding them to tag values【906305396111596†L95-L117】.

4. **Forms and offline mode** – *As an operator in the field, I want to enter data via forms and work offline if network connectivity is intermittent, so that I can capture observations and synchronize them later.*  The Perspective module includes a quick form generator and supports offline mode; data entered offline syncs when reconnected【906305396111596†L130-L142】.

5. **Report designer** – *As a process engineer, I need a report designer with charts and the ability to convert bitmap images to vector for scalable graphics, so that I can create professional reports.*  Ignition 8.3 enhances the reporting module with new chart options and raster‑to‑vector conversion【906305396111596†L118-L126】.

### 4.3 Historian & Data Storage

1. **High‑performance historian** – *As a process engineer, I want a built‑in time‑series historian with zero configuration and high‑performance queries, so that I can store and analyze large volumes of process data efficiently.*  Ignition 8.3 introduces a built‑in Power Historian that collects time‑series data and provides a time‑series API for custom historians【906305396111596†L26-L40】.

2. **Metadata and modeling** – *As a data modeler, I need the historian to capture metadata about equipment and measurements, so that I can organize and query historical data by context.*  Power Historian automatically models data and attaches metadata【906305396111596†L26-L40】.

### 4.4 Alarming & Notifications

1. **Alarm configuration and escalation** – *As a control engineer, I want to configure alarms with priorities, setpoints and escalation rules, so that operators receive timely notifications when process limits are exceeded.*

2. **Advanced notifications** – *As an operator, I want to receive alarm notifications via SMS, email, voice and messaging services (e.g., WhatsApp), so that I can respond quickly.*  Ignition 8.3 provides new alarm integrations including WhatsApp and Twilio Voice【906305396111596†L225-L233】.

3. **Alarm aggregation and analysis** – *As a compliance manager, I need ISA‑18.2 compliant alarm statistics (count, duration, chattering) and aggregated overviews, so that I can evaluate system health and performance.*  The new alarm aggregation system in Ignition 8.3 provides statistics and ISA‑compliant tools【987429409847858†L204-L210】.

### 4.5 Reporting & Analytics

1. **Custom reports** – *As a process engineer, I need to define and schedule reports that include trends, tables and images, so that I can document process performance and share insights.*

2. **Data export** – *As a data analyst, I want to export historical and real‑time data in standard formats (CSV, JSON), so that I can analyze it using external tools.*

3. **Real‑time dashboards** – *As an operator, I need dashboards with configurable widgets (gauges, charts, bar graphs) that update in near‑real time, so that I can make informed decisions during production.*

### 4.6 Integration & Interoperability

1. **APIs and scripting** – *As a developer, I want a Python‑based scripting environment and a REST API, so that I can automate tasks, integrate with external systems and extend functionality.*  Ignition’s open scripting environment and API support custom automation【417854487715557†L46-L63】.

2. **MQTT and messaging** – *As an integration specialist, I want the platform to publish and subscribe to MQTT topics and support Sparkplug B, so that I can integrate with IIoT ecosystems.*  Ignition offers MQTT modules and can act as a gateway for Sparkplug messages (no specific citation but widely known; this statement is included based on general product knowledge).  

3. **Enterprise system integration** – *As an enterprise architect, I need connectors for ERP, MES, historian and cloud services (AWS, Azure, SAP), so that data flows seamlessly between OT and IT.*

4. **File‑based configuration and source control** – *As a DevOps engineer, I want project configuration stored in JSON files and accessible via Git, so that I can manage changes through version control.*  Ignition 8.3 uses file‑based configuration storage ready for source control【987429409847858†L107-L174】.

### 4.7 Security & Access Control

1. **Encrypted transport** – *As a QA specialist, I need all communications to be encrypted using TLS 1.2 or higher, so that data integrity and confidentiality are preserved.*  Ignition supports TLS 1.2/1.3 encryption【417854487715557†L86-L100】.

2. **Authentication and SSO** – *As an IT administrator, I want to integrate with Active Directory and other federated identity providers with multi‑factor authentication (MFA) and single sign‑on (SSO), so that user management aligns with corporate policies.*  Ignition supports federated identity, MFA and SSO【417854487715557†L86-L100】.

3. **Role‑based access control (RBAC)** – *As a security officer, I need to assign permissions at the project, tag and resource levels, so that users only access functions and data they are authorized to use.*  Ignition’s security model includes role‑based ACLs and audit logging【417854487715557†L86-L100】.

4. **Secrets management** – *As a security engineer, I want passwords, certificates and API keys managed outside configuration files and integrated with external secret vaults, so that credentials are protected.*  Ignition 8.3 introduces a secrets management framework that removes secrets from configuration files and integrates with third‑party secret platforms【906305396111596†L192-L243】.

5. **Audit logging** – *As a compliance manager, I need a tamper‑evident audit trail of user actions, configuration changes, alarms and report generation, so that I can reconstruct events and demonstrate compliance.*  Audit logging is built into the platform【417854487715557†L86-L100】.

### 4.8 Reliability & Availability

1. **Redundancy and failover** – *As an IT manager, I need the SCADA server to support redundant clustering and automatic failover, so that operations continue during hardware or software failures.*  Ignition supports redundancy and failover【417854487715557†L146-L152】.

2. **Store‑and‑forward reliability** – *As a network engineer, I need store‑and‑forward to maintain data integrity during network outages and resume synchronization upon recovery, so that no data is lost.*  Ignition’s store‑and‑forward engine provides this buffering【646938752162582†L218-L231】 and has been enhanced with robust caching and diagnostics【906305396111596†L53-L90】.

3. **Deployment modes** – *As a DevOps engineer, I want the ability to run separate development, test, staging and production environments with different resources, so that we can safely test changes before release.*  Ignition 8.3 introduces deployment modes and project resources for separate environments【987429409847858†L167-L175】.

### 4.9 Deployment & Configuration

1. **Cross‑platform support** – *As an infrastructure engineer, I want the platform to run on Windows, Linux and macOS with minimal hardware, so that I can deploy it on any industrial PC, virtual machine or container.*  Ignition runs cross‑platform and requires minimal hardware【417854487715557†L31-L43】.

2. **Containerization and quick launch** – *As a systems administrator, I need pre‑built Docker containers and quick‑launch installers, so that installation and upgrades are simple and reproducible.*  The modern gateway UI and quick‑launch containerization described for Ignition 8.3 support rapid deployment【987429409847858†L107-L174】.

3. **Gateway configuration UI** – *As an administrator, I want a user‑friendly gateway interface with integrated search, so that I can configure devices, projects, security and modules efficiently.*  Ignition 8.3 introduces a redesigned gateway configuration page with integrated search【987429409847858†L107-L174】.

4. **File‑based backup & restore** – *As an IT manager, I want the ability to back up configuration and projects via files and restore them across systems, so that I can quickly recover from failures and replicate environments.*

### 4.10 Scalability & Performance

1. **High throughput** – *As a system architect, I want the SCADA platform to handle thousands of devices and millions of tags while maintaining low latency (< 500 ms for control updates), so that production scales without degradation.*

2. **Elastic load distribution** – *As an IT manager, I want to distribute workloads across multiple servers and containers, so that we can scale horizontally as demand increases.*

### 4.11 Compliance & Data Integrity

1. **Electronic records** – *As a QA specialist, I need the platform to handle electronic records and signatures in compliance with 21 CFR Part 11, ensuring that records are complete, accurate and readily retrievable.*

2. **Data integrity (ALCOA principles)** – *As a QA reviewer, I need data to be attributable, legible, contemporaneous, original and accurate, with controlled timestamps and versioning, so that auditability is maintained.*

3. **Regulatory audit trail** – *As a compliance manager, I want the audit trail to be tamper‑evident and to record who performed each action and when, so that we can reconstruct events for inspections.*

### 4.12 Usability & Support

1. **Documentation and training** – *As a project manager, I need comprehensive vendor documentation, tutorials and examples, so that my team can develop and maintain projects efficiently.*

2. **Vendor support and maintenance** – *As a support lead, I want access to vendor support, updates and security patches, so that the SCADA platform remains secure and operational.*

3. **Community and third‑party modules** – *As a developer, I want to be able to leverage a marketplace of third‑party modules and share modules with other users, so that we can extend functionality without reinventing the wheel.*

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
