# Example Country Regulatory Compliance Mapping

## Overview

This document maps Example Countryn cybersecurity and data protection regulations to voluntary international frameworks, specifically NIST Cybersecurity Framework 2.0. This layered approach ensures:

1. **Requirements Layer**: Example Countryn legal obligations (jurisdiction-specific)
2. **Framework Layer**: NIST CSF 2.0 voluntary framework (reusable, jurisdiction-agnostic)
3. **Implementation Layer**: ODA Reference Architecture (reusable across all jurisdictions)

```
┌─────────────────────────────────────────────────────────┐
│  Example Countryn Legal Requirements                           │
│  - Example Cybersecurity Act                             │
│  - Example Data Protection Act                                           │
│  - AIGE Accountability                                 │
└─────────────────────┬───────────────────────────────────┘
                      │ maps to
                      ▼
┌─────────────────────────────────────────────────────────┐
│  Voluntary Framework: NIST CSF 2.0                     │
│  - Govern, Identify, Protect, Detect, Respond, Recover│
└─────────────────────┬───────────────────────────────────┘
                      │ implemented by
                      ▼
┌─────────────────────────────────────────────────────────┐
│  ODA Reference Architecture                            │
│  - OAuth2/OIDC, Encryption, Monitoring, etc.          │
└─────────────────────────────────────────────────────────┘
```

**Benefit**: Other countries (France for Orange, etc.) can create their own requirements layer while reusing the framework and implementation layers.

---

## Example Countryn Regulatory Landscape

### 1. Example Cybersecurity Act

**Authority**: National Cyber Security Agency (Example Regulatory Authority)

**Applicability**: Critical National Information Infrastructure (CNII) operators, licensees under Communications and Multimedia Act 1998 (including telecommunications operators like Example MNO)

**Key Obligations**:
- Incident reporting to Example Regulatory Authority within prescribed timeframes
- Implementation of cybersecurity risk management framework
- Appointment of Chief Information Security Officer (CISO)
- Cybersecurity audit requirements
- Supply chain security management
- Third-party risk management

### 2. Personal Data Protection Act 2010 (Example Data Protection Act Amendments)

**Authority**: Personal Data Protection Commissioner, Department of Personal Data Protection

**Applicability**: All organizations processing personal data of Example Countryn individuals

**Key Principles** (7 Principles):
1. General Principle: Personal data processed lawfully and fairly
2. Notice and Choice Principle: Inform and obtain consent
3. Disclosure Principle: Data disclosed only with consent or legal basis
4. Security Principle: Reasonable security measures
5. Retention Principle: Data retained only as long as necessary
6. Data Integrity Principle: Data accurate, complete, not misleading, up-to-date
7. Access Principle: Data subjects have right to access and correct

**2024 Amendments** (Expected/Proposed):
- Mandatory data breach notification (within 72 hours)
- Increased penalties for non-compliance
- Extraterritorial application
- Data Protection Officer (DPO) requirements
- Data localization considerations
- Enhanced consent mechanisms

### 3. Artificial Intelligence Governance and Ethics (AIGE) Framework

**Authority**: Example Country Digital Economy Corporation (Example Digital Authority)

**Status**: Voluntary framework (becoming de facto standard for government contracts)

**Pillars**:
1. **Accountability**: Clear assignment of responsibility for AI systems
2. **Transparency**: Explainable AI decisions
3. **Fairness**: Non-discriminatory AI outcomes
4. **Safety and Reliability**: Robust and secure AI systems
5. **Privacy**: Data protection in AI processing
6. **Human Oversight**: Human-in-the-loop for critical decisions

**Scope**: AI/ML systems deployed in production environments, particularly in critical sectors (telecommunications, finance, healthcare)

---

## Compliance Mapping: Example Countryn Law → NIST CSF 2.0

### Example Cybersecurity Act Compliance

#### CSA-01: Cybersecurity Risk Management Framework

**Legal Requirement**:
- Operators of CNII must implement a cybersecurity risk management framework
- Framework must be based on recognized international standards

**Maps to NIST CSF 2.0**:
- **GV.RM (Risk Management Strategy)**: Establish risk management objectives
- **ID.RA (Risk Assessment)**: Identify vulnerabilities and threats
- **PR.IR (Infrastructure Resilience)**: Implement resilience mechanisms

**NIST CSF 2.0 Reference**: `SECURITY-NIST-CSF-MAPPING.md:GV.RM-01, ID.RA-01, PR.IR-02`

