# Breach Simulation: Bottom-Up Compliance Flow

## Overview

This document demonstrates how security incidents flow bottom-up through the three-layer compliance model:

```
Layer 3: Code Change (Implementation)
            ↓ triggers detection
Layer 2: NIST CSF 2.0 Control Violation
            ↓ triggers regulatory obligation
Layer 1: Example Countryn Legal Requirement (Example Cybersecurity Act, Example DPA)
```

Each simulation shows:
1. **Root Cause**: Code change or configuration drift
2. **Detection**: Monitoring system that catches the issue
3. **NIST CSF 2.0 Mapping**: Which controls are violated
4. **Regulatory Trigger**: Example Countryn legal obligations activated
5. **Alert Workflow**: Automated notifications and escalations
6. **Response Procedure**: Incident response playbook

---

## Simulation 1: Unauthorized API Access (Authentication Bypass)

### Scenario Description

A developer accidentally commits code that disables JWT token validation in the TMFC035 Permissions Management component, allowing unauthenticated access to user role management APIs.

---

### Layer 3: Code Change (Implementation)

**File**: `components/example-mno/tmfc035-deployment/middleware/auth.js`

**Before (Secure)**:
```javascript
// JWT validation middleware
const validateToken = async (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];

  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }

  try {
    const decoded = await jwt.verify(token, process.env.JWT_PUBLIC_KEY);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
};

// Apply to all routes
app.use('/tmf-api/userRolePermission/*', validateToken);
app.use('/tmf-api/partyRoleManagement/*', validateToken);
```

**After (Vulnerable) - Committed by mistake**:
```javascript
// JWT validation middleware
const validateToken = async (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];

  // TODO: Remove this bypass after testing
  if (process.env.NODE_ENV === 'development') {
    console.log('[DEV MODE] Bypassing authentication');
    req.user = { id: 'dev-user', role: 'admin' };
    return next();
  }

  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }

  try {
    const decoded = await jwt.verify(token, process.env.JWT_PUBLIC_KEY);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
};

// Apply to all routes
app.use('/tmf-api/userRolePermission/*', validateToken);
app.use('/tmf-api/partyRoleManagement/*', validateToken);
```

**Problem**: The code includes a development bypass that checks `NODE_ENV === 'development'`. If this environment variable is accidentally set in production, authentication is completely bypassed.

**Deployment Event**:
```yaml
# Kubernetes deployment with wrong environment variable
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-mno-permissions-management
spec:
  template:
    spec:
      containers:
      - name: tmfc035
        image: example-mno/tmfc035:v1.2.3
        env:
        - name: NODE_ENV
          value: "development"  # ❌ WRONG! Should be "production"
```

**Timeline**:
- **T+0 minutes**: Deployment with NODE_ENV=development goes live
- **T+2 minutes**: First unauthenticated API call succeeds
- **T+5 minutes**: Prometheus detects anomaly in API metrics
- **T+7 minutes**: SIEM correlation rule triggers alert
- **T+10 minutes**: Security team receives PagerDuty alert

---

### Layer 2: NIST CSF 2.0 Detection

#### Detection Point 1: Continuous Monitoring (DE.CM-09)

**Prometheus Alert**:
```yaml
# Alert: Authentication bypass detected
groups:
- name: authentication
  rules:
  - alert: AuthenticationBypassDetected
    expr: |
      rate(http_requests_total{
        endpoint=~"/tmf-api/(userRolePermission|partyRoleManagement).*",
        status_code=~"2..",
        auth_header_present="false"
      }[5m]) > 0
    for: 2m
    labels:
      severity: critical
      nist_function: DETECT
      nist_category: DE.CM-09
      csa_category: "Unauthorized Access"
      pdpa_breach: "potential"
    annotations:
      summary: "Unauthenticated API access detected on {{ $labels.endpoint }}"
      description: "{{ $value }} requests per second without authentication"
      runbook: "https://wiki.example-mno.internal/runbooks/auth-bypass"
```

