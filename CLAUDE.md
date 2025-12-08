# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a **documentation and diagram repository** for TM Forum Open Digital Architecture (ODA) reference architecture with integrated identity and access management. This is NOT a code repository - it contains architecture diagrams (SVG), source diagram files (PlantUML), and documentation (Markdown).

## Repository Structure

```
oda-reference-architecture/
├── diagrams/
│   ├── architecture/          # Production-ready SVG diagrams
│   │   ├── oda-architecture-presentation-ready.svg
│   │   └── oda-logical-architecture-with-capabilities.svg
│   └── source/               # PlantUML source files
│       └── configure-users-and-roles.puml
└── docs/
    └── ARCHITECTURE.md       # Detailed technical documentation
```

## Working with Diagrams

### Viewing Diagrams

SVG diagrams can be viewed directly in browsers or embedded in documentation. All production diagrams are in `diagrams/architecture/`.

### Editing PlantUML Diagrams

Source diagrams are in `diagrams/source/*.puml` format.

**To generate SVG from PlantUML:**
```bash
# Install PlantUML (requires Java)
brew install plantuml

# Generate SVG from .puml file
plantuml -tsvg diagrams/source/configure-users-and-roles.puml

# Or using Docker if PlantUML not installed locally
docker run -v $(pwd):/data plantuml/plantuml -tsvg /data/diagrams/source/configure-users-and-roles.puml
```

**Output:** Generated SVGs should be placed in `diagrams/architecture/` with descriptive names.

## Architecture Overview

### Five Domain Architecture

This reference architecture implements TM Forum ODA across five domains organized in three layers:

**System of Engagement (SoE)**
- Engagement Domain: Customer-facing channels and digital touchpoints

**System of Records (SoR)**
- Party Domain: Customer/partner management (TMFC028, TMFC035, TMFC020, TMFC039)
- Commerce Domain: Product catalog, ordering, inventory (TMFC001, TMFC002, TMFC005)
- Production Domain: Service and resource management (TMFC008, TMFC012)

**System of Innovation (SoI)**
- Intelligence Domain: Analytics, AI/ML, predictive intelligence

### Key Architectural Concepts

**Component-Based Architecture:**
- Each component is independently deployable
- Components interact via standardized TMF Open APIs
- API-first design with loose coupling

**Zero Trust Security:**
- All requests require authentication via OAuth2/OIDC
- JWT token validation on every API call
- Least privilege access control

**Identity Management Lifecycle:**
- Bootstrap: Automated OAuth2 client provisioning during component installation
- Dynamic Role Management: TMF672 API-based role synchronization with IDM
- Uninstall: Automatic cleanup of clients, users, and roles

**ODA Canvas Platform Components:**
- Component Operators: Kubernetes-based lifecycle management
- Identity Operator: Bridges ODA components with external IDM (Keycloak, Azure AD, etc.)
- Secret Store: Secure credential storage
- API Gateway: Token validation and request routing

## TMF Standards Reference

Key TMF APIs used in this architecture:

| API | Domain | Purpose |
|-----|--------|---------|
| TMF632 | Party | Party Management |
| TMF669 | Party | Party Role |
| TMF672 | Party | User Roles and Permissions (dynamic role management) |
| TMF651 | Party | Agreement Management |
| TMF620 | Commerce | Product Catalog |
| TMF622 | Commerce | Product Ordering |
| TMF637 | Commerce | Product Inventory |
| TMF638 | Production | Service Inventory |
| TMF639 | Production | Resource Inventory |

## Integration Patterns

**GSMA Open Gateway:**
This architecture supports GSMA Open Gateway Network APIs with developer portal integration, network API exposure (QoD, Device Location, SIM Swap, etc.), and revenue tracking.

**Multi-Tenant Deployments:**
- Tenant isolation through OAuth2 client separation
- Shared infrastructure with logical separation
- Centralized identity management across tenants

## Common Documentation Tasks

**Adding new architecture diagrams:**
1. Create PlantUML source in `diagrams/source/`
2. Generate SVG using PlantUML
3. Place output in `diagrams/architecture/` with descriptive name
4. Update README.md to reference the new diagram with use cases
5. Update docs/ARCHITECTURE.md if the diagram introduces new concepts

**Updating existing diagrams:**
1. Edit the .puml file in `diagrams/source/`
2. Regenerate SVG
3. Replace existing file in `diagrams/architecture/`
4. Update documentation if concepts changed

**Extending documentation:**
- docs/ARCHITECTURE.md contains detailed technical information
- README.md contains overview and usage information
- Keep both files synchronized when adding new architectural concepts

## Important Notes

- This is a reference architecture repository - implementations would be in separate repositories
- Diagrams are optimized for presentations (PowerPoint, Google Slides, Keynote)
- All security patterns follow OAuth 2.0 (RFC 6749) and OpenID Connect standards
- TMF672 API is central to dynamic role management between components and IDM
- The Identity Operator pattern is key to automating IAM integration
