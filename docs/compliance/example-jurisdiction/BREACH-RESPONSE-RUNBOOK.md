# Breach Response Runbook - Example Countryn Regulatory Compliance

## Quick Reference

**Example Cybersecurity Act Deadlines**:
- Example Regulatory Authority Notification: **24 hours** from detection
- Incident Report: **Within reporting window**

**Example DPA 2024 Deadlines**:
- Commissioner Notification: **72 hours** from awareness of breach
- Individual Notification: **Without undue delay** (typically within 7 days)
- DSAR Response: **21 days** from request

**Emergency Contacts**:
- CISO: +60-12-XXX-XXXX
- DPO: +60-12-XXX-XXXX
- SOC Hotline: 1800-88-XXXX
- Example Regulatory Authority Incident Hotline: 1-300-88-2999

---

## Runbook: Authentication Bypass Incident

**Incident Type**: Unauthorized Access (CSA Category 1)
**Trigger**: `AuthenticationBypassDetected` Prometheus alert
**NIST CSF**: PR.AA-03, PR.AA-04 violations; DE.CM-09 detection

### Phase 1: Detection and Initial Response (T+0 to T+10 minutes)

#### Step 1: Alert Verification (2 minutes)

**Action**: Confirm the alert is not a false positive

```bash
# Check Prometheus metrics
curl -s "http://prometheus.example-mno.internal/api/v1/query" \
  --data-urlencode 'query=http_requests_total{auth_header_present="false",status_code=~"2.."}' \
  | jq '.data.result[]'

# Expected: Positive values confirm unauthenticated access
```

**Decision Point**:
- ✅ **If confirmed**: Proceed to Step 2
- ❌ **If false positive**: Close alert, document reason

#### Step 2: Incident Declaration (2 minutes)

**Action**: Create incident record

```bash
# Create incident via API
curl -X POST https://api.example-mno.internal/security/incidents \
  -H "Content-Type: application/json" \
  -d '{
    "type": "Authentication Bypass",
    "severity": "HIGH",
    "detection_time": "'$(date -Iseconds)'",
    "affected_component": "TMFC035",
    "csa_category": "Unauthorized Access",
    "csa_reportable": true,
    "pdpa_assessment_required": true
  }'

# Response includes incident_id: INC-2024-XXXX-XXX
```

**Documentation**:
- Incident ID: _________________
- Detection Time: _________________
- Affected Component: _________________

#### Step 3: Immediate Containment (5 minutes)

**Action**: Stop the bleeding

```bash
# Option A: Rollback deployment (fastest)
kubectl rollout undo deployment/example-mno-permissions-management \
  -n example-mno-oda-party

# Verify rollback
kubectl rollout status deployment/example-mno-permissions-management \
  -n example-mno-oda-party

# Option B: Kill vulnerable pods (if rollback unavailable)
kubectl delete pods -l app=example-mno-permissions-management \
  -n example-mno-oda-party

# Option C: Scale to zero (nuclear option)
kubectl scale deployment/example-mno-permissions-management \
  --replicas=0 \
  -n example-mno-oda-party
```

**Verification**:
```bash
# Confirm no more unauthenticated requests
watch -n 5 'curl -s http://prometheus.example-mno.internal/api/v1/query \
  --data-urlencode "query=rate(http_requests_total{auth_header_present=\"false\"}[1m])"'

# Should show: 0
```

#### Step 4: Escalation (1 minute)

**Action**: Notify key stakeholders

```bash
# Trigger CISO pagerduty
curl -X POST https://events.pagerduty.com/v2/enqueue \
  -H "Content-Type: application/json" \
  -d '{
    "routing_key": "'$PAGERDUTY_CISO_KEY'",
    "event_action": "trigger",
    "payload": {
      "summary": "AUTH BYPASS: Incident '$INCIDENT_ID'",
      "severity": "critical",
      "source": "ODA Security",
      "custom_details": {
        "incident_id": "'$INCIDENT_ID'",
        "csa_deadline": "'$(date -d '+24 hours' -Iseconds)'",
        "pdpa_deadline": "'$(date -d '+72 hours' -Iseconds)'"
      }
    }
  }'

# Notify SOC team
slack_notify "#security-crisis" "🚨 CRITICAL: Authentication bypass incident $INCIDENT_ID"
```