**Alert Triggered**:
```json
{
  "alertname": "AuthenticationBypassDetected",
  "severity": "critical",
  "endpoint": "/tmf-api/userRolePermission/v4/userRole",
  "rate": "0.5 req/s",
  "nist_function": "DETECT",
  "nist_category": "DE.CM-09",
  "csa_category": "Unauthorized Access",
  "pdpa_breach": "potential",
  "timestamp": "2024-12-08T15:30:00Z"
}
```

#### Detection Point 2: Adverse Event Analysis (DE.AE-02)

**Elasticsearch SIEM Correlation Rule**:
```json
{
  "rule_id": "auth-bypass-correlation",
  "rule_name": "Authentication Bypass Pattern Detection",
  "description": "Detects successful API calls without JWT token validation",
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "api.endpoint": "/tmf-api/userRolePermission/v4/userRole"
          }
        },
        {
          "match": {
            "http.status_code": 200
          }
        },
        {
          "bool": {
            "must_not": {
              "exists": {
                "field": "jwt.token_id"
              }
            }
          }
        }
      ]
    }
  },
  "nist_controls": ["DE.AE-02", "DE.CM-01"],
  "csa_incident_type": "Unauthorized Access",
  "pdpa_data_category": "Personal Data - User Roles",
  "severity": "high",
  "actions": [
    {
      "type": "alert",
      "target": "security-team-pagerduty"
    },
    {
      "type": "create_incident",
      "target": "incident-management-system"
    },
    {
      "type": "webhook",
      "target": "https://api.example-mno.internal/security/incident-webhook"
    }
  ]
}
```

**SIEM Alert Generated**:
```json
{
  "incident_id": "INC-2024-1208-001",
  "type": "Authentication Bypass",
  "severity": "HIGH",
  "nist_controls_violated": ["PR.AA-03", "PR.AA-04", "DE.CM-09"],
  "detection_time": "2024-12-08T15:30:00Z",
  "affected_component": "TMFC035 Permissions Management",
  "affected_api": "TMF672 User Role Permission",
  "requests_without_auth": 15,
  "time_window": "5 minutes",
  "source_ips": ["10.20.30.45", "10.20.30.46"],
  "user_agents": ["curl/7.68.0", "PostmanRuntime/7.29.2"]
}
```

#### NIST CSF 2.0 Control Violations

| NIST Category | Control | Violation | Evidence |
|---------------|---------|-----------|----------|
| **PR.AA-03** | Users are authenticated | ❌ Failed | API requests accepted without JWT tokens |
| **PR.AA-04** | Identity assertions are protected | ❌ Failed | Token validation bypassed in code |
| **DE.CM-09** | Computing hardware and software are monitored | ✅ Working | Prometheus detected anomaly |
| **DE.AE-02** | Adverse events are analyzed | ✅ Working | SIEM correlated authentication bypass pattern |

---

### Layer 1: Example Countryn Regulatory Obligations Triggered

#### Cyber Security Act 2024: Incident Reporting (CSA-02)

**Obligation**: Report cybersecurity incidents to Example Regulatory Authority within 24 hours

**Triggered Because**:
- ✅ Incident Type: Unauthorized Access (CSA Category 1)
- ✅ Affected System: Critical National Information Infrastructure (telecommunications)
- ✅ Severity: High (authentication controls bypassed)
- ✅ Data Exposure Risk: User roles and permissions (potential PII access)

**Example Regulatory Authority Notification Required**:
```yaml
nacsa_notification:
  incident_category: "Unauthorized Access"
  affected_system: "ODA Permissions Management (TMFC035)"
  detection_time: "2024-12-08T15:30:00+08:00"
  notification_deadline: "2024-12-09T15:30:00+08:00"  # 24 hours
  initial_assessment:
    severity: "High"
    root_cause: "Configuration error - NODE_ENV set to development"
    affected_components: "TMF672 User Role Permission API"
    data_exposure_risk: "Medium"
    services_impacted: "None (no customer service disruption)"
  containment_actions:
    - "Rolled back deployment to previous version"
    - "Corrected NODE_ENV to production"
    - "Revoked all API sessions in past 10 minutes"
    - "Implemented additional deployment validation checks"
```