**Compliance Evidence**:
- Risk register documenting identified risks
- Risk treatment plans with mitigation controls
- Regular risk assessment reports
- Board-level risk reporting

---

#### CSA-02: Incident Reporting to Example Regulatory Authority

**Legal Requirement**:
- Report cybersecurity incidents to Example Regulatory Authority within 24 hours of detection
- Categories: Unauthorized access, data breach, service disruption, malware, DDoS

**Maps to NIST CSF 2.0**:
- **DE.AE-02 (Adverse Event Analysis)**: Analyze potentially adverse events
- **RS.MA-01 (Incident Management)**: Execute incident response plan
- **RS.MA-02 (Incident Triage)**: Triage and validate incidents

**NIST CSF 2.0 Reference**: `SECURITY-NIST-CSF-MAPPING.md:DE.AE-02, RS.MA-01, RS.MA-02`

**Implementation Requirements**:
- Automated incident detection (SIEM integration)
- Incident classification playbook aligned with Example Regulatory Authority categories
- Automated notification workflow to Example Regulatory Authority portal
- Incident register with timestamps

**Example MNO Example**:
```yaml
# Incident response automation
incidents:
  detection:
    - SIEM: Elasticsearch with correlation rules
    - Alerting: Prometheus AlertManager
  classification:
    - Unauthorized access → Example Regulatory Authority Category 1
    - Data breach → Example Regulatory Authority Category 2
    - Service disruption → Example Regulatory Authority Category 3
  reporting:
    - Example Regulatory Authority notification: Automated within 24 hours
    - Internal escalation: Immediate to ODA Platform Team
```

---

#### CSA-03: Chief Information Security Officer (CISO) Appointment

**Legal Requirement**:
- Appoint a qualified CISO responsible for cybersecurity governance
- CISO must have direct reporting line to senior management

**Maps to NIST CSF 2.0**:
- **GV.OV-01 (Oversight)**: Establish cybersecurity risk management accountability
- **GV.RM-02 (Roles and Responsibilities)**: Define roles for cybersecurity

**NIST CSF 2.0 Reference**: `SECURITY-NIST-CSF-MAPPING.md:GV.OV-01, GV.RM-02`

**Organizational Requirements**:
- CISO role definition with cybersecurity responsibilities
- Reporting structure to C-suite or Board
- Authority to enforce security policies
- Access to adequate resources and budget

---

#### CSA-04: Cybersecurity Audit

**Legal Requirement**:
- Annual independent cybersecurity audit
- Audit scope must cover technical, operational, and governance controls
- Auditor must be registered with Example Regulatory Authority

**Maps to NIST CSF 2.0**:
- **ID.IM-01 (Improvement)**: Identify improvements from evaluations
- **GV.OV-01 (Oversight)**: Independent verification of controls

**NIST CSF 2.0 Reference**: `SECURITY-NIST-CSF-MAPPING.md:ID.IM-01, GV.OV-01`

**Audit Scope**:
- Identity and access management (OAuth2/OIDC implementation)
- Encryption controls (data at rest and in transit)
- Network security (segmentation, firewalls, IDS/IPS)
- Monitoring and logging (SIEM, log retention)
- Incident response capabilities
- Business continuity and disaster recovery
- Third-party risk management

**Deliverables**:
- Audit report with findings and recommendations
- Management response to audit findings
- Remediation plan with timelines
- Example Regulatory Authority submission (if required)

---

#### CSA-05: Supply Chain Security

**Legal Requirement**:
- Assess and manage cybersecurity risks in supply chain
- Due diligence on third-party vendors
- Contractual cybersecurity obligations

**Maps to NIST CSF 2.0**:
- **GV.SC-01 (Supply Chain Risk Management)**: Manage supply chain cybersecurity risks
- **ID.AM-01 (Asset Management)**: Maintain inventory including third-party services

**NIST CSF 2.0 Reference**: `SECURITY-NIST-CSF-MAPPING.md:GV.SC-01, ID.AM-01`

**ODA Implementation**:
- **Component Sources**: Official TM Forum ODA repositories only
- **Container Registry**: Verified and scanned images
- **Dependency Management**: Automated vulnerability scanning (Trivy, Grype)
- **SBOM**: Software Bill of Materials for all components
- **Vendor Assessment**: Security questionnaires for cloud providers (AWS, Azure, GCP)