---

### Phase 2: Evidence Collection (T+10 to T+30 minutes)

#### Step 5: Preserve Logs (10 minutes)

**Action**: Collect all relevant logs before they rotate

```bash
# Create evidence directory
EVIDENCE_DIR="/evidence/$INCIDENT_ID"
mkdir -p $EVIDENCE_DIR/{logs,metrics,database,k8s}

# Export pod logs
kubectl logs deployment/example-mno-permissions-management \
  --since-time="$DETECTION_TIME_MINUS_30M" \
  --namespace=example-mno-oda-party \
  --all-containers=true \
  > $EVIDENCE_DIR/logs/pod-logs.txt

# Export Elasticsearch logs
curl -X POST "https://elasticsearch.example-mno.internal/example-mno-oda-permissions-*/_search" \
  -H 'Content-Type: application/json' \
  -d'{
    "query": {
      "range": {
        "@timestamp": {
          "gte": "'$DETECTION_TIME_MINUS_30M'",
          "lte": "'$DETECTION_TIME_PLUS_30M'"
        }
      }
    },
    "size": 10000
  }' | jq '.' > $EVIDENCE_DIR/logs/elasticsearch.json

# Calculate checksums
find $EVIDENCE_DIR -type f -exec sha256sum {} \; > $EVIDENCE_DIR/checksums.txt
```

#### Step 6: Capture Metrics (5 minutes)

**Action**: Export Prometheus metrics

```bash
# Export relevant metrics
METRICS=(
  "http_requests_total"
  "authentication_failures_total"
  "jwt_validation_duration_seconds"
  "api_latency_seconds"
)

for metric in "${METRICS[@]}"; do
  curl -G "http://prometheus.example-mno.internal/api/v1/query_range" \
    --data-urlencode "query=$metric" \
    --data-urlencode "start=$DETECTION_TIME_MINUS_30M" \
    --data-urlencode "end=$DETECTION_TIME_PLUS_30M" \
    --data-urlencode "step=15s" \
    | jq '.' > $EVIDENCE_DIR/metrics/$metric.json
done
```

#### Step 7: Database Snapshot (5 minutes)

**Action**: Capture affected data state

```bash
# Export affected user_roles table
psql -h postgres.example-mno.internal -U readonly -d oda_party <<EOF > $EVIDENCE_DIR/database/user_roles.sql
COPY (
  SELECT *
  FROM user_roles
  WHERE updated_at BETWEEN '$DETECTION_TIME_MINUS_30M' AND '$DETECTION_TIME_PLUS_30M'
) TO STDOUT WITH CSV HEADER;
EOF

# Export audit trail
psql -h postgres.example-mno.internal -U readonly -d oda_party <<EOF > $EVIDENCE_DIR/database/audit_trail.sql
COPY (
  SELECT *
  FROM audit_trail
  WHERE timestamp BETWEEN '$DETECTION_TIME_MINUS_30M' AND '$DETECTION_TIME_PLUS_30M'
  ORDER BY timestamp
) TO STDOUT WITH CSV HEADER;
EOF
```

#### Step 8: Archive Evidence (10 minutes)

**Action**: Store evidence immutably in S3

```bash
# Upload to compliance S3 bucket
aws s3 sync $EVIDENCE_DIR/ \
  s3://example-mno-compliance/incidents/$INCIDENT_ID/ \
  --storage-class GLACIER_IR \
  --sse AES256 \
  --metadata "incident_id=$INCIDENT_ID,detection_time=$DETECTION_TIME"

# Enable object lock (prevent deletion for 7 years)
aws s3api put-object-legal-hold \
  --bucket example-mno-compliance \
  --key incidents/$INCIDENT_ID/ \
  --legal-hold Status=ON

# Verify upload
aws s3 ls s3://example-mno-compliance/incidents/$INCIDENT_ID/ --recursive
```