**Automated Workflow**:
```javascript
// Incident management system webhook handler
async function handleSecurityIncident(incident) {
  // Check if Example Regulatory Authority reporting required
  const requiresExample Regulatory Authority = (
    incident.csa_category === 'Unauthorized Access' &&
    incident.severity in ['HIGH', 'CRITICAL']
  );

  if (requiresExample Regulatory Authority) {
    // Create Example Regulatory Authority report draft
    const nacsaReport = {
      incident_id: incident.id,
      category: incident.csa_category,
      detection_time: incident.detection_time,
      deadline: addHours(incident.detection_time, 24),
      status: 'DRAFT'
    };

    await db.nacsaReports.insert(nacsaReport);

    // Alert CISO
    await pagerduty.trigger({
      routing_key: process.env.CISO_PAGERDUTY_KEY,
      event_action: 'trigger',
      payload: {
        summary: `Example Regulatory Authority Reporting Required: ${incident.type}`,
        severity: 'critical',
        source: 'ODA Security',
        custom_details: {
          incident_id: incident.id,
          deadline: nacsaReport.deadline,
          csa_category: incident.csa_category
        }
      }
    });

    // Create Jira ticket for Example Regulatory Authority report
    await jira.createIssue({
      project: 'SEC',
      issuetype: 'Incident',
      summary: `Example Regulatory Authority Report Required: ${incident.type}`,
      description: `Incident ID: ${incident.id}\nDeadline: ${nacsaReport.deadline}`,
      priority: 'Highest',
      labels: ['Example Regulatory Authority', 'CSA2024', 'regulatory'],
      duedate: nacsaReport.deadline
    });
  }
}
```

#### Example DPA 2024: Data Breach Assessment (Example DPA-2024)

**Obligation**: Determine if personal data was compromised; if yes, notify Commissioner within 72 hours

**Assessment**:
```yaml
pdpa_breach_assessment:
  incident_id: "INC-2024-1208-001"
  assessment_started: "2024-12-08T15:35:00+08:00"

  personal_data_exposure:
    affected_api: "TMF672 User Role Permission"
    data_categories:
      - "User identities (names, email addresses)"
      - "Role assignments"
      - "Permission sets"
    sensitivity: "Medium"
    affected_individuals: "To be determined (log analysis ongoing)"

  exposure_analysis:
    time_window: "8 minutes (T+2 to T+10)"
    unauthorized_requests: 15
    source_ips: ["10.20.30.45", "10.20.30.46"]
    endpoints_accessed:
      - "GET /userRole (12 requests)"
      - "POST /userRole (2 requests)"
      - "PATCH /userRole/123 (1 request)"

  actual_data_accessed:
    method: "Log analysis"
    query: |
      SELECT request_id, endpoint, response_code, response_body_hash
      FROM api_logs
      WHERE timestamp BETWEEN '2024-12-08 15:32:00' AND '2024-12-08 15:40:00'
        AND endpoint LIKE '/tmf-api/userRolePermission%'
        AND jwt_token_id IS NULL
        AND response_code = 200

  preliminary_finding: "BREACH CONFIRMED"
  reason: "Unauthorized access to 47 user role records containing PII"
  affected_count: 47

  pdpa_notification_required: true
  commissioner_deadline: "2024-12-11T15:30:00+08:00"  # 72 hours
  individual_notification_required: true
```

