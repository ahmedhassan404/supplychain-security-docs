# ChainGuard: System Functional and Non-functional Requirements

## Table of Contents
- Introduction
- Functional Requirements
- Non-functional Requirements

## Introduction

### Purpose
This document defines the functional and non-functional requirements for ChainGuard, a Dependency Analysis and Supply Chain Security Platform for Web Applications. The platform performs automated vulnerability detection, machine-learning risk scoring, and detailed security reporting.

### Scope
ChainGuard enhances supply chain security for modern web applications by:
- Scanning dependency manifests (package.json, lock files)
- Detecting known vulnerabilities using multiple security databases
- Performing ML-powered package classification
- Generating dependency graphs, risk scores, and SBOM files
- Providing dashboards and exportable reports

### Document Structure
- Functional Requirements: Define what the system must do
- Non-functional Requirements: Define constraints, performance, security, and operational expectations

## Functional Requirements

### FR1: Authentication and Authorization

#### FR1.1: User Registration
- FR1.1.1: Allow registration using email, name, password
- FR1.1.2: Validate email format and password strength
- FR1.1.3: Prevent duplicate email registration
- FR1.1.4: Return a JWT token upon successful registration
- FR1.1.5: The system SHALL support OAuth authentication through GitHub
- FR1.1.6: The system SHALL support OAuth authentication through Google

#### FR1.2: User Login
- FR1.2.1: Authenticate via email + password
- FR1.2.2: Return JWT token
- FR1.2.3: Invalidate tokens on logout
- FR1.2.4: Support token expiration (default 24 hours)

#### FR1.3: User Profile
- FR1.3.1: Retrieve authenticated user profile
- FR1.3.2: Support RBAC (user/admin)
- FR1.3.3: Track creation/update timestamps

#### FR1.4: Session Management
- FR1.4.1: Validate JWT tokens on protected endpoints
- FR1.4.2: Return 401 for invalid/expired tokens
- FR1.4.3: Support token refresh

### FR2: Project Management

#### FR2.1: Project Creation
- Create project (name + description)
- Validate name uniqueness per user
- Associate projects with owners
- Support npm package manager

#### FR2.2: Project Retrieval
- List all user projects
- Support pagination
- Retrieve project details
- Include latest scan info

#### FR2.3: Project Update
- Update metadata (name, description, repo URL)
- Validate ownership
- Track update timestamps

#### FR2.4: Project Deletion
- Allow deletion
- Remove associated scans and data
- Validate ownership

### FR3: File Upload and Scanning

#### FR3.1: File Upload
- Support multipart/form-data
- Accept: package.json, package-lock.json
- Validate file format + size (max 50MB)
- Return scan ID

#### FR3.2: Scan Initiation
- Configurable options (devDeps, transitive, CVEs, licenses)
- Asynchronous job queue
- Return initial scan status

#### FR3.3: Scan Status
- Real-time progress
- Track status: uploading, scanning, complete, error
- Provide % progress
- Support WebSocket/SSE

#### FR3.4: Dependency Graph Extraction
- Parse lock files
- Build dependency graph
- Store graph in graph database
- Compute dependency depth/path

### FR4: Vulnerability Detection

#### FR4.1: CVE Database Integration
- Integrate with NVD, OSV, GitHub, npm
- Maintain internal CVE DB

#### FR4.2: Vulnerability Matching
- Match by package name/version
- Support version ranges
- Identify fixed versions
- Assign severity levels

#### FR4.3: CVE Information
- Provide CVSS scores
- Provide descriptions + references
- Track publication/update dates
- Provide remediation steps

#### FR4.4: Vulnerability Management
- View all vulnerabilities
- Filter by severity, status, package
- Mark vulnerabilities as fixed/ignored
- Track status changes

### FR5: Machine Learning & Risk Analysis

#### FR5.1: Classification
- Classify packages (safe, suspicious, malicious)
- Provide confidence scores
- Risk score (0–10)
- Explain classification

#### FR5.2: Feature Extraction
- Metadata features
- Text embeddings (README)
- Graph features (centrality, depth)
- Behavioral features
- Static analysis features

#### FR5.3: Model Inference
- XGBoost for tabular data
- BERT/Sentence Transformers for text
- GNN (GraphSAGE/GAT)
- Ensemble models
- Inference <500ms/package

