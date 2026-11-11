# AI Secure Campus

**Cybersecurity, local artificial intelligence and evidence-based incident investigation**

*Artificial Intelligence and Big Data Centre of Excellence · IES Doctor Lluís Simarro (Xàtiva) · PP3.2 — Innovation project*  
**Documentation version:** 1.0

> **Core idea.** AI Secure Campus does not replace a SIEM, an intrusion detection system or a human analyst. It integrates Security Onion, a modular set of specialised security APIs and a locally operated language model, enabling staff to ask questions, retrieve real evidence, reconstruct timelines and make informed decisions. Sensors and Security Onion perform detection; AI Secure Campus adds contextual interpretation and investigation support.

## 1. Executive summary

AI Secure Campus is an applied innovation initiative within **Primary Project 3.2 (PP3.2)** of the IES Doctor Lluís Simarro Centre of Excellence in AI and Big Data. The goal is to turn the school's digital infrastructure into a **living educational Security Operations Centre (SOC)** combining network observability, threat detection, asset inventories, endpoint telemetry and locally hosted artificial intelligence.

Its operating principle is simple: **the evidence remains in specialised systems and is supplied to the AI model through controlled tools**. The language model is not the source of truth and does not need unrestricted access to network equipment or databases. It calls API-SOC, API-Net and API-Endpoint for specific, validated operations; interprets structured results; separates facts from hypotheses; and identifies risks and gaps in coverage.

The project serves infrastructure and security administrators investigating incidents; vocational education students and teachers using real or synthetic datasets under supervision; and educational or professional organisations seeking a reproducible example of SIEM, API and local-AI integration.

Practical validation during November 2026 covered Suricata–Zeek correlation, Community ID grouping, time-based investigations across different connections, optional network enrichment and AI-assisted reporting. The project is not presented as a commercial product, an autonomous SOC or an automated incident-response system.

## 2. Role within the Centre of Excellence: PP3.2

Within **PP3 — Innovation Projects**, AI Secure Campus is **PP3.2**. It complements **CENTINELA+ (PP3.1)**, which focuses on sensing and analysis in a smart educational facility. AI Secure Campus instead provides **security, visibility and investigation capabilities for the digital services, networks and assets of the educational ecosystem**.

Its four areas of contribution are:

- **Technological innovation:** security sensors, data analytics, microservices and local language models with tools.
- **Methodological innovation:** challenge-based SOC education, incident investigation, network forensics and evidence handling.
- **Transfer and reproducibility:** documented API contracts, independent components and repeatable evaluations at other centres.
- **Responsible governance:** institutional data control, explainable outputs, human review and honest reporting of verified results.

The initial PP3.2 vision also mentioned **spiking neural networks (SNN), predictive analytics and MLOps**. These remain potential research directions; **the documented operational core is the local-AI SOC assistant using Security Onion data**. This publication does not claim that a production neuromorphic detector or a complete predictive-MLOps pipeline has been deployed.

## 3. The challenge and objectives

A modern school network includes classrooms, VLANs, user devices, management systems, servers, connected devices and internal services. The volume of telemetry makes isolated manual inspection inefficient. An IDS signature may raise an alert, but investigators still need to establish who communicated, which protocol was used, when it happened, what the service returned and what else occurred nearby.

The aim is to **shorten the time between an observed signal and a verifiable hypothesis**, by:

1. Centralising visibility without replacing specialised tools.
2. Prioritising alerts by **original severity, signature meaning and query coverage**.
3. Connecting records from different sources without mistaking proximity for causation.
4. Enriching incidents with network and endpoint context when available.
5. Supporting natural-language investigation with citations, boundaries and human oversight.
6. Turning technical achievements into reusable educational material.

### 3.1 What the system does—and does not do