**Automated Example DPA Workflow**:
```javascript
async function assessExample DPABreach(incident) {
  // Query database for affected records
  const affectedRecords = await db.query(`
    SELECT DISTINCT user_id, email, role_name
    FROM user_roles ur
    JOIN api_logs al ON al.response_data LIKE '%' || ur.id || '%'
    WHERE al.timestamp BETWEEN $1 AND $2
      AND al.jwt_token_id IS NULL
      AND al.endpoint LIKE '/tmf-api/userRolePermission%'
  `, [incident.start_time, incident.end_time]);

  if (affectedRecords.length > 0) {
    // Example DPA breach confirmed
    const breach = {
      incident_id: incident.id,
      breach_type: 'Unauthorized Access',
      affected_individuals: affectedRecords.length,
      data_categories: ['User Identities', 'Role Assignments'],
      commissioner_deadline: addHours(incident.detection_time, 72),
      individual_notification_required: true,
      status: 'CONFIRMED'
    };

    await db.pdpaBreaches.insert(breach);

    // Notify DPO
    await slack.postMessage({
      channel: '#data-protection',
      text: `⚠️ Example DPA Breach Confirmed: ${incident.type}`,
      blocks: [
        {
          type: 'section',
          text: {
            type: 'mrkdwn',
            text: `*Example DPA Data Breach Confirmed*\n` +
                  `Incident: ${incident.id}\n` +
                  `Affected Individuals: ${breach.affected_individuals}\n` +
                  `Commissioner Notification Deadline: ${breach.commissioner_deadline}`
          }
        },
        {
          type: 'actions',
          elements: [
            {
              type: 'button',
              text: { type: 'plain_text', text: 'View Incident' },
              url: `https://security.example-mno.internal/incidents/${incident.id}`
            },
            {
              type: 'button',
              text: { type: 'plain_text', text: 'Start Notification Process' },
              url: `https://compliance.example-mno.internal/pdpa/notify/${breach.id}`
            }
          ]
        }
      ]
    });

    // Create regulatory notification tasks
    await createRegulatoryTasks(breach);
  }

  return affectedRecords.length > 0;
}
```

---

### Alert Workflow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│ T+0: Code Deployed with NODE_ENV=development                │
└───────────────────────┬─────────────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────────────┐
│ T+2: First Unauthenticated Request Succeeds                 │
│ Layer 3: Implementation Layer                               │
└───────────────────────┬─────────────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────────────┐
│ T+5: Prometheus Alert Fires                                 │
│ Layer 2: NIST CSF DE.CM-09 (Continuous Monitoring)         │
│ Action: Alert sent to PagerDuty                            │
└───────────────────────┬─────────────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────────────┐
│ T+7: SIEM Correlation Rule Triggers                         │
│ Layer 2: NIST CSF DE.AE-02 (Adverse Event Analysis)        │
│ Action: Create incident INC-2024-1208-001                  │
└───────────────────────┬─────────────────────────────────────┘
                        │
           ┌────────────┴────────────┐
           │                         │
           ▼                         ▼
┌──────────────────────┐   ┌──────────────────────┐
│ T+10: CSA Assessment │   │ T+10: Example DPA Assessment│
│ Layer 1: Example Countryn   │   │ Layer 1: Example Countryn   │
│ Cyber Security Act   │   │ Example DPA 2024            │
│                      │   │                      │
│ Obligation:          │   │ Obligation:          │
│ - Example Regulatory Authority notification │   │ - Commissioner notify│
│   within 24 hours    │   │   within 72 hours    │
│ - Incident report    │   │ - Individual notify  │
│   to Example Regulatory Authority portal    │   │ - Breach register    │
└──────────┬───────────┘   └──────────┬───────────┘
           │                          │
           └────────────┬─────────────┘
                        │
           ┌────────────▼─────────────┐
           │ T+12: CISO Notified      │
           │ - PagerDuty High Priority│
           │ - Slack #security-crisis │
           │ - Email to executives    │
           └──────────────────────────┘
```

---

### Response Procedure

#### Immediate Actions (T+10 to T+30 minutes)