---

### Phase 3: Impact Assessment (T+30 to T+60 minutes)

#### Step 9: Determine Affected Data (15 minutes)

**Action**: Identify what data was accessed

```sql
-- Query: Find all API requests without authentication
SELECT
  al.timestamp,
  al.source_ip,
  al.endpoint,
  al.http_method,
  al.response_code,
  al.response_size_bytes,
  COUNT(*) OVER (PARTITION BY al.source_ip) as requests_from_ip
FROM api_logs al
WHERE al.timestamp BETWEEN :detection_time_minus_30m AND :detection_time_plus_30m
  AND al.jwt_token_id IS NULL
  AND al.endpoint LIKE '/tmf-api/userRolePermission%'
  AND al.response_code BETWEEN 200 AND 299
ORDER BY al.timestamp;

-- Query: Identify affected user records
SELECT DISTINCT
  ur.user_id,
  u.email,
  u.phone_number,
  u.nric,
  ur.role_name,
  al.timestamp as accessed_at,
  al.source_ip
FROM user_roles ur
JOIN users u ON u.id = ur.user_id
JOIN api_logs al ON al.response_data::jsonb @> jsonb_build_object('id', ur.id)
WHERE al.timestamp BETWEEN :detection_time_minus_30m AND :detection_time_plus_30m
  AND al.jwt_token_id IS NULL
  AND al.response_code BETWEEN 200 AND 299;
```

**Documentation Template**:
```yaml
impact_assessment:
  affected_individuals: _____ (count)
  data_categories_exposed:
    - User identities (name, email, phone)
    - Role assignments
    - [ ] Passwords (if yes, immediate password reset required)
    - [ ] Financial data
    - [ ] Health data
  unauthorized_requests: _____ (count)
  time_window: _____ minutes
  source_ips: [______, ______, ______]
  endpoints_accessed:
    - GET /userRole: _____ requests
    - POST /userRole: _____ requests
    - PATCH /userRole/{id}: _____ requests
    - DELETE /userRole/{id}: _____ requests
```

#### Step 10: Example Cybersecurity Act Assessment (15 minutes)

**Action**: Determine Example Regulatory Authority reporting requirements

**Checklist**:
- [ ] Is this a "Critical National Information Infrastructure" (CNII) system?
      ✅ Yes (telecommunications operator)
- [ ] Does this fall into CSA reportable categories?
      - [ ] Unauthorized access ✅
      - [ ] Data breach ✅
      - [ ] Service disruption ❌
      - [ ] Malware ❌
      - [ ] DDoS ❌
- [ ] Severity assessment:
      - [ ] HIGH: Authentication controls bypassed ✅
      - [ ] MEDIUM: Limited scope/impact ❌
      - [ ] LOW: Minimal impact ❌

**Result**:
- ✅ **Example Regulatory Authority Reporting Required**
- ⏰ **Deadline**: {{ detection_time + 24 hours }}

#### Step 11: Example DPA 2024 Assessment (15 minutes)

**Action**: Determine if personal data breach occurred

**Checklist**:
- [ ] Was personal data accessed?
      ✅ Yes (names, emails, phone numbers, role data)
- [ ] Is the organization a "data user" under Example DPA?
      ✅ Yes
- [ ] Number of affected individuals: _____
      (If > 0, breach confirmed)
- [ ] Likelihood of harm to individuals:
      - [ ] HIGH: Sensitive data (NRIC, financial, health) ❌
      - [ ] MEDIUM: Identity data + role information ✅
      - [ ] LOW: Non-sensitive data only ❌