| Capability | Implemented responsibility |
|---|---|
| Signature detection and traffic records | **Security Onion, Suricata and Zeek**, not the LLM on its own. |
| Alert triage and classification | **API-SOC**, including categories, priorities, totals and coverage warnings. |
| Flow-based investigation | **API-SOC**, using Community ID and other justified correlations. |
| Network activity investigation | **API-SOC /analysis/**, by time window, host, domain and connection. |
| VLAN and asset location context | **API-Net**, subject to inventory coverage and authorisation. |
| Host processes and system context | **Elastic Agent + API-Endpoint**, when suitable endpoint events are present. |
| Explanations and recommendations | **Local Qwen**, based on tool results and explicit uncertainty. |
| Unapproved blocking or firewall changes | **Not part of the validated operating mode described here.** |

## 4. Architectural principles

- **Specialisation:** Security Onion is authoritative for monitored traffic and IDS observations; API-Net for network inventory; API-Endpoint for endpoint telemetry.
- **Modularity:** the three services use separate contracts and can evolve independently.
- **Controlled access:** the model does not need Elasticsearch credentials or direct RouterOS administration.
- **Read-only investigation:** the current detection and enrichment workflow retrieves data rather than automatically remediating devices.
- **Evidence before narrative:** timestamps, source documents and correlation identifiers matter more than fluent prose.
- **Explicit uncertainty:** “not observed”, “not retrieved”, “outside coverage” and “not demonstrated” are not synonyms.
- **Local processing:** the main analytical workflow can remain within institutional infrastructure, subject to correct security and privacy configuration.
- **Layered security:** segmentation, credential protection, perimeter controls, access logging and future user-based authorisation.

## 5. AI Secure Campus architecture

The solution can be described as **five cooperating layers**, from campus infrastructure to the human decision.

<!-- ARCH-DIAGRAM -->

### 5.1 Layer 1: campus networks and assets

The institution separates users, services and equipment into network segments and VLANs. MikroTik infrastructure and filtering policies control connectivity; monitored links and services produce the events used for investigation. Endpoints may contribute additional information if appropriate agents and integrations have been deployed.

The public architecture intentionally omits administrative addresses, credentials, sensitive management interfaces and precise details that could expose assets. Operational deployment instructions remain internal.

### 5.2 Layer 2: distributed monitoring with Security Onion

The installation uses **Security Onion 3.4.0** in a distributed arrangement: one **manager-search** node and **two network sensors**. The sensors observe traffic and generate detections; manager-search provides centralised search, indexing and access to the console. This separates packet observation from investigation within available hardware resources.

| Component | Responsibilities |
|---|---|
| **Network sensors** | Observe available traffic, run Suricata and Zeek, and provide PCAP evidence where configured. |
| **Manager-search** | Management, indexing and searches through Elasticsearch, plus central access to Security Onion. |
| **Security Onion Console** | **Alerts, Hunt, Correlate, Cases, Detections and PCAP** interfaces, among others. |
| **Endpoint sources** | Additional host events ingested through agents where suitable coverage exists. |

A manager-search combines management and search responsibilities; larger deployments may separate them across dedicated nodes.

**Real Security Onion screenshot — Alerts.** Four initial synthetic PHP2Shell detections, before retrieving the related Zeek evidence.

![Security Onion Alerts showing synthetic PHP2Shell detections](figuras_ai_secure_campus/security-onion-alertas.png)

### 5.3 Layer 3: specialised API services

These services form a **controlled investigation gateway** between evidence systems and AI clients:

- **API-SOC:** a FastAPI container on the **Titan** server. It performs read-only searches against Security Onion/Elasticsearch and returns summaries, alerts, correlations and traceable evidence.
- **API-Endpoint:** a separate service on **Titan** that queries available host telemetry for endpoint context.
- **API-Net:** deployed in the **network-management area**, separate from Titan and the traffic sensors, to retrieve MikroTik, VLAN, inventory and device-location information.

None of these APIs replaces its underlying platform. Each exposes a deliberately limited interface.

### 5.4 Layer 4: local AI and tool orchestration

A **locally hosted Qwen model** runs on Titan and uses structured tools. A question triggers a choice of API operation; the model processes returned JSON, evaluates evidence and produces an explanation with references. Complex investigations may progress from a detection rule to a host and from that host to multiple connections.

The model is **not** the Suricata detection engine, Elasticsearch query engine or an independent source of telemetry. The tool contracts constrain the available operations and the analytical instructions prohibit invented processes, users, device locations, mitigations or confirmed incidents.

Local inference supports institutional control of data and technological independence. The exact model release and user interface can evolve independently of the architecture.

### 5.5 Layer 5: human review and decision-making

The **SOC analyst** validates conclusions using source systems and decides whether to broaden a search, request further evidence, open a case, inspect configuration or close a synthetic exercise. Generated explanations carry no automatic authority. The same workflow supports technical reports, teaching materials and validation records.

## 6. End-to-end data flow

A typical path is:

**Network traffic → sensor → Suricata/Zeek → normalised events → Elasticsearch → API-SOC → assistant tool → local Qwen → analyst.**

1. A device initiates or receives a network communication.
2. A sensor with visibility generates evidence: **Suricata** detects signature matches; **Zeek** records connections, DNS, HTTP, TLS, files and other supported protocol events.
3. Security Onion ingests and indexes the records, retaining timestamps, datasets and network attributes.
4. API-SOC issues bounded searches by period, SID, IP, domain or Community ID. It **does not accept arbitrary Elasticsearch DSL supplied by the caller**.
5. The backend returns structured evidence, counts and warnings for missing fields or partial coverage.
6. Qwen interprets the results and may request additional queries before drawing conclusions.
7. The analyst checks references and determines whether further investigation is needed.

### 6.1 Evidence model

Relevant fields include **@timestamp, event.dataset, source.ip, source.port, destination.ip, destination.port, network.transport, network.community_id, rule.name, rule.uuid, http.*, dns.*** and—where available—Zeek identifiers such as **log.id.uid** and file metadata.

**Community ID** helps connect observations of the same network flow made by different tools. It does **not** identify an individual HTTP transaction and does not guarantee that every observation represents a single unique TCP session. Zeek UID can relate Zeek records, but it must not be equated automatically with a similarly named Suricata field. Temporal proximity also does not prove causation.

## 7. API-SOC: the investigation and analysis core

API-SOC uses **Python, FastAPI and HTTP/OpenAPI contracts**. It queries Security Onion data with service credentials, validated parameters and restricted source fields. It abstracts SIEM indexes for software clients, analysts and the local assistant.

| Function family | Example endpoints | Purpose |
|---|---|---|
| Health | /health | API availability. |
| SOC overview | /soc/summary, /soc/critical | Aggregated metrics and significant alerts. |
| Prioritisation | /soc/critical/detect, /soc/critical/workflow | Classification and guided triage. |
| Fast Triage | /soc/triage/fast | Reliable signature counts, classifications and compact priority lists. |
| Rule context | /soc/alerts/by-rule | Sources, destinations, timeline and context for an SID. |
| Initial investigation | /soc/alerts/investigate | Suricata linked to Zeek conn and HTTP with confidence and matching method. |
| Extended correlation | /soc/alerts/correlate | Related Suricata and Zeek datasets grouped by Community ID. |
| Network analytics | /analysis/ | Host, domain, traffic and timeline analysis. |
| Reports | /soc/report/json, /soc/report/html, /soc/report/markdown | Structured and shareable reports. |

These are real software contracts, not a promise that every dataset is always available.

### 7.1 Fast Triage: avoid overlooking rare signatures

**/soc/triage/fast** addresses a common weakness of Top-N lists: high-volume informational signatures can hide infrequent yet important detections. It uses **paginated composite aggregations** by signature and SID, separates original IDS severity from semantic classification, and maintains independent lists of frequent, high-severity and high-priority signatures.

