# ODA Architecture Documentation

## Architecture Overview

This document provides detailed technical documentation for the ODA (Open Digital Architecture) reference architecture with integrated identity and access management.

## Design Principles

### 1. Component-Based Architecture
- **Modularity**: Each component is independently deployable and scalable
- **Loose Coupling**: Components interact via standardized TMF Open APIs
- **High Cohesion**: Related functionality grouped within domain boundaries
- **API-First**: All interactions through well-defined REST APIs

### 2. Zero Trust Security
- **Never Trust, Always Verify**: All requests authenticated and authorized
- **Least Privilege**: Components granted minimum necessary permissions
- **Micro-Segmentation**: Network and application-level isolation
- **Continuous Verification**: Token validation on every request

### 3. Domain-Driven Design
- **Bounded Contexts**: Clear domain boundaries (Party, Commerce, Production, Engagement, Intelligence)
- **Ubiquitous Language**: TMF standards provide common vocabulary
- **Domain Events**: Event-driven communication for loose coupling
- **Aggregate Roots**: Domain entities with clear ownership

## Architecture Layers

### Layer 1: External Identity Provider

**Purpose**: Centralized authentication and authorization service

**Components**:
- **Identity Management System (IDM)**: OAuth2/OIDC provider (Keycloak, Azure AD, Okta)
- **User Directory**: Stores user accounts and profiles
- **Role Repository**: Manages roles and permissions
- **Token Service**: Issues and validates JWT tokens

**Standards**:
- OAuth 2.0 (RFC 6749)
- OpenID Connect (OIDC)
- JWT (RFC 7519)
- SCIM 2.0 for user provisioning

### Layer 2: ODA Canvas Platform

**Purpose**: Orchestration and lifecycle management for ODA Components

**Components**:

#### Component Operators
- Kubernetes operators managing component lifecycle
- Handle installation, upgrade, scaling, and removal
- Implement operator pattern for declarative management

#### Identity Operator
- Bridges ODA Components with external IDM
- Automates OAuth2 client provisioning
- Synchronizes roles between components and IDM
- Implements TMF672 event listeners for dynamic roles

#### Secret Store
- Secure credential storage (Kubernetes Secrets, HashiCorp Vault)
- Rotation and lifecycle management of secrets
- Encryption at rest and in transit
- Access auditing

#### API Gateway
- Single entry point for all external API traffic
- JWT token validation
- Rate limiting and throttling
- Request routing and load balancing
- API versioning support

### Layer 3: System of Engagement (SoE)

**Purpose**: Customer-facing interaction layer

#### Engagement Domain

**Capabilities**:
- Omnichannel experience management
- Customer journey orchestration
- Self-service portals
- Digital touchpoints (Web, Mobile, API)
- Campaign and offer management
- Notification services

**Integration Points**:
- Calls Party Domain for customer profile data
- Calls Commerce Domain for product information
- Sends events to Intelligence Domain for analytics

**Security**:
- All requests require valid JWT tokens
- User session management
- CORS configuration for web clients
- Mobile app certificate pinning

### Layer 4: System of Records (SoR)

**Purpose**: Core business data and operations

#### Party Domain

**TMF Capabilities**:
- **TMFC028 Party Management**: Customer, partner, supplier lifecycle
- **TMFC035 Party Roles & Permission Management**: RBAC implementation
- **TMFC020 Digital Identity Management**: Identity lifecycle
- **TMFC039 Agreement Management**: Contracts and SLAs

**Key Features**:
- Central identity authority for all domains
- Implements TMF672 API for dynamic role management
- Customer 360° view
- Consent and privacy management (GDPR, CCPA)

**APIs Exposed**:
- TMF632 Party Management API
- TMF669 Party Role API
- TMF672 User Roles and Permissions API
- TMF651 Agreement Management API

##### TMFC035 Component Implementation

The TMFC035 Permissions Management component is a reference implementation available in this repository:

**Component Specification**: `components/oda-canvas/TMFC035-PermissionsManagement.yaml`

**Core Function**:
- **Exposed APIs**:
  - TMF672 User Role Permission Management API (v4.0.0) - Port 8080
    - Full CRUD on permissions (GET, GET/id, POST, PATCH/id, DELETE/id)
    - Full CRUD on userRole (GET, GET/id, POST, PATCH/id, DELETE/id)
  - TMF669 Party Role Management API (v4.0.0) - Port 8080
    - Full CRUD on partyRole (GET, GET/id, POST, PATCH/id, DELETE/id)