**Third-Party Services**:
- IDM Platform IDM: Open-source with security audit trail
- Apache Kafka: CNCF graduated project with security standards
- Istio Service Mesh: CNCF project with CVE monitoring
- Prometheus/Grafana: CNCF graduated projects

---

### Personal Data Protection Act (PDPA) 2024 Compliance

#### PDPA-P4: Security Principle

**Legal Requirement**:
- Implement reasonable security measures to protect personal data
- Protect against unauthorized access, collection, use, disclosure, modification, disposal
- Implement technical and organizational measures

**Maps to NIST CSF 2.0**:
- **PR.DS-01 (Data Security)**: Protect data at rest
- **PR.DS-02 (Data Security)**: Protect data in transit
- **PR.AA (Access Control)**: Manage identity and access
- **PR.DS-05 (Data Lifecycle)**: Manage data throughout lifecycle

**NIST CSF 2.0 Reference**: `SECURITY-NIST-CSF-MAPPING.md:PR.DS-01, PR.DS-02, PR.AA-01 to PR.AA-06`

**Technical Safeguards**:
- **Encryption**: AES-256 at rest, TLS 1.3 in transit
- **Access Control**: OAuth2/OIDC with MFA, RBAC via TMF672
- **Network Security**: mTLS service mesh, network policies
- **Monitoring**: Real-time access logging, anomaly detection
- **Secrets Management**: HashiCorp Vault or Kubernetes Secrets with encryption

**Organizational Safeguards**:
- Access control policies
- Security awareness training
- Background checks for privileged access
- Incident response procedures
- Data classification guidelines

---

#### PDPA-P2: Notice and Choice Principle

**Legal Requirement**:
- Inform data subjects about data collection purposes
- Obtain explicit consent before processing personal data
- Provide opt-out mechanisms

**Maps to NIST CSF 2.0**:
- **GV.RM-02 (Roles and Responsibilities)**: Define data processing roles
- **PR.DS-05 (Data Lifecycle)**: Manage consent throughout lifecycle

**NIST CSF 2.0 Reference**: `SECURITY-NIST-CSF-MAPPING.md:GV.RM-02, PR.DS-05`

**ODA Implementation**:
- **Party Domain (TMFC028)**: Customer consent management
- **Consent Records**: Stored with timestamp, purpose, expiry
- **Consent API**: TMF632 Party Management API includes consent attributes
- **Withdrawal Mechanism**: Self-service portal for consent withdrawal

**Example MNO Example**:
```
Party Management API (TMF632):
  /individual/{id}/consent
    - purpose: "Marketing communications"
    - status: "granted" | "withdrawn"
    - grantedAt: "2024-12-08T10:30:00Z"
    - expiresAt: "2025-12-08T10:30:00Z"
```

---

#### PDPA-P5: Retention Principle

**Legal Requirement**:
- Retain personal data only as long as necessary
- Dispose of data when no longer required
- Document retention periods

**Maps to NIST CSF 2.0**:
- **PR.DS-05 (Data Lifecycle)**: Manage data throughout lifecycle including deletion
- **GV.RM-01 (Risk Management)**: Establish data retention objectives

**NIST CSF 2.0 Reference**: `SECURITY-NIST-CSF-MAPPING.md:PR.DS-05`

**Implementation**:
- **Retention Policies**: Defined per data category
  - Customer profiles: 7 years post-relationship
  - Transaction logs: 7 years (financial regulations)
  - Marketing data: Until consent withdrawn
  - Audit logs: 90 days (operational), 7 years (compliance)
- **Automated Deletion**: Scheduled jobs delete expired data
- **Anonymization**: Alternative to deletion for analytics

**Example MNO Configuration**:
```yaml
dataRetention:
  customerData:
    retentionPeriod: 7years
    postTermination: true
  auditLogs:
    operational: 90days
    compliance: 7years
  marketingConsent:
    untilWithdrawn: true
  backups:
    retention: 30days
```

---

#### PDPA-2024: Mandatory Data Breach Notification

**Legal Requirement** (2024 Amendment):
- Notify Personal Data Protection Commissioner within 72 hours of becoming aware of breach
- Notify affected individuals without undue delay
- Document all data breaches

**Maps to NIST CSF 2.0**:
- **RS.MA-01 (Incident Management)**: Execute incident response plan
- **RC.CO-01 (Recovery Communication)**: Manage public relations
- **DE.AE-04 (Impact Assessment)**: Understand scope of adverse events

**NIST CSF 2.0 Reference**: `SECURITY-NIST-CSF-MAPPING.md:RS.MA-01, RC.CO-01, DE.AE-04`