Reliability features include explicit UTC windows, pagination limits, **partial_results** flags, separate severity/category totals, warnings about missing severity, query-timing telemetry and operation without slow network-inventory or Endpoint enrichment.

It provides fast, reproducible initial orientation—not a declaration that a P2 alert means compromise.

### 7.2 Semantic signature classification

A shared Python classifier evaluates signature prefixes, relevant terms and prioritisation rules.

| Class | Operational interpretation |
|---|---|
| **critical_or_high_risk** | Higher-risk signature requiring priority review and context. |
| **security_relevant** | Security-related signal even when its original severity is not critical. |
| **context_review** | Needs host role, time and correlated evidence to interpret. |
| **informational** | Routine or informational telemetry that may still matter in context. |
| **unclassified** | Not yet classified well enough; must not be silently discarded. |

**Original severity, semantic class and investigation priority are different concepts.** A 403 Forbidden detection may be contextual, while an EXPLOIT or HUNTING rule does not establish that exploitation succeeded.

### 7.3 SID and rule context

**/soc/alerts/by-rule** can answer “which hosts produced this signature?” or “which destinations occur most often?”. It supports exact names or SIDs, time filters, grouping by source/destination, bounded samples and opt-in enrichment. Overall totals come from Elasticsearch, **not** merely from the visible aggregation buckets. Responses identify missing IP data, incomplete results and enrichment errors.

### 7.4 Suricata–Zeek investigation

**/soc/alerts/investigate** starts from a rule or alert and retrieves nearby **zeek.conn** and **zeek.http** evidence. It prioritises **network.community_id**, the network five-tuple and time proximity; bounded five-tuple/time matching is a fallback when identifiers are absent. The result declares its matching method, confidence and an initial **not_established** incident verdict.

**Security Onion Hunt screenshot.** Correlation brings together distinct Suricata and Zeek records for the same observed network flow.

![Security Onion Hunt displaying Suricata and Zeek evidence](figuras_ai_secure_campus/security-onion-hunt.png)

### 7.5 Extended correlation and AI-optimised output

**/soc/alerts/correlate** accepts one or more SIDs or a Community ID and searches permitted datasets: **suricata.alert, zeek.conn, zeek.http, zeek.file, zeek.ssl, zeek.dns and zeek.quic**. Some Zeek file events without Community ID may be linked through a valid, temporally close Zeek UID; the system does not mistakenly join Suricata UID values.