**1. Containment (NIST RS.MI-02)**:
```bash
# Rollback to previous deployment
kubectl rollout undo deployment/example-mno-permissions-management -n example-mno-oda-party

# Verify rollback successful
kubectl rollout status deployment/example-mno-permissions-management -n example-mno-oda-party

# Revoke all active sessions
redis-cli -h redis.example-mno.internal FLUSHDB  # Clear session cache

# Block suspicious source IPs
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: block-suspicious-ips
  namespace: example-mno-oda-party
spec:
  podSelector:
    matchLabels:
      app: example-mno-permissions-management
  policyTypes:
  - Ingress
  ingress:
  - from:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 10.20.30.45/32
        - 10.20.30.46/32
EOF
```

**2. Evidence Preservation (NIST RS.AN-06)**:
```bash
# Export logs for forensic analysis
kubectl logs deployment/example-mno-permissions-management \
  --since-time=2024-12-08T15:25:00Z \
  --namespace=example-mno-oda-party \
  > /evidence/INC-2024-1208-001/pod-logs.txt

# Export Elasticsearch logs
curl -X GET "https://elasticsearch.example-mno.internal/example-mno-oda-permissions-*/_search" \
  -H 'Content-Type: application/json' \
  -d'{
    "query": {
      "range": {
        "@timestamp": {
          "gte": "2024-12-08T15:25:00",
          "lte": "2024-12-08T15:45:00"
        }
      }
    }
  }' > /evidence/INC-2024-1208-001/elasticsearch-logs.json

# Take database snapshot of affected records
pg_dump -h postgres.example-mno.internal \
  -t user_roles -t role_permissions \
  --data-only \
  > /evidence/INC-2024-1208-001/database-snapshot.sql

# Store evidence immutably in S3
aws s3 sync /evidence/INC-2024-1208-001/ \
  s3://example-mno-compliance/incidents/INC-2024-1208-001/ \
  --storage-class GLACIER_IR \
  --sse AES256
```

**3. Impact Assessment (NIST DE.AE-04)**:
```sql
-- Identify affected user records
SELECT
  COUNT(DISTINCT ur.user_id) as affected_users,
  COUNT(*) as total_records,
  array_agg(DISTINCT ur.role_name) as roles_accessed
FROM user_roles ur
WHERE ur.id IN (
  SELECT (response_data::json->>'id')::int
  FROM api_logs
  WHERE timestamp BETWEEN '2024-12-08 15:32:00' AND '2024-12-08 15:40:00'
    AND endpoint LIKE '/tmf-api/userRolePermission%'
    AND jwt_token_id IS NULL
    AND response_code = 200
);

-- Result:
-- affected_users: 47
-- total_records: 47
-- roles_accessed: ['party_admin', 'party_readonly', 'role_manager']
```

#### Short-term Actions (T+30 minutes to T+24 hours)

**4. Root Cause Analysis (NIST RS.AN-03)**:
```yaml
root_cause_analysis:
  incident_id: "INC-2024-1208-001"

  root_cause: "Configuration Management Failure"

  contributing_factors:
    - "Development bypass code present in production codebase"
    - "Environment variable NODE_ENV incorrectly set to 'development'"
    - "Insufficient deployment validation (no pre-deployment smoke tests)"
    - "Lack of environment-specific configuration management"

  timeline:
    - "2024-12-05 10:00": Developer adds authentication bypass for local testing
    - "2024-12-05 15:00": Code committed without removing bypass
    - "2024-12-06 14:00": Code review approved (bypass not flagged)
    - "2024-12-08 15:28": Deployment to production with wrong NODE_ENV
    - "2024-12-08 15:30": First unauthorized access
    - "2024-12-08 15:37": Prometheus alert fires
    - "2024-12-08 15:40": Incident confirmed by SOC
    - "2024-12-08 15:45": Rollback initiated
    - "2024-12-08 15:50": Service restored with authentication enforced

  lessons_learned:
    - "Development bypass code must never reach production"
    - "Pre-deployment validation must include authentication tests"
    - "Environment variables must be validated at container startup"
    - "Code review checklist must include security bypass checks"
```