**Breach Response Workflow**:
1. **Detection** (DE.CM): SIEM alerts on suspicious data access
2. **Assessment** (DE.AE): Determine if personal data compromised
3. **Containment** (RS.MI): Isolate affected systems
4. **Notification** (RC.CO):
   - Commissioner notification within 72 hours
   - Individual notification (email, portal, public notice)
5. **Remediation** (RC.RP): Implement corrective actions
6. **Documentation**: Breach register with all details

**Notification Template**:
```
Subject: Important Security Notification - Data Breach

Dear [Customer],

We are writing to inform you of a security incident that may have
affected your personal data.

Incident Details:
- Date Discovered: [Date]
- Data Affected: [Categories]
- Number of Individuals: [Count]
- Root Cause: [Summary]

Actions Taken:
- [Containment measures]
- [Remediation steps]

Your Rights:
- File complaint with Personal Data Protection Commissioner
- Request details of your affected data

Contact: security@example-mno.com.my | 1800-88-xxxx
```

---

#### PDPA-P7: Access Principle

**Legal Requirement**:
- Data subjects have right to access their personal data
- Data subjects can request correction of inaccurate data
- Respond to access requests within 21 days

**Maps to NIST CSF 2.0**:
- **PR.DS-05 (Data Lifecycle)**: Enable data subject rights
- **ID.AM-01 (Asset Management)**: Know what data exists for retrieval

**NIST CSF 2.0 Reference**: `SECURITY-NIST-CSF-MAPPING.md:PR.DS-05, ID.AM-01`

**ODA Implementation**:
- **Self-Service Portal**: Customer can view/download their data
- **Data Export API**: TMF632 Party Management API
- **Data Correction**: Update API endpoints with audit trail
- **Verification**: Identity verification before data release

**Example MNO Example**:
```
Data Subject Access Request (DSAR) Portal:
  GET /individual/{id}/data-export
    - Authentication: OAuth2 with MFA
    - Response: JSON/PDF with all personal data
    - Includes: Profile, orders, transactions, consents, audit trail
    - Delivered: Secure download or encrypted email
    - SLA: Within 21 days
```

---

### Artificial Intelligence Governance and Ethics (AIGE) Framework

#### AIGE-1: Accountability

**Framework Requirement**:
- Assign clear accountability for AI system outcomes
- Establish AI governance structure
- Document AI system lifecycle
- Implement AI risk management

**Maps to NIST CSF 2.0**:
- **GV.OV-01 (Oversight)**: Establish accountability and authority
- **GV.RM-02 (Roles and Responsibilities)**: Define AI governance roles
- **ID.RA (Risk Assessment)**: AI-specific risk assessment

**NIST CSF 2.0 Reference**: `SECURITY-NIST-CSF-MAPPING.md:GV.OV-01, GV.RM-02, ID.RA-01`

**ODA Intelligence Domain Implementation**:

**AI System Inventory**:
- Predictive analytics for network optimization
- Churn prediction models
- Fraud detection algorithms
- Recommendation engines
- Chatbots and virtual assistants

**Accountability Structure**:
```
AI Governance Committee
├── Chief AI Officer (CAIO) - Overall accountability
├── Data Science Lead - Model development and validation
├── CISO - AI security and privacy
├── Compliance Officer - Regulatory compliance
└── Ethics Board - Ethical review of AI systems
```

**AI System Documentation**:
- **Model Card**: Model purpose, training data, performance metrics, limitations
- **Risk Assessment**: Bias analysis, privacy impact, security vulnerabilities
- **Approval**: Governance committee sign-off before production deployment
- **Monitoring**: Continuous performance and bias monitoring
- **Incident Response**: AI-specific incident playbooks

**Example: Churn Prediction Model**:
```yaml
aiSystem:
  name: "Customer Churn Prediction"
  owner: "Data Science Team"
  accountability: "Chief AI Officer"

  dataProcessing:
    personalData: true
    sensitiveData: false
    dataMinimization: true
    retention: "Model training: 2 years, Predictions: 90 days"

  fairness:
    protectedAttributes: ["age", "gender", "location"]
    biasTestingFrequency: "Quarterly"
    fairnessMetrics: ["Demographic parity", "Equal opportunity"]

  transparency:
    explainability: "SHAP values provided"
    customerNotification: true
    optOut: true

  humanOversight:
    automationLevel: "Recommendation (not decision)"
    humanReview: "High-risk churn predictions reviewed by retention team"
```

