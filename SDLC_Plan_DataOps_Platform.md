# Software Development Life Cycle Plan – Industrial DataOps Platform (HighByte Intelligence Hub)

## 1. Purpose and Scope

This plan outlines the SDLC activities for implementing HighByte Intelligence Hub as an industrial DataOps platform in a regulated manufacturing environment.  The platform collects, contextualizes, conditions and routes operational data from machines, sensors, databases and enterprise systems.  It does **not** directly control process parameters; instead, it acts as middleware for data integration and governance.  As such, the base platform is treated as a **GAMP Category 4** configurable product.  According to FDA guidance, software that collects and records data for monitoring or manages data (e.g., processing, storing or organizing) without directly affecting production is **not high process risk**.  However, custom pipelines or scripts that automate control logic may elevate specific instances to Category 5 and require separate assessments.  This plan addresses the core platform; each pipeline, model or custom function must be evaluated for intended use and risk.

## 2. SDLC Phases

### 2.1 Concept

* **Define business need and intended use** – Identify why a DataOps platform is required (e.g., to model and contextualize data, orchestrate flows, reduce integration time, expose operational data to analytics and AI).  Document high‑level requirements in a URS.
* **Stakeholder identification** – Include OT/IT integrators, process engineers, data scientists, quality assurance, compliance officers and system administrators.
* **Risk classification** – Confirm that the platform’s role is to merge and route data rather than control process parameters.  Under FDA CSA, collecting and managing data is a “not high process risk” activity.  Note that if pipelines perform automated adjustments or if JavaScript functions send commands back to equipment, those elements may become high process risk and must be managed separately.
* **Determine GAMP category** – Base platform is a configurable product (Category 4).  Configuration includes connectors, models, pipelines and namespaces.  Use of vendor‑supplied scripting (JavaScript functions or Node packages for data transformation) may be considered custom code and fall under Category 5.
* **High‑level risk assessment** – Identify hazards such as data mapping errors, incorrect units, message loss, or unauthorized data exposure.  Plan mitigation strategies and define boundaries for low‑risk vs. high‑risk pipelines.

### 2.2 Procure

* **Supplier qualification** – Evaluate the vendor’s quality and support.  Review product documentation, release notes and security posture.  Assess HighByte’s capabilities for codeless integration and data conditioning: the hub can collect data over open standards (OPC UA, MQTT, REST, SQL) and merge data from multiple systems into complex payloads without coding.
* **Due diligence** – Obtain documentation on security features and quality controls, such as role‑based access control, audit logging and identity management.  Verify support for TLS and certificate management, integration with Active Directory and SAML, and high availability.
* **Procurement justification** – Document the rationale for selecting HighByte Intelligence Hub, including its ability to model data, orchestrate flows, perform edge deployment, and support REST API access.

### 2.3 Plan

* **Project planning** – Define project scope, deliverables, milestones and resources.  Establish roles for configuration, testing, quality assurance and operations.
* **Validation plan** – Develop a CSA plan describing assurance activities.  Specify required documentation: URS, configuration specification, pipeline design specifications (for custom functions), test plan and traceability matrix.  Emphasize that pipelines and models will undergo individual intended‑use assessments.
* **Training plan** – Plan training for system administrators, integrators and users on configuring connectors, models, pipelines and security settings.  Include vendor training offerings.

### 2.4 Build

* **Infrastructure preparation** – Provision hardware or virtual machines appropriate for edge or central deployments.  HighByte can run on lightweight edge devices, industrial switches, IoT gateways or as a Docker image.  Set up environments (development, validation, production) and ensure network segmentation.
* **Installation** – Install the Intelligence Hub software according to vendor instructions.  For high availability, install database servers (e.g., PostgreSQL) and configure primary/secondary nodes with heartbeats and file synchronization.  Document installation steps for IQ.
* **Baseline environment** – Record software versions, hardware specifications and configuration files.  Store certificates required for secure communications.

### 2.5 Configure (Category 4)