**5. Example Regulatory Authority Incident Report (CSA-02)**:
```yaml
nacsa_incident_report:
  report_id: "Example Regulatory Authority-2024-1208-001"
  operator: "Example MNO Sdn Bhd"
  operator_license: "MCMC-LIC-001234"

  incident_details:
    category: "Unauthorized Access"
    detection_time: "2024-12-08T15:30:00+08:00"
    report_time: "2024-12-08T18:30:00+08:00"  # Within 24 hours
    affected_system: "ODA Permissions Management System (TMFC035)"
    system_classification: "Critical National Information Infrastructure"

  incident_description: |
    A configuration error resulted in authentication bypass on the TMF672
    User Role Permission API. Unauthorized access was possible for 8 minutes
    before detection and remediation. 47 user records were accessed without
    proper authentication.

  root_cause: "Configuration management failure - NODE_ENV set to development"

  containment_actions:
    - action: "Rollback deployment"
      timestamp: "2024-12-08T15:45:00+08:00"
    - action: "Revoke active sessions"
      timestamp: "2024-12-08T15:46:00+08:00"
    - action: "Block suspicious IPs"
      timestamp: "2024-12-08T15:47:00+08:00"
    - action: "Verify authentication restored"
      timestamp: "2024-12-08T15:50:00+08:00"

  impact_assessment:
    customer_service_impact: "None"
    data_exposure: "47 user role records"
    financial_impact: "Negligible"
    duration_minutes: 8

  remediation_plan:
    immediate:
      - "Remove development bypass code from codebase"
      - "Implement environment variable validation"
      - "Add pre-deployment authentication smoke tests"
    short_term:
      - "Update code review checklist"
      - "Enhanced deployment pipeline validation"
      - "Security training for development team"
    long_term:
      - "Implement automated security testing in CI/CD"
      - "Deploy runtime application self-protection (RASP)"

  status: "CONTAINED"
  follow_up_required: false
```

**6. Example DPA Commissioner Notification (Example DPA-2024)**:
```yaml
pdpa_commissioner_notification:
  notification_id: "Example DPA-2024-1208-001"
  organization: "Example MNO Sdn Bhd"
  dpo_name: "Jane Tan"
  dpo_email: "dpo@example-mno.com.my"
  dpo_phone: "+60-3-8888-8888"

  breach_details:
    breach_type: "Unauthorized Access"
    detection_time: "2024-12-08T15:30:00+08:00"
    notification_time: "2024-12-09T10:00:00+08:00"  # Within 72 hours
    affected_individuals: 47

  personal_data_categories:
    - category: "Identity Data"
      fields: ["Full Name", "Email Address", "Phone Number"]
    - category: "Role Assignment Data"
      fields: ["User Role", "Permissions", "Access Level"]

  circumstances_of_breach: |
    A configuration error in the authentication middleware allowed
    unauthorized access to the User Role Permission API for 8 minutes.
    The breach was detected by automated monitoring and immediately
    contained through deployment rollback.

  likely_consequences:
    risk_level: "Medium"
    assessment: |
      Exposed data includes user identity information and role assignments.
      While no sensitive financial or health data was compromised, the
      information could be used for social engineering attacks. Risk is
      mitigated by short exposure window (8 minutes) and immediate
      containment.

  measures_taken:
    - "Immediate rollback of affected deployment"
    - "Revocation of all active API sessions"
    - "Blocking of suspicious source IPs"
    - "Root cause analysis completed"
    - "Remediation measures implemented"
    - "Notification to affected individuals (in progress)"

  measures_to_prevent_recurrence:
    - "Removal of development bypass code"
    - "Enhanced deployment validation"
    - "Automated security testing in CI/CD pipeline"
    - "Security training for development team"

  individual_notification:
    required: true
    method: "Email and SMS"
    timeline: "Within 7 days of breach"
    template: "Example DPA-BREACH-NOTIFICATION-001"
```