**Risk Assessment**:
```yaml
pdpa_breach:
  status: CONFIRMED / NOT CONFIRMED
  affected_count: _____
  harm_likelihood: HIGH / MEDIUM / LOW
  harm_description: |
    Exposed data could be used for:
    - Social engineering attacks
    - Phishing campaigns targeting specific roles
    - Unauthorized privilege escalation attempts

  commissioner_notification: YES / NO
  individual_notification: YES / NO
  media_notification: YES / NO (if > 1000 affected)
```

#### Step 12: Root Cause Determination (15 minutes)

**Action**: Identify how the breach occurred

**Investigation**:
```bash
# Check deployment history
kubectl rollout history deployment/example-mno-permissions-management \
  -n example-mno-oda-party

# Get current deployment config
kubectl get deployment example-mno-permissions-management \
  -n example-mno-oda-party \
  -o yaml > $EVIDENCE_DIR/k8s/deployment-current.yaml

# Check environment variables
kubectl get deployment example-mno-permissions-management \
  -n example-mno-oda-party \
  -o jsonpath='{.spec.template.spec.containers[0].env[*]}' | jq '.'

# Review recent commits
git log --since="7 days ago" --oneline --all -- components/example-mno/tmfc035-deployment/
```

**Root Cause Template**:
```yaml
root_cause:
  category:
    - Configuration Error
    - Code Vulnerability
    - Process Failure
    - Human Error
    - External Attack

  description: |
    [Describe exactly what caused the breach]

  contributing_factors:
    - [Factor 1]
    - [Factor 2]

  timeline:
    - time: [timestamp]
      event: [what happened]
```

---

### Phase 4: Regulatory Reporting (T+1 hour to T+24 hours)

#### Step 13: Example Regulatory Authority Incident Report (within 24 hours)

**Action**: Submit report to Example Regulatory Authority portal

**Report Sections**:

1. **Operator Information**:
```yaml
operator:
  name: "Example MNO Sdn Bhd"
  license_number: "MCMC-LIC-001234"
  contact_person: "[CISO Name]"
  contact_email: "ciso@example-mno.com.my"
  contact_phone: "+60-3-XXXX-XXXX"
```

2. **Incident Details**:
```yaml
incident:
  detection_time: "[ISO 8601 timestamp]"
  report_time: "[ISO 8601 timestamp]"
  category: "Unauthorized Access"
  severity: "HIGH"
  affected_system: "ODA Permissions Management (TMFC035)"
  system_classification: "CNII"
```

3. **Impact Assessment**:
```yaml
impact:
  customer_services_affected: "No"
  number_of_customers_affected: 0
  service_downtime: "0 minutes"
  data_compromised: "Yes"
  data_categories:
    - "User identities (47 records)"
    - "Role assignments"
```

4. **Containment Actions**:
```yaml
containment:
  - timestamp: "[ISO 8601]"
    action: "Rolled back deployment to secure version"
  - timestamp: "[ISO 8601]"
    action: "Revoked all active API sessions"
  - timestamp: "[ISO 8601]"
    action: "Blocked suspicious source IPs"
  - timestamp: "[ISO 8601]"
    action: "Verified authentication restored"
```

5. **Root Cause**:
```yaml
root_cause:
  category: "Configuration Error"
  description: |
    Environment variable NODE_ENV incorrectly set to "development"
    in production deployment, activating authentication bypass code.
```

6. **Remediation**:
```yaml
remediation:
  immediate:
    - "Removed authentication bypass code"
    - "Corrected NODE_ENV to production"
    - "Implemented deployment validation checks"
  short_term:
    - "Enhanced code review process"
    - "Automated environment variable validation"
  long_term:
    - "Security testing in CI/CD pipeline"
    - "Runtime application self-protection (RASP)"
```

**Submission**:
```bash
# Example Regulatory Authority portal submission (manual via web portal)
# URL: https://incidents.nacsa.gov.my

# Alternative: Email submission (if portal unavailable)
# To: incidents@nacsa.gov.my
# Subject: Incident Report - [Operator Name] - [Incident Date]
# Attach: nacsa-report-$INCIDENT_ID.pdf
```