- **Dependent APIs**:
  - TMF632 Party Management API (required) - Individual and organization data retrieval
  - TMF701 Process Flow Management API (optional) - Process orchestration support

**Security Function**:
- Sidecar pattern for secrets management on port 8080
- Canvas system role-based access control
- OAuth2 client credentials for service-to-service authentication
- Pod selector-based security isolation

**Event Notification**:
- **Published Events**:
  - User Role Events: `userRoleCreateEvent`, `userRoleAttributeValueChangeEvent`, `userRoleStateChangeEvent`, `userRoleDeleteEvent`
  - Permission Events: `permissionCreateEvent`, `permissionAttributeValueChangeEvent`, `permissionStateChangeEvent`, `permissionDeleteEvent`
  - Party Role Events: `PartyRoleCreateEvent`, `PartyRoleAttributeValueChangeEvent`, `PartyRoleStateChangeEvent`, `PartyRoleDeleteEvent`

- **Subscribed Events**:
  - TMF632 Party Management events (individual and organization lifecycle)
  - TMF701 Process Flow events (process and task flow state changes)

**Management Function**:
- Prometheus metrics exposed on port 4000
- Standard Kubernetes health checks
- Helm-based deployment with configurable values

**Client Implementations**:

See `components/example-mno/` directory for Example MNO-specific production configurations including:
- Keycloak IDM integration with OAuth2/OIDC
- Apache Kafka event bus for asynchronous event processing
- Istio service mesh with mTLS enforcement
- Kong API Gateway integration
- Elasticsearch logging and Jaeger distributed tracing
- High availability configuration with 3 replicas
- Example Countryn data residency compliance

#### Commerce Domain

**TMF Capabilities**:
- **TMFC001 Product Catalog Management**: Product offerings and pricing
- **TMFC002 Product Order Capture & Management**: Order lifecycle
- **TMFC005 Product Inventory Management**: Subscription management

**Key Features**:
- Product catalog with dynamic pricing
- Order orchestration and fulfillment
- Revenue management and billing
- Inventory and subscription tracking

**APIs Exposed**:
- TMF620 Product Catalog API
- TMF622 Product Ordering API
- TMF637 Product Inventory API
- TMF678 Customer Bill Management API

**Role Examples**:
- Product Manager: Create/update products
- Pricing Analyst: Manage pricing rules
- Order Admin: Process orders
- Sales Agent: Create customer orders

#### Production Domain

**TMF Capabilities**:
- **TMFC008 Service Inventory Management**: Active services
- **TMFC012 Resource Inventory Management**: Network resources

**Key Features**:
- Service activation and configuration
- Resource topology and relationships
- Network element management
- Capacity planning and optimization
- Performance monitoring

**APIs Exposed**:
- TMF638 Service Inventory API
- TMF639 Resource Inventory API
- TMF641 Service Ordering API
- TMF640 Service Activation API

**Role Examples**:
- Service Provisioner: Activate services
- NOC Engineer: Monitor service health
- Network Engineer: Manage resources
- Capacity Planner: Forecast capacity

**Integration Patterns**:
- Receives service orders from Commerce Domain
- Updates service status to Engagement Domain
- Sends performance metrics to Intelligence Domain

### Layer 5: System of Innovation (SoI)

**Purpose**: Analytics, insights, and intelligent automation

#### Intelligence Domain

**Capabilities**:
- Business intelligence and analytics
- AI/ML model training and inference
- Predictive analytics
- Real-time insights and monitoring
- Intelligent automation and orchestration
- Data lake and data warehouse

**Key Features**:
- Read-only access to SoR domains
- Real-time and batch analytics
- Machine learning pipelines
- Anomaly detection
- Recommendation engines
- Automated decision-making

**Data Sources**:
- Party Domain: Customer behavior and preferences
- Commerce Domain: Order patterns and revenue
- Production Domain: Service quality and performance
- Engagement Domain: Interaction patterns

**Security**:
- Service accounts with read-only permissions
- Data anonymization and masking
- Access logging and auditing
- Data retention policies

## Identity Management Lifecycle

### Bootstrap Process