---

#### AIGE-2: Transparency

**Framework Requirement**:
- Explain AI decisions to affected individuals
- Document AI system logic and decision criteria
- Provide meaningful information about automated processing

**Maps to NIST CSF 2.0**:
- **GV.RM-02 (Roles and Responsibilities)**: Communicate AI processing
- **PR.DS-05 (Data Lifecycle)**: Transparency in data processing

**NIST CSF 2.0 Reference**: `SECURITY-NIST-CSF-MAPPING.md:GV.RM-02, PR.DS-05`

**Implementation**:
- **Explainable AI (XAI)**: SHAP, LIME for model interpretability
- **Customer Notification**: "This decision was automated" disclosure
- **Decision Records**: Log all AI decisions with explanations
- **Right to Human Review**: Customers can request human override

**Example MNO Customer Portal**:
```
Offer Recommendation:
  "Based on your usage patterns, we recommend Plan XYZ"

  [?] Why this recommendation?
      - Your average monthly data usage: 50GB
      - Your current plan: 30GB
      - Predicted next month usage: 55GB
      - This recommendation can save you RM 20/month

  [⚠] This recommendation was generated by our AI system.
      You can request human review: Contact 1800-88-xxxx
```

---

#### AIGE-3: Fairness

**Framework Requirement**:
- Ensure AI systems do not discriminate
- Test for bias across protected attributes
- Mitigate identified biases
- Document fairness assessments

**Maps to NIST CSF 2.0**:
- **ID.RA-03 (Risk Assessment)**: Identify threats including bias
- **PR.DS-05 (Data Lifecycle)**: Fair data processing

**NIST CSF 2.0 Reference**: `SECURITY-NIST-CSF-MAPPING.md:ID.RA-03`

**Fairness Testing**:
- **Pre-deployment**: Bias audit before production
- **Ongoing Monitoring**: Quarterly fairness assessments
- **Metrics**: Disparate impact ratio, equal opportunity difference
- **Remediation**: Model retraining if bias detected

**Protected Attributes** (Example Countryn Context):
- Race/ethnicity
- Gender
- Age
- Location (urban vs rural)
- Religion
- Disability status

**Example: Credit Scoring Model**:
```python
# Fairness constraint: Approval rate parity
fairness_metrics = {
    "demographic_parity": 0.95,  # Within 5% across groups
    "equal_opportunity": 0.90,   # Within 10% for qualified applicants
}

# If metrics violated:
if not meets_fairness_threshold(model, fairness_metrics):
    trigger_bias_mitigation()
    notify_ai_governance_committee()
    suspend_model_deployment()
```

---

#### AIGE-4: Safety and Reliability

**Framework Requirement**:
- Ensure AI systems are robust and reliable
- Implement safeguards against adversarial attacks
- Test AI systems thoroughly before deployment
- Monitor AI system performance continuously

**Maps to NIST CSF 2.0**:
- **PR.IR-02 (Infrastructure Resilience)**: Resilience requirements for AI systems
- **PR.DS-08 (Data Integrity)**: Integrity checking for AI models
- **DE.CM-09 (Continuous Monitoring)**: Monitor AI system performance

**NIST CSF 2.0 Reference**: `SECURITY-NIST-CSF-MAPPING.md:PR.IR-02, PR.DS-08, DE.CM-09`

**AI Security Controls**:

**Model Security**:
- **Adversarial Testing**: Test against adversarial examples
- **Model Versioning**: Track all model versions with Git/MLflow
- **Model Signing**: Cryptographic signatures on production models
- **Input Validation**: Sanitize inputs to prevent poisoning attacks
- **Output Validation**: Check for unexpected outputs

**Monitoring**:
- **Performance Drift**: Alert if accuracy drops below threshold
- **Data Drift**: Detect distribution shifts in input data
- **Concept Drift**: Detect changes in data-label relationships
- **Anomaly Detection**: Flag unusual predictions

**Example MNO Intelligence Domain**:
```yaml
aiMonitoring:
  performance:
    metric: "Accuracy"
    baseline: 0.92
    threshold: 0.85
    alerting: "Slack + PagerDuty"

  dataDrift:
    method: "KS-test"
    frequency: "Daily"
    pValue: 0.05

  adversarialTesting:
    frequency: "Quarterly"
    testSuite: "CLEVERHANS"

  modelSigning:
    algorithm: "RSA-4096"
    keyManagement: "HashiCorp Vault"
```