#### Step 14: Example DPA Commissioner Notification (within 72 hours)

**Action**: Submit breach notification if required

**Notification Form**:

1. **Data User Information**:
```yaml
data_user:
  name: "Example MNO Sdn Bhd"
  registration_number: "199101009694 (209478-U)"
  dpo_name: "[DPO Name]"
  dpo_email: "dpo@example-mno.com.my"
  dpo_phone: "+60-3-XXXX-XXXX"
```

2. **Breach Details**:
```yaml
breach:
  nature: "Unauthorized Access"
  discovery_date: "[Date]"
  notification_date: "[Date]"
  affected_individuals: [Count]
  personal_data_categories:
    - "Full Name"
    - "Email Address"
    - "Phone Number"
    - "Role/Permission Data"
```

3. **Circumstances**:
```yaml
circumstances: |
  A configuration error resulted in authentication bypass on the
  User Role Permission API for 8 minutes. Automated monitoring
  detected the issue and triggered immediate containment through
  deployment rollback. Forensic analysis identified 47 affected
  user records.
```

4. **Likely Consequences**:
```yaml
consequences:
  severity: "MEDIUM"
  assessment: |
    Risk Level: Medium

    The exposed information includes user identity data and role
    assignments. While no financial or health information was
    compromised, the data could potentially be used for:
    - Targeted phishing campaigns
    - Social engineering attacks
    - Unauthorized access attempts

    Mitigating Factors:
    - Short exposure window (8 minutes)
    - Immediate containment and remediation
    - No evidence of data exfiltration
    - Affected individuals notified
```

5. **Measures Taken**:
```yaml
measures:
  immediate:
    - "System secured within 10 minutes of detection"
    - "All potentially compromised sessions revoked"
    - "Suspicious IPs blocked"
    - "Forensic evidence preserved"

  notification:
    - "Commissioner notification: Within 72 hours"
    - "Individual notification: Within 7 days"

  remediation:
    - "Root cause identified and corrected"
    - "Enhanced deployment controls implemented"
    - "Security training for development team"
    - "Continuous monitoring enhanced"
```

**Submission**:
```bash
# Example DPA Commissioner portal
# URL: https://eaduan.pdp.gov.my

# Email: pdp@kpdnhep.gov.my
# Subject: Data Breach Notification - [Organization] - [Date]
```

#### Step 15: Individual Notifications (within 7 days)

**Action**: Notify affected individuals

**Notification Email Template**:
```
Subject: Important Security Notice - Your Example MNO Account

Dear [Customer Name],

We are writing to inform you of a security incident that affected
your Example MNO account information.

═══════════════════════════════════════════════════════════

WHAT HAPPENED:
On [Date] between [Start Time] and [End Time], a configuration
error in our systems allowed brief unauthorized access to certain
account information.

INFORMATION AFFECTED:
□ Your name
□ Your email address
□ Your phone number
□ Your account access roles

WHAT WE DID:
✓ Detected and resolved the issue within 8 minutes
✓ Secured all affected systems
✓ Implemented additional security measures
✓ Notified the Personal Data Protection Commissioner

WHAT YOU SHOULD DO:
• Be cautious of unexpected emails or calls asking for information
• Do not click suspicious links or share account credentials
• Report any unusual activity: security@example-mno.com.my

YOUR RIGHTS UNDER Example DPA:
• Request details of your personal data we hold
• Request correction of inaccurate data
• Withdraw consent for data processing
• File complaint with Example DPA Commissioner
• Seek compensation if you suffer damage

CONTACT INFORMATION:
Security Team: security@example-mno.com.my | 1800-88-1234
Data Protection Officer: dpo@example-mno.com.my
Example DPA Commissioner: pdp@kpdnhep.gov.my | +603-8882-5555

We sincerely apologize for this incident and any concern it causes.

Regards,
Example MNO Security Team

═══════════════════════════════════════════════════════════
This is an automated security notification. Please do not reply
to this email. For assistance, contact the numbers above.
```