```
1. Component Installation
   ├─> Component Operator creates Identity Resource (K8s CR)
   └─> Identity Operator detects new component

2. OAuth2 Client Provisioning
   ├─> Identity Operator calls IDM API
   ├─> IDM creates new OAuth2 client
   ├─> Client ID and secret generated
   └─> Credentials stored in Secret Store

3. Bootstrap Admin Role
   ├─> Identity Operator reads SecurityFunction segment
   ├─> Canvas system user assigned to Admin role
   └─> Canvas can now manage component via APIs

4. System User Creation
   ├─> IDM creates service account for component
   ├─> System user assigned necessary permissions
   └─> Used for component-to-component API calls

5. Role Management Setup
   ├─> Static roles created from component manifest
   └─> Dynamic roles enabled via TMF672 API (optional)
```

### Dynamic Role Management (TMF672)

**Event-Driven Synchronization**:

```
Business Admin creates role via TMF672 API
    ↓
Component publishes permissionSpecificationSet event
    ↓
Identity Operator receives event notification
    ↓
Identity Operator calls IDM to create role
    ↓
Role immediately available for assignment
```

**Benefits**:
- Self-service role management
- No manual IDM configuration
- Instant role availability
- Audit trail of all changes

### Uninstall Process

```
1. Component Operator initiates deletion
2. Identity Operator receives on.delete event
3. IDM cleanup:
   ├─> Delete OAuth2 client
   ├─> Delete system user
   └─> Delete all associated roles
4. Secret Store cleanup
5. Component removed from cluster
```

## Authentication Flows

### User Authentication (Engagement Domain)

```
1. User accesses portal
2. Redirected to IDM login page (OAuth2 Authorization Code flow)
3. User authenticates with credentials
4. IDM redirects back with authorization code
5. Portal exchanges code for access token
6. Portal calls APIs with JWT token in Authorization header
7. API Gateway validates token with IDM
8. Request forwarded to appropriate component
```

### Service-to-Service Authentication

```
1. Component A needs to call Component B
2. Component A retrieves credentials from Secret Store
3. Component A requests token from IDM (Client Credentials flow)
4. IDM validates client credentials
5. IDM issues JWT access token
6. Component A calls Component B with token
7. API Gateway validates token
8. Request authorized based on component's roles
```

## Security Controls

### Network Security
- Mutual TLS (mTLS) for service-to-service communication
- Network policies for pod-to-pod isolation
- Ingress/egress rules per component
- DDoS protection at API Gateway

### Application Security
- Input validation and sanitization
- SQL injection prevention
- XSS protection
- CSRF tokens for state-changing operations
- Secure headers (HSTS, CSP, X-Frame-Options)

### Data Security
- Encryption at rest (database, storage)
- Encryption in transit (TLS 1.3)
- Sensitive data masking in logs
- PII handling per data protection regulations
- Data retention and deletion policies

### Access Control
- Role-Based Access Control (RBAC)
- Attribute-Based Access Control (ABAC) where needed
- Principle of least privilege
- Regular access reviews
- Segregation of duties

### Audit and Compliance
- Comprehensive audit logging
- Tamper-proof log storage
- Log retention per compliance requirements
- Security event monitoring (SIEM integration)
- Regular security assessments

### NIST Cybersecurity Framework 2.0 Alignment

This architecture is designed to align with NIST Cybersecurity Framework (CSF) 2.0 across all six core functions:

**GOVERN (GV)**:
- Clear organizational context with domain-driven design
- Risk management through Zero Trust Architecture
- Role-based access control via TMF672
- Identity Operator provides centralized IAM governance

**IDENTIFY (ID)**:
- Component inventory via Kubernetes Custom Resources
- Asset tracking through TMF638/TMF639 APIs
- Threat modeling with STRIDE analysis
- Continuous vulnerability scanning

**PROTECT (PR)**:
- OAuth2/OIDC for identity management and authentication
- JWT token-based authorization on every API call
- Encryption at rest and in transit (TLS 1.3, mTLS)
- High availability and resilience patterns

**DETECT (DE)**:
- Prometheus metrics monitoring (port 4000)
- Centralized logging with Elasticsearch
- Distributed tracing with Jaeger
- Service mesh telemetry (Istio/Linkerd)
- Real-time event correlation via Kafka

**RESPOND (RS)**:
- Incident response playbooks
- Automated credential revocation
- Network isolation via Kubernetes policies
- Evidence preservation in immutable logs

**RECOVER (RC)**:
- Automated backup and restore procedures
- Defined RTO/RPO objectives
- Multi-zone deployment for disaster recovery
- Post-incident review processes

**Detailed Mapping**: See `docs/SECURITY-NIST-CSF-MAPPING.md` for comprehensive control mappings, gap analysis, and implementation recommendations.