**7. Individual Notifications (Example DPA-P7)**:
```
Subject: Important Security Notice - Example MNO Account

Dear [Customer Name],

We are writing to inform you of a security incident that may have
affected your Example MNO account information.

INCIDENT DETAILS:
On December 8, 2024, between 15:32 and 15:40 (Example Country Time), a
configuration error in our systems allowed unauthorized access to
certain account information.

INFORMATION AFFECTED:
- Your name
- Email address
- Phone number
- Account access roles and permissions

WHAT WE'RE DOING:
- The issue was detected and resolved within 8 minutes
- All affected systems have been secured
- We have implemented additional security measures
- We have notified the Personal Data Protection Commissioner

WHAT YOU SHOULD DO:
- Be cautious of phishing attempts via email or SMS
- Do not share your account credentials
- Report any suspicious activity to security@example-mno.com.my

YOUR RIGHTS:
You have the right to:
- Request details of your personal data we hold
- File a complaint with the Personal Data Protection Commissioner
- Request compensation if you suffer damage

CONTACT US:
Security Team: security@example-mno.com.my | 1800-88-1234
Data Protection Officer: dpo@example-mno.com.my

We sincerely apologize for this incident and any concern it may cause.

Example MNO Security Team
```

---

### Compliance Evidence Generated

**Incident Register**:
```json
{
  "incident_id": "INC-2024-1208-001",
  "type": "Authentication Bypass",
  "detection_time": "2024-12-08T15:30:00+08:00",
  "containment_time": "2024-12-08T15:50:00+08:00",
  "resolution_time": "2024-12-08T16:30:00+08:00",

  "nist_csf_violations": [
    {
      "category": "PR.AA-03",
      "description": "Users are authenticated",
      "status": "VIOLATED"
    },
    {
      "category": "PR.AA-04",
      "description": "Identity assertions are protected",
      "status": "VIOLATED"
    }
  ],

  "nist_csf_detections": [
    {
      "category": "DE.CM-09",
      "description": "Continuous monitoring detected anomaly",
      "status": "WORKING"
    },
    {
      "category": "DE.AE-02",
      "description": "Adverse event analysis identified breach",
      "status": "WORKING"
    }
  ],

  "regulatory_notifications": [
    {
      "regulation": "Cyber Security Act 2024",
      "requirement": "CSA-02: 24-hour incident reporting",
      "deadline": "2024-12-09T15:30:00+08:00",
      "submitted": "2024-12-08T18:30:00+08:00",
      "status": "COMPLIANT"
    },
    {
      "regulation": "Example DPA 2024",
      "requirement": "72-hour Commissioner notification",
      "deadline": "2024-12-11T15:30:00+08:00",
      "submitted": "2024-12-09T10:00:00+08:00",
      "status": "COMPLIANT"
    }
  ],

  "evidence_location": "s3://example-mno-compliance/incidents/INC-2024-1208-001/",
  "root_cause_analysis": "s3://example-mno-compliance/incidents/INC-2024-1208-001/rca.pdf",
  "lessons_learned": "https://wiki.example-mno.internal/incidents/INC-2024-1208-001"
}
```

---

## Simulation 2: Data Exposure via Misconfigured Logging

### Scenario Description

A developer enables debug logging to troubleshoot an issue, inadvertently logging sensitive customer data (PII) to Elasticsearch, which is accessible to the entire engineering team.

---

### Layer 3: Code Change (Implementation)

**File**: `components/example-mno/tmfc035-deployment/config/logger.js`

**Before (Secure)**:
```javascript
const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  defaultMeta: { service: 'tmfc035-permissions' },
  transports: [
    new winston.transports.Console(),
    new ElasticsearchTransport({
      level: 'info',
      clientOpts: { node: process.env.ELASTICSEARCH_URL },
      index: 'example-mno-oda-permissions'
    })
  ]
});

// Sanitize sensitive data before logging
logger.sanitize = (obj) => {
  const sanitized = { ...obj };
  const sensitiveFields = ['password', 'token', 'ssn', 'nric', 'creditCard'];

  for (const field of sensitiveFields) {
    if (sanitized[field]) {
      sanitized[field] = '[REDACTED]';
    }
  }

  return sanitized;
};
```