**Bulk Notification Script**:
```python
#!/usr/bin/env python3
import psycopg2
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

# Get affected users
conn = psycopg2.connect("host=postgres.example-mno.internal dbname=oda_party")
cur = conn.execute("""
    SELECT DISTINCT user_id, email, full_name, phone_number
    FROM breach_affected_users
    WHERE incident_id = %s
""", (INCIDENT_ID,))

affected_users = cur.fetchall()

# Send notifications
for user_id, email, name, phone in affected_users:
    # Email notification
    send_breach_notification_email(email, name, INCIDENT_ID)

    # SMS notification (via Twilio/AWS SNS)
    send_breach_notification_sms(phone, INCIDENT_ID)

    # Log notification
    log_breach_notification(user_id, INCIDENT_ID, 'sent')

print(f"Notified {len(affected_users)} individuals")
```

---

### Phase 5: Post-Incident Activities (T+24 hours to T+30 days)

#### Step 16: Root Cause Analysis Report (within 7 days)

**Action**: Document comprehensive RCA

**Report Template**: See [RCA Template](./templates/root-cause-analysis.md)

**Key Sections**:
1. Executive Summary
2. Timeline of Events
3. Root Cause Identification
4. Contributing Factors
5. Lessons Learned
6. Remediation Actions
7. Recommendations

#### Step 17: Corrective Actions (within 30 days)

**Action**: Implement fixes to prevent recurrence

**Immediate Actions** (within 7 days):
- [ ] Remove all development bypass code from production codebase
- [ ] Implement environment variable validation at startup
- [ ] Add pre-deployment smoke tests for authentication
- [ ] Update deployment runbooks

**Short-term Actions** (within 30 days):
- [ ] Enhance code review checklist with security items
- [ ] Implement automated security scanning in CI/CD
- [ ] Deploy static analysis tools (SonarQube, Semgrep)
- [ ] Conduct security training for development team

**Long-term Actions** (within 90 days):
- [ ] Implement runtime application self-protection (RASP)
- [ ] Deploy Web Application Firewall (WAF) rules
- [ ] Enhance monitoring with behavioral analytics
- [ ] Conduct penetration testing

#### Step 18: Lessons Learned Session (within 14 days)

**Action**: Conduct blameless postmortem

**Participants**:
- CISO
- Security team
- Development team
- Operations team
- Compliance/DPO

**Discussion Points**:
1. What went well?
2. What didn't go well?
3. What did we learn?
4. What will we do differently?

**Output**: Action items with owners and deadlines

---

## Checklist Summary

### Immediate (T+0 to T+30 minutes)
- [ ] Verify alert (2 min)
- [ ] Create incident record (2 min)
- [ ] Contain the breach (5 min)
- [ ] Escalate to CISO/DPO (1 min)
- [ ] Preserve logs (10 min)
- [ ] Capture metrics (5 min)
- [ ] Database snapshot (5 min)

### Short-term (T+30 min to T+24 hours)
- [ ] Impact assessment (30 min)
- [ ] CSA assessment (15 min)
- [ ] Example DPA assessment (15 min)
- [ ] Root cause determination (15 min)
- [ ] Example Regulatory Authority report (within 24 hours)

### Medium-term (T+24 hours to T+7 days)
- [ ] Example DPA Commissioner notification (within 72 hours)
- [ ] Individual notifications (within 7 days)
- [ ] Root cause analysis report (within 7 days)

### Long-term (T+7 days to T+30 days)
- [ ] Corrective actions implementation
- [ ] Lessons learned session (within 14 days)
- [ ] Follow-up audit (within 30 days)

---

**Document Version**: 1.0
**Last Updated**: December 2024
**Next Review**: June 2025
**Owner**: Example MNO CISO Office