### Layered Compliance Approach

This architecture implements a three-layer compliance model that separates jurisdiction-specific requirements from the reusable implementation:

```
┌─────────────────────────────────────────────────────────┐
│  Layer 1: Jurisdiction Requirements (Local)            │
│  - Example Country: Cyber Security Act 2024, PDPA, AIGE      │
│  - France: GDPR, NIS2 Directive, ANSSI                │
│  - Singapore: CSA Cybersecurity Act, PDPA             │
└─────────────────────┬───────────────────────────────────┘
                      │ maps to
                      ▼
┌─────────────────────────────────────────────────────────┐
│  Layer 2: Voluntary Framework (Universal)              │
│  - NIST Cybersecurity Framework 2.0                    │
│  - ISO 27001/27701                                     │
│  - CIS Controls                                        │
└─────────────────────┬───────────────────────────────────┘
                      │ implemented by
                      ▼
┌─────────────────────────────────────────────────────────┐
│  Layer 3: ODA Implementation (Reusable)                │
│  - OAuth2/OIDC, Encryption, Monitoring, etc.          │
│  - Same across all deployments                        │
└─────────────────────────────────────────────────────────┘
```

**Benefits**:
1. **Reusability**: Same ODA implementation serves multiple jurisdictions
2. **Compliance Efficiency**: Map local requirements to existing NIST CSF 2.0 controls
3. **Audit Simplification**: Demonstrate compliance by referencing framework mapping
4. **Future-Proofing**: New regulations map to established frameworks

**Example**:

**Example Country (Example MNO)**:
- Requirements: Cyber Security Act 2024 requires 24-hour incident reporting
- Maps to: NIST CSF 2.0 RS.MA-01 (Incident Management)
- Implementation: Elasticsearch SIEM with automated alerting (already deployed)
- Evidence: `docs/compliance/malaysia/REGULATORY-COMPLIANCE.md`

**France (Orange)** (hypothetical):
- Requirements: NIS2 Directive requires incident reporting within 24 hours
- Maps to: NIST CSF 2.0 RS.MA-01 (Incident Management)
- Implementation: Same Elasticsearch SIEM (reused from Example MNO)
- Evidence: `docs/compliance/france/REGULATORY-COMPLIANCE.md` (to be created)

**Documentation Structure**:
```
docs/compliance/
├── malaysia/
│   └── REGULATORY-COMPLIANCE.md  (CSA 2024, PDPA, AIGE)
├── france/                        (Future: NIS2, GDPR, ANSSI)
├── singapore/                     (Future: CSA Act, PDPA)
└── README.md                      (Compliance framework overview)
```

This approach allows Orange, TM, and YTL implementations to reference their own jurisdiction-specific compliance documents while sharing the same underlying ODA architecture and NIST CSF 2.0 mapping.

## Deployment Patterns

### Single-Tenant Deployment
- Dedicated infrastructure per customer
- Complete isolation
- Custom configurations per tenant
- Higher resource cost

### Multi-Tenant Deployment
- Shared infrastructure
- Logical isolation via OAuth2 clients
- Shared but isolated data
- Cost-effective for SaaS

### Hybrid Deployment
- Shared SoE and SoI
- Dedicated SoR per tenant
- Balance of isolation and efficiency
- Common for enterprise deployments

## Integration with GSMA Open Gateway

This architecture supports GSMA Open Gateway Network APIs:

### Developer Portal Integration
```
Developer Portal (Engagement Domain)
    ↓ (Registration)
Party Domain (TMFC028, TMFC035)
    ↓ (OAuth2 client creation)
Identity Operator → IDM
    ↓ (API access)
API Gateway → Production Domain (Network APIs)
```

### Network API Exposure
- QoD (Quality on Demand) API
- Simple Edge Discovery API
- Device Location API
- SIM Swap API
- Number Verification API

### Revenue Models
- Pay-per-use tracking (Commerce Domain)
- Usage analytics (Intelligence Domain)
- Billing integration (Commerce Domain)

## Performance Considerations

### Scalability
- Horizontal scaling of stateless components
- Database read replicas for read-heavy domains
- Caching layers (Redis) for frequently accessed data
- Event streaming (Kafka) for async communication

### High Availability
- Multi-zone deployment
- Database replication
- Circuit breakers for fault tolerance
- Graceful degradation