---

#### AIGE-5: Privacy

**Framework Requirement**:
- Protect personal data in AI training and inference
- Implement privacy-preserving AI techniques
- Minimize data collection for AI
- Comply with PDPA requirements

**Maps to NIST CSF 2.0**:
- **PR.DS-01 (Data Security)**: Protect data at rest (including training data)
- **PR.DS-02 (Data Security)**: Protect data in transit
- **PR.DS-05 (Data Lifecycle)**: Privacy in data processing

**NIST CSF 2.0 Reference**: `SECURITY-NIST-CSF-MAPPING.md:PR.DS-01, PR.DS-02, PR.DS-05`

**Privacy-Preserving AI Techniques**:

**Data Minimization**:
- Collect only necessary features for AI training
- Feature selection to remove unnecessary personal data
- Aggregation instead of individual-level data when possible

**Anonymization & Pseudonymization**:
- K-anonymity for training datasets
- Differential privacy for aggregate statistics
- Tokenization of PII in training pipelines

**Federated Learning** (Future Enhancement):
- Train models on decentralized data
- No raw data leaves customer devices
- Privacy-preserving for network analytics

**Secure Multi-Party Computation** (Future Enhancement):
- Collaborative model training without data sharing
- Useful for multi-operator insights

**Example MNO AI Privacy Controls**:
```yaml
aiPrivacy:
  trainingData:
    anonymization: "K-anonymity (k=10)"
    pseudonymization: true
    encryption: "AES-256"
    accessControl: "Data science team only"

  differentialPrivacy:
    enabled: true
    epsilon: 1.0
    delta: 1e-5

  dataRetention:
    trainingData: "2 years post model retirement"
    predictions: "90 days"
    modelArtifacts: "5 years (audit)"
```

---

#### AIGE-6: Human Oversight

**Framework Requirement**:
- Maintain meaningful human oversight of AI systems
- Define automation levels (human-in-the-loop, human-on-the-loop)
- Enable human override of AI decisions
- Document oversight procedures

**Maps to NIST CSF 2.0**:
- **GV.OV-01 (Oversight)**: Human accountability for AI
- **RS.MI-02 (Incident Mitigation)**: Human intervention for AI incidents

**NIST CSF 2.0 Reference**: `SECURITY-NIST-CSF-MAPPING.md:GV.OV-01, RS.MI-02`

**Automation Levels**:

**Level 1: Recommendation**
- AI suggests, human decides
- Use case: Product recommendations, churn predictions
- Human oversight: Retention team reviews high-risk predictions

**Level 2: Automated with Human Review**
- AI decides, human reviews exceptions
- Use case: Fraud detection (auto-block suspicious transactions)
- Human oversight: Fraud analyst reviews blocked transactions daily

**Level 3: Fully Automated**
- AI decides autonomously for low-risk scenarios
- Use case: Spam filtering, network optimization
- Human oversight: Weekly performance reviews, monthly audits

**High-Risk Decisions Requiring Human Review**:
- Contract termination
- Credit limit reduction
- Service suspension
- Pricing changes affecting customer significantly

**Example MNO Oversight Structure**:
```yaml
aiOversight:
  churnPrediction:
    automationLevel: "Recommendation"
    humanReview: "Always"
    reviewedBy: "Retention Specialists"

  fraudDetection:
    automationLevel: "Automated with review"
    humanReview: "Exceptions only"
    reviewedBy: "Fraud Analysts"
    reviewFrequency: "Daily"

  networkOptimization:
    automationLevel: "Fully automated"
    humanReview: "Performance monitoring"
    reviewedBy: "Network Operations"
    reviewFrequency: "Weekly"

  overrideProcess:
    customerRequest: true
    escalation: "Customer can request human review"
    sla: "24 hours"
```

---

## Compliance Matrix Summary