Documents are grouped by Community ID and deduplicated using index and document identifier. Both response modes preserve coverage and warnings:

- **view=full:** complete individual evidence details for audit and compatibility.
- **view=summary:** a compact per-flow summary of signatures, HTTP transactions and file metadata, retaining all retrieved document references through **evidence_index_catalog** and **evidence_refs**.

This reduces redundant context delivered to Qwen, without changing the underlying searches. A Zeek file event or an HTTP 403 alone cannot prove execution, compromise or effective blocking.

### 7.6 Reports and presentation

JSON, HTML and Markdown reports support evidence archiving, technical summaries and classroom exercises. They must distinguish measured counts, interpretation, hypotheses, recommendations and coverage limits. AI text is not a formal incident record without professional review.

## 8. Network activity investigation engine

The **/analysis/** tools address questions beyond signature detection: which domains a host resolved, which connections it initiated, which protocols were present, whether traffic was periodic, how much confidence we have in service attribution, and what changed around an alert.

Zeek **DNS, Conn, HTTP, SSL/TLS and QUIC** are key data sources, with optional Suricata and network context. Service categorisation happens **after discovery** so unknown domains remain visible.

### 8.1 Available capabilities

| Capability | Function |
|---|---|
| Domain discovery | Lists names resolved by a source IP or network in a defined window. |
| Host investigation | Summarises domains, services and sessions for an IP. |
| Timeline | Orders DNS, connections, HTTP, TLS, QUIC and relevant alert events. |
| External destinations | Reports public destinations, protocols, ports and bytes where observed. |
| Services and categories | Recognises possible software, telemetry, updates, AI and other service families. |
| Explainable anomalies | Describes periodicity or connection/DNS ratios without assigning malicious intent. |
| DNS-to-connection attribution | Distinguishes resolution from observed connections and corroborating HTTP Host or TLS SNI. |
| Optional enrichment | Adds VLAN or asset-location information from API-Net with confidence warnings. |

### 8.2 Evidence levels

- **OBSERVED:** DNS lookup or resolution was seen; it does not prove a later connection.
- **CONNECTED:** a compatible connection to a resolved IP occurred within the selected window; attribution may still be tentative.
- **CONFIRMED:** HTTP Host, TLS SNI or QUIC server-name evidence corroborates the domain relationship.
- **ATTRIBUTED / USER_ATTRIBUTED:** reserved for sufficient endpoint process/user evidence; they must not be asserted without it.

Shared DNS infrastructure, CDN addresses and temporal coincidences can create weak matches; the API must expose confidence and unknown states.

### 8.3 Windows and coverage

Analysis uses explicit time windows, consistent UTC handling, logical DNS deduplication, session-gap rules and bounded query volumes. Missing fields or limited retrieval must be reported, not replaced silently. Training PCAPs use **absolute UTC windows** to keep comparisons reproducible.

## 9. API-Net: institutional network context

API-Net links security investigations with the **logical and physical network topology**. It operates in the **management segment**, separate from Titan and Security Onion sensors. It retrieves information on networks, VLANs, gateways and—when inventory evidence permits—MikroTik equipment, MAC addresses, switches and ports.

An analyst may ask: Which VLAN and subnet contain the IP? Is there an asset-inventory record? Can the IP be linked to an observed MAC and access port? Is the port a trunk, making the physical location uncertain? Which properties were verified and which merely inferred from the CIDR?

Enrichment is optional because detailed network-device interrogation can be much slower than an aggregated Elasticsearch search. A fast VLAN lookup and deeper location lookup are treated differently. Without reliable physical identification, rack, switch, port and MAC should remain unknown or be marked low confidence.

MikroTik firewalls and segmentation policies remain independently administered. **The current assistant is not validated to autonomously change firewall rules.** Future controlled actions would require identity checks, granular authorisation, human approval, auditing and reversible procedures.

## 10. API-Endpoint: host visibility through Elastic Agent

Network evidence tells us **which devices communicated, when, to whom and over which protocols**. It does not necessarily reveal **which process started a connection, which user was active or what happened inside the operating system**.

**API-Endpoint provides value by querying and enriching the telemetry collected by Elastic Agent on instrumented systems.** Elastic Agent collects permitted information, Security Onion ingests and indexes it, and API-Endpoint exposes relevant context to AI Secure Campus. The service runs on **Titan**.

### 10.1 Elastic Agent as the source of host observability

**Elastic Agent** is the unified agent used with Security Onion to collect authorised events from supported Windows, Linux and macOS endpoints. Unlike Suricata and Zeek, which inspect traffic seen by network sensors, it can provide system-level observations.

Depending on active policies and integrations, the agent may supply:

- **Host inventory:** identity, OS, agent state and device attributes.
- **Process telemetry:** creations, terminations, executables, arguments and parent processes.
- **Endpoint network events:** connections and, where captured, the responsible process and network endpoints.
- **File events:** selected operations and metadata.
- **System/security events:** logins, authentication, services and Windows/Linux logs.
- **Osquery Manager:** further host-state queries where supported and permitted.

**Simply installing an agent does not enable every event type.** Coverage depends on configured integrations, permissions and proportionate data handling.

### 10.2 Fleet management and Security Onion

**Elastic Fleet** manages agent enrolment, policies and health. Depending on platform and configuration, policies can use **Elastic Defend, System, Windows and Osquery Manager** integrations.

The path is:

**Endpoint with Elastic Agent → Fleet policy and integrations → Security Onion ingestion → Elasticsearch → API-Endpoint → AI Secure Campus assistant.**

API-Endpoint does not replace agents, install sensors or invent process events from packet captures.

### 10.3 API-Endpoint during an investigation

| Analyst question | Evidence needed | Caution |
|---|---|---|
| Which computer used this IP? | Agent identity and host inventory | IP addresses can change; check timestamps. |
| Is the agent healthy? | Fleet status and policy metadata | Enrolment does not ensure all data is collected. |
| Which process opened the connection? | Time-stamped process-linked network event | Requires matching identity, time and destination. |
| Was a suspicious process launched? | Process telemetry | An IDS alert cannot prove process execution. |
| Were files written locally? | Endpoint file events | **zeek.file** is traffic metadata, not a disk-write record. |
| Which user was involved? | Authenticated process/user events | Attribute only with adequate, traceable evidence. |

### 10.4 Correlating traffic and endpoint activity

An ideal investigation combines **Suricata** detections, **Zeek** network context and **Elastic Agent** host events. If a web alert is followed by an outgoing connection, endpoint process evidence may help test possible relationships. A process attribution requires real event timestamps and endpoints; common IPs or domains are not sufficient.

**Current limitation:** API-Endpoint and host-context queries exist, but conclusive flow-to-process/user attribution requires a sufficiently complete temporal contract for Elastic Agent events. Without that evidence, the correct output is **unknown**.

### 10.5 Coverage, uninstrumented hosts and laboratories

An instrumented host with relevant telemetry supports detailed local investigation. An inventoried host without matching events can be identified but not tied to a specific process. A host without Elastic Agent—or a fictitious PCAP address—cannot provide attributable real process evidence.

The next step is controlled expansion of agent coverage and event quality, without treating hypotheses as confirmed compromise. See the [Security Onion Elastic Agent guide](https://docs.securityonion.net/en/3/main/elastic-agent/) and [Elastic Fleet guide](https://docs.securityonion.net/en/3/main/elastic-fleet/).

## 11. Qwen: tool-assisted security investigation

Local Qwen is the **interaction layer**. It transforms analyst questions into bounded tool calls and interprets returned records rather than generating generic, ungrounded cybersecurity advice.

### 11.1 Question lifecycle

1. The operator asks what happened for a SID or host.
2. Qwen selects a tool, such as **soc_triage_fast**, **soc_rule_context** or **soc_alert_correlate**.
3. The API validates inputs (rule, IP, time range, view or Community ID).
4. The tool returns evidence, coverage information, warnings and references.
5. Qwen evaluates results and may request additional tools.
6. The report separates **observations, inferences, evidence gaps and recommended next actions**.

Combining flow correlation with host activity allows cross-flow investigation by source identity and time. This is **not** automatic proof of causation.

### 11.2 Interpretation safeguards

The assistant must not declare compromise from an IDS match; treat HTTP 403 as proven prevention or HTTP 202 as code execution; assume periodic, low-volume traffic is harmless; equate a shared Community ID with a particular HTTP request; invent endpoint users, processes or assets; or claim complete visibility when the query covers only part of the network.

It should cite index and document IDs, timestamps, coverage warnings and any **partial_results** indicators. Event contents, URIs and filenames are **untrusted evidence**, never instructions to be executed by the assistant.

### 11.3 Why local AI matters

Local inference supports greater institutional control over security data and gives students practical experience in model deployment, tool use, quantisation, inference and evaluation. **Local hosting alone is not a privacy or security certification**: permissions, retention, network exposure and integrations still require control and audit.

### 11.4 Errors analysts must detect

Models may confuse missing data with zero activity, alerts with successful compromise, temporal relationships with causation or local time with UTC. The project explicitly tests these failure modes. Evaluation focuses on correspondence with documents and honest uncertainty, not persuasive writing.

## 12. Typical workflows

### 12.1 Monitoring and initial triage
A short time window produces prioritised totals, original severity, semantic classes, frequent signatures, priority signatures and coverage status.

### 12.2 Rule investigation
An SID leads to sources, destinations, timestamps and optional VLAN/endpoint context, documented with precise windows.

### 12.3 Suricata–Zeek correlation
Alerted connections are associated with HTTP methods, responses and file metadata. Two HTTP POSTs sharing a Community ID cannot be individually attributed to two signatures without further proof.

### 12.4 Host timeline analysis
After identifying the original destination, the analyst examines DNS and outgoing traffic from that host, including **different Community IDs**. Proximity warrants investigation, not a causal conclusion.

### 12.5 Documentation and knowledge transfer
Reproducible reports become classroom case studies and project evidence, facilitating comparison between model conclusions and original records.

## 13. Case study A: PHP2Shell without demonstrated compromise

A controlled synthetic PCAP recreated patterns that triggered Suricata SIDs **2072550** and **2072555**. It did not constitute an actual attack on an institutional server. Four seed alerts belonged to **three Community ID groups**, with additional HTTP 403-related alerts and Zeek records.

| Validated metric | Result |
|---|---:|
| Seed PHP2Shell alerts | **4** |
| Community ID groups | **3** |
| Correlated Suricata alerts (including 403) | **8** |
| zeek.conn events | **3** |
| zeek.http events | **4** |
| zeek.file events | **10** |
| Unique evidence documents | **25** |
| Retrieval truncation | **None observed** |
| Demonstrated compromise | **No** |

The most informative flow contained **two distinct signatures and two HTTP requests over one TCP connection**. Correlation correctly deduplicated the evidence. Every HTTP request received **403 Forbidden**, but this does not prove the request was never partially processed.

**Educational lesson:** four alerts are not necessarily four connections—and certainly not four confirmed incidents.

## 14. Case study B: investigating later activity

A second synthetic PCAP retains the initial signatures and adds subsequent events involving the same fictitious server address. The purpose is to test **discovery and characterisation of other flows**, not to assert infection.

### 14.1 Verified timeline (UTC)

| UTC time | Observation | Defensible interpretation |
|---|---|---|
| 15:00:24–15:01:50 | Three incoming connections, four HTTP POSTs matching PHP2Shell signatures and responses 403. | Detection observed; outcome not established. |
| 15:00:56 | DNS query for audit-telemetry.invalid. | Resolution observed, not proof of a visit. |
| 15:01:02, :14, :26 | Three periodic POSTs to synthetic telemetry, HTTP 204. | Repeated HTTP activity. |
| 15:02:21 | Fourth telemetry POST, HTTP 204. | Another observed transaction. |
| 15:02:34 | DNS query for stage-followup.invalid. | Later resolution from the same host. |
| 15:02:40 | GET for a test manifest, HTTP 200. | Transfer observed; execution not proven. |
| 15:02:52 and 15:03:04 | Two check-in POSTs, HTTP 202 Accepted. | Requests accepted for possible processing; C2 not confirmed. |

The investigation identified **seven outbound HTTP flows**, in addition to the **three incoming groups**: ten Community ID groups of interest. Qwen discovered the destinations and domains through tools without being told them in advance.

### 14.2 Interpreting HTTP responses

- **403 Forbidden:** a refusal response was observed; effective prevention is not proven.
- **204 No Content:** a successful response without a body; it does not establish benign intent.
- **200 OK:** a response was delivered; it does not prove downloaded material was executed.
- **202 Accepted:** the remote server accepted a request for potential processing; not evidence of execution, infection or exfiltration.

The events justify further investigation, but **temporal proximity is not causation**. External addresses and domains in the PCAP are reserved documentation values, not real indicators of compromise. The fictitious host has no real Elastic Agent process telemetry.

### 14.3 What Qwen evaluation demonstrated

The initial investigation discovered DNS and outgoing connections but left some HTTP transactions unchecked. A subsequent run retrieved **all seven outbound HTTP transactions**, including the two 202 responses, and explicitly corrected the earlier hypothesis.

The lesson is twofold: the integration supports iterative, evidence-backed analysis, and **model completeness needs validation** even when the source platform has all the records.

**Technical verdict:** synthetic later activity is present; no independent evidence proves host compromise or causation by PHP2Shell.

## 15. Performance and validation

**Evidence visualisation.** The chart is based on validated synthetic lab results; it is not a screenshot of the assistant interface.

![AI Secure Campus: 8 Suricata alerts and 17 Zeek events](figuras_ai_secure_campus/ai-secure-campus-evidencias.svg)

The figures below refer to **specific tests**, not guaranteed performance under all network workloads.

| Measure | Observed result | Scope |
|---|---|---|
| Security Onion | **3.4.0** | Validated installation version. |
| Automated tests | **165 passed** | Investigation and correlation regression suite. |
| Correlation view=summary | **194 ms mean** | Fixed PHP2Shell test window. |
| Correlation view=full | **199 ms mean** | Same test scenario. |
| JSON size reduction | **58.8%** | About 14.5 KB to 6 KB; 25 references retained. |
| Default response | **view=full** | Backward compatibility preserved. |
| Fast Triage | **about 76 ms** | A separate earlier benchmark. |
| Earlier detection endpoint | **about 1.29 s** | Different endpoint and workload. |

### 15.1 What is—and is not—measured

API latency excludes most of the time Qwen spends selecting tools, processing context, making additional calls and writing an investigation. The results measure a specific local configuration and time window. They are not detection accuracy, a guarantee of constant throughput or evidence of an incident.

## 16. Software quality, testing and maintenance

The application uses controlled development practices: source review, automated regression testing, container deployments and a recoverable previous image. Tests check classifications, exact document counts, deduplication, bounded requests, HTTP errors, correct timestamps and API backward compatibility.

The documented tests do not replace integrated validation against the actual Security Onion deployment. Reproducible synthetic PCAPs allow comparisons between expected events and the data returned by the APIs and the model.

### 16.1 Traceability

A high-quality report distinguishes **raw facts**, **correlation methods**, **inferences**, **unknowns** and **recommended actions**, retaining the index/document identifiers needed to inspect sources. Warnings and truncated results are visible rather than hidden.

## 17. Security, privacy and data governance

### 17.1 Separation of privileges

The current investigation operations are read-only. The LLM uses tool contracts rather than unrestricted credentials. API-Net remains in the management area; API-SOC and API-Endpoint are separate services on Titan. Neither a published HTML page nor an untrusted client should directly access operational APIs.

### 17.2 Required protections

Operational deployment should use network isolation, controlled credentials, TLS, logging, minimal privilege, secret rotation and strong authentication. **API-SOC does not itself provide complete application-level user authentication and role-based access controls**; its exposure must therefore remain within a trusted perimeter until a reviewed authorisation layer is installed.

Any future write operation would need granular permissions, approvals, auditing and rollback.

### 17.3 Responsible publication

Public documents omit passwords, internal management addresses, sensitive host identities and raw production events. Training screenshots must be reviewed and synthetic cases labelled clearly. Data retention, access rights and local inference are governed by institutional policy.

## 18. Known technical limitations and uncertainty

- **Network visibility** depends on sensor positioning, protocols, encryption and collection capacity.
- **Endpoint coverage** depends on Elastic Agent installation, health and policy configuration.
- **Process attribution** cannot be inferred reliably without time-stamped agent events.
- **Community ID** is a flow correlation key, not a unique HTTP request or proof of causality.
- **DNS attribution** can be weak when hosts share resolvers, CDNs or destinations.
- **Incomplete indexes** and parser/mapping differences may hide protocol fields.
- **AI analysis** may omit available events or make overconfident inferences.
- **Deep network inventory** lookups may take longer and are therefore optional.

These limits are part of the teaching methodology: students must justify what has been shown, what remains uncertain and why.

## 19. Educational impact and knowledge transfer

AI Secure Campus supports **Networked Computer Systems Administration (ASIR)**, **Multiplatform Application Development (DAM)** and the **AI and Big Data specialisation programme**.

| Learning area | Practical applications |
|---|---|
| Networks and systems | VLANs, segmentation, protocols, firewalls, service observability. |
| Cybersecurity | Monitoring, Suricata, Zeek, PCAPs and evidence-based investigation. |
| Software development | REST APIs, FastAPI, OpenAPI, containers, validation and CI/CD. |
| Big Data | Elasticsearch, queries, aggregations, cardinality and time windows. |
| Applied AI | Local models, tool calling, grounded reasoning and evaluation. |
| Ethics and governance | Privacy, least privilege, uncertainty and human oversight. |

Students receive observed records and investigative questions rather than merely reading descriptions of attacks. They can reproduce queries, compare findings and propose evidence-based improvements.

A transferable adoption path starts with a small safe laboratory, documented API contracts and harmless synthetic PCAPs, then gradually adds inventory and endpoint telemetry.

## 20. Roadmap: operational, improving and exploratory

| Area | Current status | Next improvement |
|---|---|---|
| Distributed Security Onion and two sensors | **Operational and validated** | Refine retention and measurable coverage. |
| API-SOC, Fast Triage and classification | **Operational** | Regular benchmarks and interpretive quality. |
| Suricata–Zeek Community ID correlation | **Operational and validated** | Transaction-level matching when supported by evidence. |
| Compact response for Qwen | **Operational** | More exhaustive multistep investigations. |
| DNS/HTTP host timelines | **Operational with limits** | Consistent field mapping and attribution. |
| API-Net enrichment | **Available** | Latency and inventory quality improvements. |
| API-Endpoint | **Integrated; temporal attribution limited** | More Elastic Agent coverage and process-linked network events. |
| API authentication/authorisation | **Further hardening required** | Identity, RBAC, auditing and approval governance. |
| Autonomous infrastructure changes | **Not validated** | Controlled, reversible human-approved actions. |
| SNN and predictive MLOps | **Exploratory** | Separate research objectives and evaluation. |

The priority is not to add modules for novelty, but to improve **evidence quality, coverage, authorisation, traceability and analyst utility**.

## 21. Frequently asked questions

### Does AI Secure Campus replace Security Onion?
No. Security Onion supplies specialised packet and event monitoring. AI Secure Campus queries those sources and helps explain and investigate them.

### Does the AI automatically detect every attack?
No. Detection is limited by sensors, signatures, protocols and available telemetry. The assistant supports classification and investigation; it does not guarantee universal detection.

### Is every exploit alert a confirmed incident?
No. A signature match may indicate an attempt without demonstrating execution, persistence or compromise.

### Why use a local AI model?
To support technological autonomy, control over institutional data and practical learning. Local processing still requires appropriate security and privacy safeguards.

### What is the difference between correlated and confirmed?
Correlated observations share a documented linking criterion. A confirmed security incident needs evidence sufficient to establish the event; these are different conclusions.

### Do students investigate real production incidents?
The campus can produce operational telemetry, but repeatable teaching exercises should use **controlled synthetic captures** or prepared datasets, with permissions and privacy protections.

### What if a host has no agent?
Network-level investigation remains possible, but an observed connection alone cannot identify the responsible process or user.

### Can the chat automatically block devices?
Not as an operationally validated capability. Such actions would require specific authorisation and human review.

## 22. Technical glossary

**API-SOC:** programmatic interface for Security Onion triage, detection, correlation and analysis.  
**API-Net:** network topology, VLAN and asset-inventory interface.  
**Elastic Agent:** host telemetry collector managed through Elastic Fleet.  
**Elastic Fleet:** centralised agent enrolment, health and policy management.  
**API-Endpoint:** AI Secure Campus service for querying contextual host telemetry from instrumented devices.  
**SOC analyst:** person validating security observations and deciding investigative or response actions.  
**Community ID:** common network-flow identifier for cross-tool correlation.  
**Elasticsearch:** indexed event storage and search engine.  
**Fast Triage:** initial alert prioritisation with explicit coverage.  
**IDS:** intrusion detection system.  
**LLM:** large language model that interprets questions and tool results.  
**Manager-search:** Security Onion node combining management and search.  
**OpenAPI:** machine-readable HTTP interface specification.  
**PCAP:** packet-capture file for investigation and reproducible testing.  
**SID:** Suricata detection-signature identifier.  
**Suricata:** network analysis and signature-based detection engine.  
**Zeek:** network analyser producing structured flow and protocol logs.  
**zeek.file:** metadata about files or content observed in traffic, not proof of local file creation.  
**view=summary:** compact correlation result retaining evidence references.  
**SNN:** spiking neural networks; an exploratory research line.  
**MLOps:** machine-learning lifecycle practices, not an established detector in this release.

## 23. Technical references and editorial review

- [AI and Big Data Centre of Excellence — IES Doctor Lluís Simarro](https://ieslluissimarro.org/projectes/excelencia/index.html)
- [Centre of Excellence activities](https://ieslluissimarro.org/projectes/excelencia/actividades.html)
- [Security Onion distributed architecture](https://docs.securityonion.net/en/3/main/architecture/)
- [Elastic Agent in Security Onion](https://docs.securityonion.net/en/3/main/elastic-agent/)
- [Security Onion Elastic Fleet](https://docs.securityonion.net/en/3/main/elastic-fleet/)
- Project-team validated synthetic PHP2Shell A/B laboratory evidence, November 2026. **No production credentials, management addresses or sensitive document IDs are published.**

**Editorial recommendation:** maintain two levels of documentation: this **public edition** describing aims, architecture and non-sensitive evidence, and a restricted operational annex containing exact deployment topology, addresses, access procedures, secrets and incident-response processes. Review all screenshots before publication.

---

**IES Doctor Lluís Simarro · AI and Big Data Centre of Excellence**  
**Primary Project 3.2 — AI Secure Campus · Applied innovation in cybersecurity and local AI**

*This documentation reflects the technically validated state as of 11 November 2026 and will evolve with the project.*