* **Configuration specification** – Translate the URS into a detailed configuration specification.  Document connectors (OPC UA, MQTT, REST, SQL), data sources, polling intervals and credentials.  Define data models representing machines, products and processes with standardized attribute names and units.  Identify how data points are merged, contextualized and nested.
* **Pipelines and orchestration** – Specify pipelines that read, filter, buffer, transform, format and compress data.  Include store‑and‑forward configuration to buffer data when destinations are unavailable.  Document event‑based triggers (On Change), conditional logic (Switch) and model validation stages.
* **Data conditioning** – Configure functions for filtering sensor jitter and aggregating metrics such as average, minimum, maximum, count and delta.  Record any JavaScript or Node functions used for transformations; these may fall under Category 5.
* **Unified namespace and MQTT broker** – If using the embedded MQTT broker to build a unified namespace, define topic structure and ensure JSON or Sparkplug payloads are chosen.  Use the UNS Client to inspect broker contents and decode payloads.
* **Security configuration** – Import certificates to secure connections, configure TLS settings, enforce client authentication and set role‑based access controls.  Create users, roles and claims; define allowed actions for each resource (e.g., read, write, publish, subscribe).  Restrict scope with tags.
* **Audit logging and event log** – Enable the event log to capture runtime and audit messages.  Verify that log entries include time, event level, source and message.  Use the “Verify Logs” function to confirm log integrity.  Plan for exporting JSON logs for external monitoring.
* **Identity management** – Configure Active Directory or SAML identity providers if required.  Map roles returned by the identity provider to hub roles.
* **High availability** – Configure primary and secondary nodes, set heartbeat interval and ensure file synchronization for fail‑over.
* **REST Data Server** – Set up the REST Data Server to act as an API gateway, exposing modeled data, instances, pipelines and connections.  Define API keys with appropriate expiration and scope.

### 2.6 Design (Category 5 when applicable)

The core platform does not require custom code; however, any use of JavaScript or Node functions within pipelines or development of plug‑ins constitutes custom software.  For each such instance:

* **Design specification** – Document the purpose, inputs, outputs, algorithms and error handling for the custom script.  Include unit/module design if complex.
* **Traceability** – Link design elements to URS requirements and configuration items.
* **Risk assessment** – Evaluate whether the custom function influences process parameters or product quality.  If so, treat it as high process risk and apply additional assurance.

### 2.7 Test

* **Installation Qualification (IQ)** – Verify that the platform and supporting infrastructure (database, operating system, libraries) are installed according to specifications.  Confirm correct versions of connectors and plug‑ins.
* **Operational Qualification (OQ)** – Test configured connectors, models and pipelines against the configuration specification.  Confirm data acquisition, transformation and routing functions.  Test security controls: user authentication, role‑based permissions, certificate validation, and audit logging.  Exercise high availability by simulating node failure and verifying that data flows continue.
* **Performance Qualification (PQ)** – Demonstrate that the platform meets the URS under realistic operating conditions.  Simulate data loads representative of production, verify that models maintain context, pipelines process within acceptable latency, and data is delivered to downstream systems.  Validate store‑and‑forward by disconnecting destinations and confirming data is buffered and forwarded when connectivity is restored.
* **Custom function testing** – For Category 5 scripts, perform unit and integration testing.  Validate calculations and error handling.  Document test results and traceability to design specification.
* **Defect handling** – Record deviations, investigate root causes and implement corrective actions.  Retest until acceptance criteria are met.

### 2.8 Implement

* **Deployment** – Promote validated configurations and pipelines into the production environment.  Use change control procedures to manage deployment steps.
* **Training and documentation** – Train administrators and users on using the hub (configuring pipelines, models, connectors and security).  Provide documentation for operation, maintenance and troubleshooting.
* **Go‑live review** – Conduct a readiness review with quality and compliance stakeholders.  Obtain approval before activation.

### 2.9 Release and Maintenance

* **Release package** – Compile validation summary report, traceability matrix and final approval.  Baseline all configuration files, models and scripts.
* **Change management** – Manage future changes through documented procedures.  Evaluate the intended use and risk impact of any new pipeline, model or custom function.  For low‑risk changes (e.g., adding a tag), perform regression testing; for high‑risk changes, repeat design and test phases.
* **Periodic review** – Periodically review system performance, audit logs, security settings and compliance with GxP requirements.  Conduct periodic requalification and update risk assessments.
* **Incident management** – Define procedures for handling system outages, data quality issues and security incidents.  Include restoration from backups and communication to stakeholders.

## 3. References

* FDA, **Computer Software Assurance for Production and Quality System Software** (2025).  This guidance clarifies that software used to collect and manage data for monitoring or quality system support is **not high process risk** and requires assurance activities commensurate with its risk.
* R.D. McDowall, **Understanding and Interpreting the GAMP 5 Life Cycle Models for Software**.  Category 4 applications start with a URS, functional specification and configuration specification and rely on installation, configuration and testing activities.
* HighByte, **Intelligence Hub Solution Brief (v4.2)**.  The hub supports codeless integration over open standards, data conditioning and transformations, data modeling, orchestration and store‑and‑forward pipelines, REST API access, embedded MQTT broker and unified namespace, and central management features including roles, auditing and high availability.
* HighByte User Guide – *Users and Roles*, *Event Log*, *Certificates*, *SAML and Active Directory*, and *High Availability* sections.  These document role‑based access control, actions and scope, event logging, certificate management, identity provider integration, and high availability configuration.