| Example Countryn Requirement | NIST CSF 2.0 Function | NIST CSF 2.0 Category | Implementation Reference |
|----------------------|------------------------|------------------------|--------------------------|
| **Example Cybersecurity Act** |
| Risk Management Framework | GOVERN, IDENTIFY | GV.RM, ID.RA | SECURITY-NIST-CSF-MAPPING.md |
| Incident Reporting (24h) | DETECT, RESPOND | DE.AE, RS.MA | SECURITY-NIST-CSF-MAPPING.md |
| CISO Appointment | GOVERN | GV.OV, GV.RM | SECURITY-NIST-CSF-MAPPING.md |
| Annual Audit | IDENTIFY, GOVERN | ID.IM, GV.OV | SECURITY-NIST-CSF-MAPPING.md |
| Supply Chain Security | GOVERN, IDENTIFY | GV.SC, ID.AM | SECURITY-NIST-CSF-MAPPING.md |
| **Example Data Protection Act** |
| Security Principle | PROTECT | PR.DS, PR.AA | SECURITY-NIST-CSF-MAPPING.md |
| Notice & Consent | GOVERN, PROTECT | GV.RM, PR.DS | ARCHITECTURE.md:104-122 |
| Data Retention | PROTECT | PR.DS-05 | example-mno/tmfc035-values.yaml:257-260 |
| Breach Notification (72h) | RESPOND, RECOVER | RS.MA, RC.CO | SECURITY-NIST-CSF-MAPPING.md |
| Access Rights (21 days) | PROTECT | PR.DS-05 | ARCHITECTURE.md:104-122 (TMF632) |
| **AIGE Framework** |
| Accountability | GOVERN | GV.OV, GV.RM | SECURITY-NIST-CSF-MAPPING.md |
| Transparency | GOVERN, PROTECT | GV.RM, PR.DS | Intelligence Domain (planned) |
| Fairness | IDENTIFY, PROTECT | ID.RA, PR.DS | Intelligence Domain (planned) |
| Safety & Reliability | PROTECT, DETECT | PR.IR, DE.CM | SECURITY-NIST-CSF-MAPPING.md |
| Privacy | PROTECT | PR.DS-01, PR.DS-02, PR.DS-05 | SECURITY-NIST-CSF-MAPPING.md |
| Human Oversight | GOVERN, RESPOND | GV.OV, RS.MI | Intelligence Domain (planned) |

---

## Compliance Evidence Repository

### For Example Cybersecurity Act Audits

**Location**: Example MNO deployment environment

**Evidence Collection**:
1. **Risk Management**:
   - Risk register: `s3://example-mno-compliance/risk-register/`
   - Risk assessments: Quarterly reports
   - Board presentations: Monthly cybersecurity updates

2. **Incident Reports**:
   - Incident register: Elasticsearch index `example-mno-incidents-*`
   - Example Regulatory Authority submissions: `s3://example-mno-compliance/nacsa-reports/`
   - Post-incident reviews: Confluence space

3. **Audit Reports**:
   - Annual audit: `s3://example-mno-compliance/audits/`
   - Remediation tracking: Jira project "AUDIT-REMEDIATION"

4. **Supply Chain**:
   - Vendor assessments: `s3://example-mno-compliance/vendors/`
   - SBOMs: `s3://example-mno-artifacts/sbom/`
   - Vulnerability scans: Trivy reports in CI/CD

### For Example Data Protection Act Compliance

**Evidence Collection**:
1. **Consent Records**:
   - API: TMF632 Party Management `/individual/{id}/consent`
   - Database: PostgreSQL `party_consent` table
   - Audit trail: All consent changes logged

2. **Data Breach Register**:
   - Location: `s3://example-mno-compliance/breach-register/`
   - Format: JSON with Commissioner notifications
   - Retention: Permanent

3. **DSAR Processing**:
   - Portal: Customer self-service
   - Tracking: Jira tickets with SLA monitoring
   - Reports: Monthly DSAR metrics

4. **Retention Compliance**:
   - Automated deletion logs: CloudWatch Logs
   - Retention policy documentation: Confluence
   - Audit of deleted data: Immutable log in S3

### For AIGE Framework Compliance

**Evidence Collection**:
1. **AI System Inventory**:
   - Model registry: MLflow at `https://mlflow.example-mno.internal`
   - Model cards: Git repository `example-mno-ai/model-cards`
   - Governance approvals: Confluence

2. **Fairness Assessments**:
   - Bias testing reports: Quarterly
   - Fairness metrics: Prometheus `ai_fairness_*` metrics
   - Remediation actions: Jira project "AI-FAIRNESS"

3. **AI Incident Register**:
   - Location: `s3://example-mno-compliance/ai-incidents/`
   - Categories: Bias, performance degradation, privacy breach
   - Root cause analysis: Documented in Confluence

---

## Gap Analysis and Roadmap

### Current Compliance Status (December 2024)