**After (Vulnerable)**:
```javascript
const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',  // Changed to use env variable
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  defaultMeta: { service: 'tmfc035-permissions' },
  transports: [
    new winston.transports.Console(),
    new ElasticsearchTransport({
      level: process.env.LOG_LEVEL || 'info',  // Changed
      clientOpts: { node: process.env.ELASTICSEARCH_URL },
      index: 'example-mno-oda-permissions'
    })
  ]
});

// Sanitize sensitive data before logging
logger.sanitize = (obj) => {
  // DEBUG: Temporarily disabled for troubleshooting
  if (process.env.LOG_LEVEL === 'debug') {
    return obj;  // ❌ NO SANITIZATION in debug mode!
  }

  const sanitized = { ...obj };
  const sensitiveFields = ['password', 'token', 'ssn', 'nric', 'creditCard'];

  for (const field of sensitiveFields) {
    if (sanitized[field]) {
      sanitized[field] = '[REDACTED]';
    }
  }

  return sanitized;
};
```

**Deployment with Debug Logging**:
```yaml
# kubectl patch deployment
kubectl set env deployment/example-mno-permissions-management \
  LOG_LEVEL=debug \
  -n example-mno-oda-party
```

**Timeline**:
- **T+0**: LOG_LEVEL=debug enabled
- **T+5 minutes**: 1,200 log entries with PII written to Elasticsearch
- **T+30 minutes**: Data Loss Prevention (DLP) scanner detects PII in logs
- **T+35 minutes**: Security team alerted

---

### Layer 2: NIST CSF 2.0 Detection

#### Detection: Data Security Violation (PR.DS-05)

**DLP Scanner Alert**:
```yaml
dlp_alert:
  alert_id: "DLP-2024-1208-002"
  scanner: "Elasticsearch DLP Scanner"
  rule: "PII in Logs Detection"

  findings:
    - type: "NRIC (Example Countryn ID)"
      count: 47
      pattern: "\\d{6}-\\d{2}-\\d{4}"
      sample: "850123-14-5678"
    - type: "Email Address"
      count: 47
      pattern: "[a-z0-9._%+-]+@[a-z0-9.-]+\\.[a-z]{2,}"
      sample: "customer@example.com"
    - type: "Phone Number"
      count: 47
      pattern: "\\+60[0-9]{9,10}"
      sample: "+60123456789"

  index: "example-mno-oda-permissions-2024.12.08"
  time_range:
    start: "2024-12-08T16:00:00+08:00"
    end: "2024-12-08T16:30:00+08:00"

  nist_control_violated: "PR.DS-05"
  pdpa_principle_violated: "Security Principle (P4)"
  severity: "HIGH"
```

---

### Layer 1: Example Countryn Regulatory Obligations

**Example DPA Violation**: Security Principle (P4) - Failure to protect personal data
**Example DPA Breach**: Yes - Personal data exposed to unauthorized internal users

**Commissioner Notification Required**: Yes (72 hours)
**Individual Notification Required**: Yes (47 affected customers)

This simulation demonstrates a common developer mistake that triggers Example DPA obligations even though it's an "internal" exposure.

---

## Summary: Bottom-Up Flow

Each breach scenario demonstrates:

1. **Layer 3 (Implementation)**: Specific code change or configuration error
2. **Layer 2 (NIST CSF)**: Which controls detect and respond
3. **Layer 1 (Example Countryn Law)**: Which legal obligations are triggered

This layered approach ensures:
- ✅ Automated detection at implementation layer
- ✅ Consistent mapping to framework controls
- ✅ Automatic triggering of regulatory workflows
- ✅ Complete audit trail for compliance
- ✅ Reusable across all jurisdictions

---

**Document Version**: 1.0
**Last Updated**: December 2024
**Next Review**: June 2025
**Owner**: Example MNO CISO Office