#### FR5.4: Anomaly Detection
- Detect behavioral anomalies
- Detect supply chain anomalies
- Detect statistical anomalies
- Provide explanations

#### FR5.5: Threat Prediction
- Predict supply chain attack vectors
- Probability scores
- Mitigation steps
- Identify impacted packages

#### FR5.6: Model Management
- Track metrics
- Support versioning
- Feature importance
- A/B testing

### FR6: Dependency Analysis & Visualization

#### FR6.1: Dependency Tree
- Show hierarchical tree
- Distinguish direct vs transitive
- Show vulnerability status
- Filter by type/depth

#### FR6.2: Graph Visualization
- Provide nodes/edges
- Support different layouts
- Color-code by vulnerability
- Stable node positioning

#### FR6.3: Dependency Paths
- Identify all paths to a package
- Show transitive introductions
- Highlight vulnerable paths

#### FR6.4: Dependency Stats
- Total package count
- Max depth
- Vulnerable package counts

### FR7: Dashboard and Reporting

#### FR7.1: Security Statistics
- Vulnerabilities by severity
- Trends
- Risk score
- Completion %
- Fix history

#### FR7.2: Activity Feed
- Real-time feed
- Key events
- Pagination
- Relative timestamps

#### FR7.3: Dependency Health
- Health scores
- Vulnerabilities per dependency
- Trends

#### FR7.4: Report Generation
- Executive Summary
- Detailed Analysis
- Compliance (OWASP, NIST, ISO 27001, SOC2)
- Trend Analysis
- Custom date ranges
- Asynchronous generation

#### FR7.5: Report Export
- PDF
- CSV
- JSON
- Include charts

### FR8: Data Export

#### FR8.1: Scan Results
- JSON
- CSV
- PDF
- Include all dependency + vulnerability data

#### FR8.2: Vulnerability Exports
- Filter by severity/status/package
- Include CVE details

### FR9: CVE Database Synchronization

#### FR9.1: Sync Operations
- Manual sync (admin)
- Scheduled automatic sync
- Sources: NVD, OSV, GitHub, npm
- Track sync logs

#### FR9.2: CVE Search
- Search by CVE ID
- Search by package
- Search by severity
- Keyword search

### FR10: Package Information

#### FR10.1: Package Details
- Metadata
- Versions
- Licenses
- Maintainers
- Downloads
- Repo info

#### FR10.2: Package Search
- Search npm registry
- Include security score
- Pagination

#### FR10.3: Package Versions
- List versions
- Show security status
- Recommend safe version

### FR11: Real-time Updates

#### FR11.1: WebSocket
- Real-time updates
- JWT authentication
- Channel subscriptions
- Scan progress push

#### FR11.2: SSE
- Stream progress
- Auto-close on completion

## Non-Functional Requirements (NFR)

### NFR1: Performance

#### Response Time
- Auth <200ms
- Dashboard <500ms
- Scan status <100ms
- Vulnerability detail <300ms
- ML inference <500ms
- Exports <2s for small files

#### Throughput
- 100+ packages/sec
- 1000+ concurrent users
- 10,000+ requests/min

#### Scan Processing
- 100 packages <5 min
- Parallel processing
- Progress every 2–3 seconds

### NFR2: Scalability
- Horizontal scaling
- Load balancing
- ML model server scaling
- DB read replicas
- Connection pooling
- Object storage scaling

### NFR3: Reliability & Availability
- 99.9% uptime
- Health checks
- Retry logic
- Cached fallback
- Daily backups
- PITR support

### NFR4: Security
- JWT auth
- bcrypt/argon2
- RBAC
- Input validation
- TLS 1.2+
- Data encryption
- Rate limiting
- API key vault storage
- WAF + DDoS protection

### NFR5: Usability
- Modern UI
- Drag-and-drop uploads
- Clear indicators
- API documentation
- Tutorials
- WCAG 2.1 AA

### NFR6: Maintainability
- Coding standards
- 80%+ test coverage
- Structured logging
- APM
- ML model drift detection

### NFR7: Compatibility
- npm, Yarn, pnpm
- Modern browsers
- API versioning

### NFR8: Compliance
- OWASP, NIST, ISO 27001, SOC 2
- GDPR
- Data export/delete options

### NFR9: Resource Efficiency
- Caching
- DB indexing
- Query optimization
- ML resource optimization

Document End