| Requirement Area | Status | Gap |
|------------------|--------|-----|
| **Example Cybersecurity Act** |
| Risk Management Framework | ✅ Compliant | None - NIST CSF 2.0 implemented |
| Incident Reporting | ✅ Compliant | Automate Example Regulatory Authority portal integration |
| CISO Appointment | ✅ Compliant | None - Organizational structure in place |
| Annual Audit | ✅ Compliant | Schedule 2025 audit |
| Supply Chain Security | 🟡 Partial | Formalize vendor assessment process |
| **Example Data Protection Act** |
| Security Principle | ✅ Compliant | None - Comprehensive controls |
| Notice & Consent | ✅ Compliant | None - TMF632 implemented |
| Data Retention | ✅ Compliant | None - Automated deletion |
| Breach Notification | 🟡 Partial | Automate Commissioner notification |
| Access Rights | ✅ Compliant | None - Self-service portal |
| **AIGE Framework** |
| Accountability | 🟡 Partial | Establish AI Governance Committee |
| Transparency | 🟡 Partial | Implement XAI for customer-facing AI |
| Fairness | 🟡 Partial | Formalize bias testing procedures |
| Safety & Reliability | ✅ Compliant | Model monitoring implemented |
| Privacy | ✅ Compliant | Data minimization, encryption |
| Human Oversight | 🟡 Partial | Document automation levels |

### 2025 Compliance Roadmap

**Q1 2025 (January - March)**:
- [ ] Automate Example Regulatory Authority incident reporting portal integration
- [ ] Formalize supply chain vendor assessment process
- [ ] Establish AI Governance Committee
- [ ] Document AI automation levels and oversight procedures

**Q2 2025 (April - June)**:
- [ ] Implement automated PDPA Commissioner breach notification
- [ ] Deploy explainable AI (XAI) for customer-facing models
- [ ] Conduct first AIGE framework assessment
- [ ] Complete annual cybersecurity audit

**Q3 2025 (July - September)**:
- [ ] Formalize AI fairness testing procedures
- [ ] Implement quarterly bias audits
- [ ] Enhance AI model monitoring dashboards
- [ ] Third-party penetration testing

**Q4 2025 (October - December)**:
- [ ] Review and update all compliance evidence
- [ ] Prepare for 2026 Cyber Security Act audit
- [ ] AIGE framework certification (if available)
- [ ] Year-end compliance reporting to Board

---

## Appendices

### Appendix A: Regulatory Authority Contacts

**National Cyber Security Agency (Example Regulatory Authority)**:
- Website: https://www.nacsa.gov.my
- Incident Reporting Portal: https://incidents.nacsa.gov.my
- Email: incidents@nacsa.gov.my
- Hotline: 1-300-88-2999

**Personal Data Protection Department**:
- Website: https://www.pdp.gov.my
- Email: pdp@kpdnhep.gov.my
- Complaint Portal: https://eaduan.pdp.gov.my
- Phone: +603-8882 5555

**Example Country Digital Economy Corporation (Example Digital Authority)**:
- Website: https://mdec.my
- AIGE Framework: https://mdec.my/aige
- Email: aige@mdec.com.my

### Appendix B: Compliance Checklist

**Annual Compliance Activities**:
- [ ] Cybersecurity risk assessment (Quarterly)
- [ ] Cybersecurity audit (Annual)
- [ ] PDPA compliance audit (Annual)
- [ ] AI fairness assessment (Quarterly)
- [ ] Third-party vendor review (Annual)
- [ ] Incident response drill (Semi-annual)
- [ ] Business continuity test (Annual)
- [ ] Security awareness training (Quarterly)
- [ ] Board cybersecurity reporting (Monthly)

### Appendix C: Referenced Documents

**Internal Documentation**:
- ODA Reference Architecture: `docs/ARCHITECTURE.md`
- NIST CSF 2.0 Mapping: `docs/SECURITY-NIST-CSF-MAPPING.md`
- Example MNO Component Configuration: `components/example-mno/`

**External Standards**:
- NIST Cybersecurity Framework 2.0: https://www.nist.gov/cyberframework
- ISO 27001:2022: Information Security Management
- ISO 27701:2019: Privacy Information Management

**Example Countryn Legislation**:
- Example Cybersecurity Act (Act XXX of 2024)
- Personal Data Protection Act 2010 (Act 709)
- Communications and Multimedia Act 1998 (Act 588)

---

**Document Version**: 1.0
**Last Updated**: December 2024
**Next Review**: June 2025
**Owner**: Example MNO CISO Office
**Approved By**: Example MNO Chief Information Security Officer