### Monitoring
- Application metrics (Prometheus)
- Distributed tracing (Jaeger)
- Log aggregation (ELK stack)
- APM tools (Datadog, New Relic)

## Technology Stack Recommendations

### Container Orchestration
- Kubernetes 1.25+
- Helm charts for deployment
- Operators for lifecycle management

### Identity Management
- Keycloak (open source)
- Azure Active Directory B2C
- Auth0
- Okta

### API Gateway
- Kong
- Apigee
- AWS API Gateway
- Azure API Management

### Database
- PostgreSQL (Party, Commerce data)
- MongoDB (Product catalog, flexible schemas)
- Cassandra (Service inventory, high write throughput)

### Messaging
- Apache Kafka (event streaming)
- RabbitMQ (message queuing)
- NATS (lightweight messaging)

### Observability
- Prometheus + Grafana (metrics)
- ELK/EFK stack (logging)
- Jaeger (distributed tracing)

## Future Enhancements

1. **Service Mesh**: Istio/Linkerd for advanced traffic management
2. **Policy Engine**: Open Policy Agent (OPA) for fine-grained authorization
3. **GitOps**: ArgoCD/Flux for declarative deployments
4. **Chaos Engineering**: Chaos Mesh for resilience testing
5. **AI-Driven Operations**: AIOps for predictive maintenance

## ODA Component Repository

This reference architecture includes actual ODA Canvas component specifications that can be used as templates for implementation.

### Repository Structure

**Reference Components** (`components/oda-canvas/`):
- TM Forum ODA Canvas component YAML specifications
- Sourced from official ODA production repositories
- Follow ODA component specification v1 standard
- Include all functional blocks: coreFunction, securityFunction, managementFunction, eventNotification

**Client Implementations** (`components/<client-name>/`):
- Client-specific Helm values and configurations
- Production-ready deployment parameters
- Environment-specific integrations (IDM, Event Bus, Service Mesh, API Gateway)
- Security and compliance configurations
- Monitoring and observability setup

### Adding Components

To add additional ODA components to this repository:

1. **Fetch Component Specification**:
   ```bash
   # Example: Download component from ODA production S3
   curl -o components/oda-canvas/TMFC028-PartyManagement.yaml \
     https://oda-production.s3.eu-west-2.amazonaws.com/v1.0.0/TMFC028-PartyManagement.yaml
   ```

2. **Create Client Configuration**:
   ```bash
   # Create client-specific values file
   touch components/example-mno/tmfc028-values.yaml
   ```

3. **Update Documentation**:
   - Add component details to README.md
   - Document component in ARCHITECTURE.md
   - Update client README with deployment instructions

### Deployment Workflow

```
1. Select component from components/oda-canvas/
2. Customize with client-specific values from components/<client>/
3. Deploy to Kubernetes using Helm:
   helm install <release-name> <chart> -f components/<client>/<component>-values.yaml
4. Identity Operator provisions OAuth2 client in IDM
5. Component registers APIs with API Gateway
6. Component ready for use
```

## References

### TM Forum Standards
- [TM Forum ODA Documentation](https://projects.tmforum.org/wiki/display/PUB/Open+Digital+Architecture)
- [TM Forum ODA Component Specifications](https://oda-production.s3.eu-west-2.amazonaws.com/index.html)
- [TMF Open API Table](https://www.tmforum.org/open-apis/open-api-table/)
- [TMF672 User Role Permission API](https://www.tmforum.org/resources/specification/tmf672-user-role-permission-api-rest-specification/)
- [TMF669 Party Role Management API](https://www.tmforum.org/resources/specification/tmf669-party-role-management-api-rest-specification/)

### Security Standards
- [NIST Cybersecurity Framework 2.0](https://www.nist.gov/cyberframework)
- [OAuth 2.0 RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749)
- [OpenID Connect Specification](https://openid.net/specs/openid-connect-core-1_0.html)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [CIS Kubernetes Benchmark](https://www.cisecurity.org/benchmark/kubernetes)

### Industry Initiatives
- [GSMA Open Gateway Initiative](https://www.gsma.com/futurenetworks/open-gateway/)
- [CNCF Security Whitepaper](https://www.cncf.io/reports/cloud-native-security-whitepaper/)

### Cloud Native Patterns
- [Kubernetes Operators Pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)
- [Service Mesh Comparison](https://servicemesh.es/)

---

**Document Version**: 1.0  
**Last Updated**: November 2024  
**Author**: Vpnet Cloud Solutions
