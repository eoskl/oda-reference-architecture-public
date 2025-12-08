# TM Forum ODA Reference Architecture

**Open Digital Architecture (ODA) reference implementation with integrated identity and access management, security controls, and regulatory compliance framework.**

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![TM Forum](https://img.shields.io/badge/TM%20Forum-ODA-orange.svg)](https://www.tmforum.org/oda/)
[![NIST CSF 2.0](https://img.shields.io/badge/NIST%20CSF-2.0-green.svg)](https://www.nist.gov/cyberframework)

## Overview

This repository provides a production-ready reference architecture for deploying TM Forum Open Digital Architecture (ODA) with:

- **Component-Based Architecture**: Modular design following TM Forum ODA Canvas principles
- **Integrated Identity Management**: OAuth2/OIDC with dynamic role management (TMF672)
- **Security Framework**: NIST Cybersecurity Framework 2.0 alignment across all 6 core functions
- **Regulatory Compliance**: Layered model separating jurisdiction-specific requirements from reusable implementation
- **Breach Response**: Automated detection, compliance workflows, and incident response procedures

### What's Included

✅ **Architecture Documentation**: Detailed technical documentation with diagrams
✅ **ODA Component Specifications**: TMF approved component manifests (TMFC035 example)
✅ **Security Mapping**: Complete NIST CSF 2.0 control mappings
✅ **Compliance Framework**: Example jurisdiction compliance (adaptable to any country)
✅ **Implementation Examples**: Production-ready configurations for mobile network operators
✅ **Breach Simulations**: Real-world scenarios with bottom-up compliance flow
✅ **Alert Configurations**: Prometheus and SIEM rules with regulatory mappings

## Architecture Layers

### System of Engagement (SoE)
- **Engagement Domain**: Customer-facing channels, omnichannel experience, digital touchpoints

### System of Records (SoR)
- **Party Domain**: Customer and partner management (TMFC028, TMFC035, TMFC020, TMFC039)
- **Commerce Domain**: Product catalog, order management, inventory (TMFC001, TMFC002, TMFC005)
- **Production Domain**: Service and resource management (TMFC008, TMFC012)

### System of Innovation (SoI)
- **Intelligence Domain**: Analytics, AI/ML, predictive intelligence, automation

## Key Features

### 1. Layered Compliance Model

```
┌─────────────────────────────────────────────────────────┐
│  Layer 1: Jurisdiction Requirements (Local)            │
│  - Country-specific regulations                        │
│  - Industry-specific compliance                        │
└─────────────────────┬───────────────────────────────────┘
                      │ maps to
                      ▼
┌─────────────────────────────────────────────────────────┐
│  Layer 2: Voluntary Framework (Universal)              │
│  - NIST Cybersecurity Framework 2.0                    │
│  - ISO 27001/27701                                     │
└─────────────────────┬───────────────────────────────────┘
                      │ implemented by
                      ▼
┌─────────────────────────────────────────────────────────┐
│  Layer 3: ODA Implementation (Reusable)                │
│  - Same architecture across all jurisdictions          │
└─────────────────────────────────────────────────────────┘
```

**Benefits**:
- ✅ Reusable implementation across multiple countries
- ✅ Simplified compliance audits
- ✅ Consistent security controls
- ✅ Faster time to market in new jurisdictions

### 2. NIST Cybersecurity Framework 2.0 Alignment

This architecture implements all six NIST CSF 2.0 core functions:

- **GOVERN (GV)**: Risk management, roles & responsibilities, supply chain security
- **IDENTIFY (ID)**: Asset management, risk assessment, continuous improvement
- **PROTECT (PR)**: Identity management, data security, platform security, resilience
- **DETECT (DE)**: Continuous monitoring, adverse event analysis, threat detection
- **RESPOND (RS)**: Incident management, analysis, mitigation, communication
- **RECOVER (RC)**: Recovery planning, incident recovery, business continuity

**Documentation**: `docs/SECURITY-NIST-CSF-MAPPING.md` provides comprehensive control mappings with 700+ lines of detailed implementation guidance.

### 3. Identity & Access Management

**OAuth2/OIDC Integration**:
- External Identity Provider (IDM Platform, Azure AD, Okta)
- Automated OAuth2 client provisioning via Identity Operator
- TMF672 API for dynamic role management
- JWT token-based authentication on every API call

**Zero Trust Architecture**:
- "Never Trust, Always Verify" principle
- Least privilege access control
- Continuous token validation
- Network micro-segmentation

### 4. ODA Canvas Components

**TMFC035 - Permissions Management** (included as example):
- TMF672 User Role Permission Management API
- TMF669 Party Role Management API
- Event-driven role synchronization
- OAuth2/OIDC sidecar pattern
- Prometheus metrics integration

**Component Repository**: `components/oda-canvas/` contains TM Forum approved specifications.

### 5. Example Implementation

**Example MNO Configuration** (`components/example-mno/`):
- Production-ready Helm values
- IDM Platform integration
- Apache Kafka event bus
- Istio service mesh (mTLS)
- Kong API Gateway
- Prometheus + Elasticsearch + Jaeger observability stack
- High availability configuration (3 replicas, multi-zone)

**Note**: This is a generic example. Adapt it to your specific environment.

## Directory Structure

```
oda-reference-architecture/
├── README.md                           # This file
├── LICENSE                             # Apache 2.0 License
├── CLAUDE.md                           # AI assistant integration guidance
├── components/                         # ODA Component specifications
│   ├── oda-canvas/                    # Reference ODA Canvas components
│   │   └── TMFC035-PermissionsManagement.yaml
│   └── example-mno/                   # Example MNO implementation
│       ├── README.md
│       ├── tmfc035-values.yaml
│       └── alert-mapping-config.yaml
├── diagrams/
│   ├── architecture/                   # Production-ready SVG diagrams
│   │   ├── oda-architecture-presentation-ready.svg
│   │   └── oda-logical-architecture-with-capabilities.svg
│   └── source/                         # Source files (PlantUML)
│       └── configure-users-and-roles.puml
└── docs/                               # Technical documentation
    ├── ARCHITECTURE.md                 # Detailed architecture documentation
    ├── SECURITY-NIST-CSF-MAPPING.md    # NIST CSF 2.0 security mapping
    └── compliance/                     # Jurisdiction-specific compliance
        └── example-jurisdiction/       # Example compliance mapping
            ├── REGULATORY-COMPLIANCE.md
            ├── BREACH-SIMULATION.md
            └── BREACH-RESPONSE-RUNBOOK.md
```

## Getting Started

### 1. Review Architecture

Start with the architecture documentation:
- `docs/ARCHITECTURE.md` - Comprehensive technical documentation
- `diagrams/architecture/` - Visual architecture diagrams

### 2. Understand Security Framework

Review the NIST CSF 2.0 mapping:
- `docs/SECURITY-NIST-CSF-MAPPING.md` - Complete control mappings for all 6 functions

### 3. Explore Component Specifications

Check the ODA Canvas components:
- `components/oda-canvas/TMFC035-PermissionsManagement.yaml` - Example component specification
- Follow TM Forum ODA Canvas component standard v1

### 4. Adapt Example Configuration

Use the example MNO configuration as a starting point:
- `components/example-mno/tmfc035-values.yaml` - Helm values template
- `components/example-mno/alert-mapping-config.yaml` - Monitoring configuration
- Replace example values with your environment details

### 5. Map Your Regulatory Requirements

Create your jurisdiction-specific compliance mapping:
```bash
mkdir -p docs/compliance/your-country
# Map your country's regulations → NIST CSF 2.0 → This architecture
```

See `docs/compliance/example-jurisdiction/REGULATORY-COMPLIANCE.md` for template.

## Use Cases

### Telecommunications Operators
- Mobile network operators (MNOs)
- Mobile virtual network operators (MVNOs)
- Fixed-line telecommunications providers
- Integrated telcos (mobile + fixed + broadband)

### Service Providers
- Cloud service providers
- Managed service providers
- Digital service providers
- Platform-as-a-Service (PaaS) providers

### Enterprise Deployments
- Large enterprises with complex IAM requirements
- Multi-tenant SaaS applications
- Regulated industries (financial services, healthcare, government)

## TMF Capabilities Reference

| Domain | Capability Code | Capability Name |
|--------|----------------|-----------------|
| Party | TMFC028 | Party Management |
| Party | TMFC035 | Party Roles & Permission Management |
| Party | TMFC020 | Digital Identity Management |
| Party | TMFC039 | Agreement Management |
| Commerce | TMFC001 | Product Catalog Management |
| Commerce | TMFC002 | Product Order Capture & Management |
| Commerce | TMFC005 | Product Inventory Management |
| Production | TMFC008 | Service Inventory Management |
| Production | TMFC012 | Resource Inventory Management |

## Security Architecture

### Identity Management Components

1. **External Identity Provider**: OAuth2/OIDC compliant (IDM Platform, Azure AD, Okta, etc.)
2. **ODA Canvas Platform**:
   - Component Operators: Lifecycle management
   - Identity Operator: IAM synchronization
   - Secret Store: Credential management
   - API Gateway: Token validation and authorization

### Authentication Flows

- **Bootstrap**: Automated OAuth2 client provisioning during component installation
- **Dynamic Role Management**: TMF672 API-based role synchronization
- **Cross-Domain Authentication**: JWT token-based service-to-service communication
- **Zero Trust**: All API calls require valid authentication tokens

### NIST Cybersecurity Framework 2.0 Alignment

This architecture is designed to align with NIST CSF 2.0 across all six core functions (Govern, Identify, Protect, Detect, Respond, Recover). The security controls implement industry best practices for:

- Identity and Access Management (OAuth2/OIDC)
- Data Protection (encryption at rest and in transit)
- Continuous Monitoring (metrics, logs, traces)
- Incident Response (automated detection and remediation)
- Business Continuity (backup, disaster recovery)

**Detailed Mapping**: See `docs/SECURITY-NIST-CSF-MAPPING.md` for comprehensive NIST CSF 2.0 control mappings, gap analysis, and compliance guidance.

## Regulatory Compliance

This architecture implements a **layered compliance model** that separates jurisdiction-specific requirements from the reusable implementation:

**Layer 1: Jurisdiction Requirements** (Local)
- Country-specific regulations (cybersecurity acts, data protection laws)
- Industry-specific compliance (telecommunications, finance, healthcare)

**Layer 2: Voluntary Framework** (Universal)
- NIST Cybersecurity Framework 2.0
- ISO 27001/27701
- CIS Controls

**Layer 3: ODA Implementation** (Reusable)
- Same architecture and controls across all jurisdictions

### Example Jurisdiction Compliance

**Example Compliance Documentation** (in `docs/compliance/example-jurisdiction/`):
- Cybersecurity Act compliance
- Data Protection Act compliance
- AI Governance Framework compliance

**Note**: This is a template. Adapt it to your country's specific regulations.

This approach enables organizations in different countries to create their own jurisdiction-specific compliance mappings while reusing the same ODA architecture and NIST CSF 2.0 framework.

## Breach Simulation

The repository includes comprehensive breach simulations demonstrating the bottom-up compliance flow:

**Layer 3 (Implementation)**: Code change or configuration error
    ↓ triggers detection
**Layer 2 (NIST CSF)**: Framework controls detect violation
    ↓ triggers obligation
**Layer 1 (Regulation)**: Legal requirements activated

**Example Scenarios**:
1. **Authentication Bypass**: Development code accidentally deployed to production
2. **Data Exposure**: PII logged without sanitization

Each simulation includes:
- Root cause and timeline
- Detection mechanisms (Prometheus, SIEM)
- NIST CSF control violations
- Regulatory obligations triggered
- Response procedures with commands
- Evidence collection and reporting

See `docs/compliance/example-jurisdiction/BREACH-SIMULATION.md` for complete scenarios.

## Integration Patterns

### GSMA Open Gateway Integration

This architecture supports GSMA Open Gateway Network APIs:
- Developer portal integration via Engagement Domain
- API authentication through centralized IAM
- Network resource exposure via Production Domain
- Usage analytics through Intelligence Domain

### Multi-Tenant Deployments

- Tenant isolation through OAuth2 client separation
- Role-based access control per tenant
- Shared infrastructure with logical separation
- Centralized identity management across tenants

## Technology Stack

### Recommended Technologies

**Container Orchestration**: Kubernetes 1.25+
**Identity Management**: Keycloak, Azure AD, Auth0, Okta
**API Gateway**: Kong, Apigee, AWS API Gateway
**Service Mesh**: Istio, Linkerd
**Monitoring**: Prometheus + Grafana, Elasticsearch + Kibana, Jaeger
**Message Bus**: Apache Kafka, RabbitMQ, NATS
**Database**: PostgreSQL, MongoDB, Cassandra

## Contributing

Contributions are welcome! Please:
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

Licensed under the Apache License, Version 2.0 (the "License");
you may not use these files except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

## References

### TM Forum Standards
- [TM Forum Open Digital Architecture](https://www.tmforum.org/oda/)
- [TM Forum Open APIs](https://www.tmforum.org/open-apis/)
- [TM Forum ODA Component Specifications](https://oda-production.s3.eu-west-2.amazonaws.com/index.html)
- [TMF672 User Role Permission API](https://www.tmforum.org/resources/specification/tmf672-user-role-permission-api-rest-specification/)
- [TMF669 Party Role Management API](https://www.tmforum.org/resources/specification/tmf669-party-role-management-api-rest-specification/)

### Security Standards
- [NIST Cybersecurity Framework 2.0](https://www.nist.gov/cyberframework)
- [OAuth 2.0 RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749)
- [OpenID Connect](https://openid.net/connect/)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [CIS Kubernetes Benchmark](https://www.cisecurity.org/benchmark/kubernetes)

### Industry Initiatives
- [GSMA Open Gateway](https://www.gsma.com/solutions-and-impact/technologies/open-gateway/)
- [CNCF Security Whitepaper](https://www.cncf.io/reports/cloud-native-security-whitepaper/)

### Cloud Native Patterns
- [Kubernetes Operators Pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)
- [Service Mesh Comparison](https://servicemesh.es/)

## Acknowledgments

- **TM Forum**: For Open Digital Architecture specifications and API standards
- **NIST**: For Cybersecurity Framework 2.0
- **CNCF**: For cloud-native security guidance
- **Community Contributors**: Thank you for your contributions!

## Contact

For questions, discussions, or collaboration opportunities:
- Open an issue in this repository
- Star the repository if you find it useful
- Share your implementation experiences

---

**Version**: 1.0.0
**Last Updated**: December 2024
**Maintained by**: Vpnet Cloud Solutions
