# CardDemo Mainframe Modernization Analysis & Wave Plan

## Executive Summary

This document presents a comprehensive analysis of the AWS CardDemo mainframe application for modernization planning. Based on domain decomposition, complexity metrics, dependency analysis, and AWS best practices, we propose a 6-wave modernization approach using the **Strangler Fig Pattern** combined with domain-driven design principles.

---

## 1. Domain Complexity Analysis

### 1.1 Domain Summary by Complexity

| Domain | Files | LOC | Avg Complexity | Seeds | Risk Level |
|--------|-------|-----|----------------|-------|------------|
| Transaction Processing & Management | 47 | 5,859 | 55 | 5 | **HIGH** |
| Credit Card Management | 15 | 5,572 | 97 | 3 | **HIGH** |
| Account Management | 13 | 9,642 | 125 | 3 | **CRITICAL** |
| Authorization Processing | 22 | 5,500 | 44 | 2 | **HIGH** |
| Customer Service & Payments | 4 | 856 | 50 | 1 | **LOW** |
| User Administration & Security | 18 | 4,200 | 41 | 5 | **MEDIUM** |
| Reporting & Analytics | 12 | 2,536 | 37 | 4 | **MEDIUM** |
| System Navigation & Interface | 11 | 2,146 | 30 | 3 | **LOW** |
| Data Management & File Operations | 30 | 2,800 | 28 | 5 | **MEDIUM** |
| Reference Data Management | 18 | 3,500 | 77 | 3 | **MEDIUM** |
| System Administration & Infrastructure | 40 | 2,500 | 2 | 5 | **LOW** |
| Shared Services | 12 | 800 | 8 | 0 | **LOW** |

### 1.2 High-Complexity Programs (Cyclomatic Complexity > 100)

| Program | Domain | Complexity | LOC | Risk |
|---------|--------|------------|-----|------|
| **COACTUPC.cbl** | Account Management | 373 | 4,237 | CRITICAL |
| **COTRTLIC.cbl** | Reference Data | 221 | 2,099 | HIGH |
| **COTRTUPC.cbl** | Reference Data | 208 | 1,703 | HIGH |
| **COCRDUPC.cbl** | Credit Card Management | 170 | 1,561 | HIGH |
| **COCRDLIC.cbl** | Credit Card Management | 136 | 1,460 | HIGH |

**Key Finding**: Account Management's COACTUPC program has the highest complexity (373) and handles both account AND customer data updates - making it the highest-risk refactoring target.

---

## 2. Dependency Analysis

### 2.1 Data Source Dependencies (VSAM/DB2 Files)

Based on `program_to_dsn.json` analysis:

#### Core Master Files (Highest Coupling)

| Data Source | Type | Programs Accessing | Write Access | Criticality |
|-------------|------|-------------------|--------------|-------------|
| **AWS.M2.CARDDEMO.ACCTDATA.VSAM.KSDS** | VSAM-KSDS | 15+ | COACTUPC, CBACT04C, CBTRN02C, COBIL00C | **CRITICAL** |
| **AWS.M2.CARDDEMO.CUSTDATA.VSAM.KSDS** | VSAM-KSDS | 10+ | COACTUPC | **CRITICAL** |
| **AWS.M2.CARDDEMO.CARDXREF.VSAM.KSDS** | VSAM-KSDS | 12+ | None (Read-only) | **HIGH** |
| **AWS.M2.CARDDEMO.CARDDATA.VSAM.KSDS** | VSAM-KSDS | 8+ | COCRDUPC | **HIGH** |
| **AWS.M2.CARDDEMO.TRANSACT.VSAM.KSDS** | VSAM-KSDS | 8+ | COBIL00C, COTRN02C, CBTRN02C | **HIGH** |
| **AWS.M2.CARDDEMO.USRSEC.VSAM.KSDS** | VSAM-KSDS | 6 | COUSR01C, COUSR02C, COUSR03C | **HIGH** |
| **CARDDEMO.TRANSACTION_TYPE** | DB2 | 3 | COTRTUPC, COBTUPDT, COTRTLIC | **MEDIUM** |
| **CARDDEMO.AUTHFRDS** | DB2 | 1 | COPAUS2C | **MEDIUM** |

#### Data Access Patterns

```
ACCTDATA (Account Master)
├── READ: COACTVWC, COACCT01, CBSTM03B, CBEXPORT, COPAUA0C, COPAUS0C
├── READ/UPDATE/WRITE: COACTUPC, CBACT04C, CBTRN02C, COBIL00C
└── Downstream: Statement generation, Interest calc, Auth processing

CUSTDATA (Customer Master)
├── READ: COACTVWC, CBCUS01C, CBSTM03B, CBEXPORT, COPAUA0C, COPAUS0C
├── READ/UPDATE/WRITE: COACTUPC
└── Downstream: Account services, Authorization

CARDXREF (Cross-Reference)
├── READ: All account/card lookup operations
├── PATH ACCESS: CXACAIX alternate index
└── Critical for: Account-to-card relationships
```

### 2.2 Program Call Dependencies

Based on entrypoint JSON analysis:

#### Fan-In Analysis (Most Called Programs)

| Program | Called By | Function | Modernization Impact |
|---------|-----------|----------|---------------------|
| **COSGN00C** | All online transactions | Authentication | Must modernize first |
| **COMEN01C** | 10+ programs | Main Menu | Navigation gateway |
| **COADM01C** | 8+ programs | Admin Menu | Admin gateway |
| **CSUTLDTC** | Multiple programs | Date utilities | Shared service |

#### Call Chain Depth

```
Sign-on Flow:
COSGN00C → COMEN01C (regular users)
         → COADM01C (admin users)

Transaction Flow (Example: Account Update):
COACTUPC → COMEN01C → COSGN00C → COADM01C

Authorization Flow:
COPAUS0C → COPAUS1C → COPAUS2C (Fraud check via DB2)
```

### 2.3 Technology Stack Dependencies

| Technology | Programs | Modernization Challenge |
|------------|----------|------------------------|
| **CICS Online** | 25+ programs | UI replacement, session management |
| **VSAM KSDS** | 50+ programs | Data migration to RDS/Aurora |
| **VSAM PATH** | 5+ programs | Index restructuring |
| **DB2** | 4 programs | Straightforward migration |
| **IMS (via MQ)** | 3 programs | Complex - requires MQ bridge |
| **GDG (Generations)** | 5+ batch jobs | Replace with S3 versioning |
| **BMS Maps** | 20+ screens | Angular/React UI rebuild |

---

## 3. Target Architecture Principles

Based on AWS Mainframe Modernization best practices and re:Invent 2025 guidance:

### 3.1 Core Architectural Principles

1. **Domain-Driven Design (DDD)**
   - Decompose monolith into bounded contexts aligned with business domains
   - Each domain becomes a candidate microservice or service module
   - Use Anti-Corruption Layer (ACL) at domain boundaries

2. **Strangler Fig Pattern**
   - Incrementally replace functionality while keeping legacy running
   - Route traffic through API Gateway to either legacy or new services
   - Gradually increase traffic to modernized components

3. **Event-Driven Architecture**
   - Replace tight coupling with asynchronous event messaging
   - Use Amazon EventBridge for domain events
   - Enable loose coupling between services

4. **API-First Design**
   - Expose all functionality through REST/GraphQL APIs
   - Use Amazon API Gateway for routing and management
   - Enable future integration flexibility

### 3.2 Target Technology Stack

| Current | Target | Service |
|---------|--------|---------|
| CICS Online | Containerized Java/Spring Boot | Amazon ECS/EKS |
| BMS Screens | Angular/React SPA | Amazon CloudFront + S3 |
| VSAM Files | Amazon Aurora PostgreSQL | Amazon RDS |
| DB2 Tables | Amazon Aurora PostgreSQL | Amazon RDS |
| IMS Database | Amazon DynamoDB or Aurora | AWS Database Migration |
| JCL Batch | AWS Step Functions + Lambda | Serverless |
| COBOL Programs | Java (via AWS Blu Age) | Automated refactoring |
| MQ Integration | Amazon MQ or SQS | Managed messaging |

### 3.3 Data Migration Strategy

**Phase 1: Dual-Write Pattern**
- Modernized services write to both legacy VSAM and new Aurora
- Validate data consistency

**Phase 2: Read from New**
- Switch reads to Aurora while maintaining legacy writes
- Verify functional equivalence

**Phase 3: Full Cutover**
- Decommission legacy data stores
- Archive historical VSAM data to S3

---

## 4. Modernization Wave Plan

### Wave 0: Foundation & Infrastructure (Pre-requisite)

**Objective**: Establish cloud foundation and tooling

| Component | Action | AWS Service |
|-----------|--------|-------------|
| Landing Zone | Set up multi-account structure | AWS Control Tower |
| Networking | VPC, subnets, connectivity | AWS Transit Gateway |
| Security | IAM, secrets management | AWS IAM, Secrets Manager |
| CI/CD | Pipeline infrastructure | AWS CodePipeline, CodeBuild |
| Monitoring | Observability platform | CloudWatch, X-Ray |

**Deliverables**:
- AWS landing zone configured
- CI/CD pipelines operational
- Monitoring dashboards ready

---

### Wave 1: Shared Services & Authentication

**Objective**: Establish foundation services that all other domains depend on

**Domain**: Shared Services + User Administration & Security (partial)

| Program | Current Function | Target Service |
|---------|-----------------|----------------|
| COSGN00C | Sign-on/Authentication | Amazon Cognito + Custom Auth Service |
| CSUTLDTC | Date utilities | Java utility library |
| COMEN01C | Main menu | React SPA routing |
| COADM01C | Admin menu | React SPA routing |
| USRSEC file | User credentials | Amazon Cognito user pool |

**Approach**:
1. Implement Amazon Cognito for authentication
2. Create Java Auth Service wrapping Cognito
3. Build React shell application with routing
4. Deploy API Gateway with Cognito authorizer
5. Use Strangler: Route new auth through Cognito, legacy through CICS

**Dependencies Resolved**:
- All online programs depend on COSGN00C
- Breaking this dependency enables parallel development

**Complexity**: LOW-MEDIUM
**Risk**: MEDIUM (all systems depend on auth)

---

### Wave 2: Reference Data Management

**Objective**: Modernize configuration and lookup data

**Domain**: Reference Data Management

| Program | Current Function | Target Service |
|---------|-----------------|----------------|
| COTRTLIC | Transaction type list | Reference Data Service |
| COTRTUPC | Transaction type maintenance | Reference Data Service |
| COBTUPDT | Batch type updates | Lambda function |

**Data Migration**:
- CARDDEMO.TRANSACTION_TYPE → Aurora PostgreSQL
- DISCGRP, TRANCATG, TCATBALF, TRANTYPE VSAM files → Aurora tables

**Approach**:
1. Create Reference Data microservice (Spring Boot)
2. Migrate DB2 transaction types to Aurora
3. Build REST APIs for CRUD operations
4. Implement caching layer (ElastiCache)
5. Update consumers to use new APIs

**Dependencies**:
- Low coupling to other domains
- Read-only consumers can be updated incrementally

**Complexity**: MEDIUM
**Risk**: LOW

---

### Wave 3: Account & Customer Management

**Objective**: Modernize core account and customer data management

**Domain**: Account Management

| Program | Current Function | Target Service |
|---------|-----------------|----------------|
| COACTVWC | Account view | Account Service (read) |
| COACTUPC | Account/Customer update | Account Service (write) |
| COACCT01 | Account inquiry (MQ) | Account Service API |

**Data Migration**:
- ACCTDATA VSAM → Aurora accounts table
- CUSTDATA VSAM → Aurora customers table
- CARDXREF VSAM → Aurora card_xref table

**Approach**:
1. Create Account Service microservice
2. Create Customer Service microservice
3. Implement dual-write to legacy VSAM and Aurora
4. Build comprehensive validation layer (replicate COACTUPC business rules)
5. Extensive testing due to high complexity (CC=373)

**Critical Considerations**:
- COACTUPC has highest complexity - needs careful decomposition
- Split into Account Service and Customer Service
- Implement saga pattern for cross-service updates

**Complexity**: CRITICAL
**Risk**: HIGH

---

### Wave 4: Credit Card Management

**Objective**: Modernize card lifecycle management

**Domain**: Credit Card Management

| Program | Current Function | Target Service |
|---------|-----------------|----------------|
| COCRDLIC | Card list/search | Card Service |
| COCRDSLC | Card detail view | Card Service |
| COCRDUPC | Card update | Card Service |

**Data Migration**:
- CARDDATA VSAM → Aurora cards table
- CARDAIX path → Aurora secondary indexes

**Approach**:
1. Create Card Service microservice
2. Implement card lifecycle state machine
3. Event-driven updates to related services
4. PCI-DSS compliance considerations

**Dependencies**:
- Requires Account Service from Wave 3
- Card-Account relationships via CARDXREF

**Complexity**: HIGH
**Risk**: MEDIUM

---

### Wave 5: Transaction Processing

**Objective**: Modernize transaction lifecycle

**Domain**: Transaction Processing & Management + Customer Service & Payments

| Program | Current Function | Target Service |
|---------|-----------------|----------------|
| COTRN00C | Transaction browse | Transaction Service |
| COTRN01C | Transaction detail | Transaction Service |
| COTRN02C | Transaction add | Transaction Service |
| COBIL00C | Bill payment | Payment Service |
| CBTRN02C | Batch posting | Step Functions workflow |
| CBACT04C | Interest calculation | Lambda function |

**Data Migration**:
- TRANSACT VSAM → Aurora transactions table
- GDG files → S3 with versioning

**Approach**:
1. Create Transaction Service microservice
2. Create Payment Service microservice
3. Implement event-driven transaction posting
4. Replace JCL batch with Step Functions
5. Interest calculation as scheduled Lambda

**Dependencies**:
- Requires Account Service (balance updates)
- Requires Card Service (card validation)

**Complexity**: HIGH
**Risk**: HIGH

---

### Wave 6: Authorization Processing & Reporting

**Objective**: Modernize real-time authorization and batch reporting

**Domain**: Authorization Processing + Reporting & Analytics

| Program | Current Function | Target Service |
|---------|-----------------|----------------|
| COPAUA0C | Auth request (MQ) | Authorization Service |
| COPAUS0C | Auth summary | Authorization Service |
| COPAUS1C | Auth detail | Authorization Service |
| COPAUS2C | Fraud check | Fraud Detection Service |
| CORPT00C | Report menu | Reporting Service |
| CBSTM03A/B | Statement generation | Statement Service |

**Data Migration**:
- IMS PAUTSUM/PAUTDTL → DynamoDB (high throughput)
- AUTHFRDS DB2 → Aurora (fraud data)

**Approach**:
1. Create Authorization Service (low latency critical)
2. Create Fraud Detection Service with ML integration
3. Implement Amazon SageMaker for fraud scoring
4. Replace MQ with Amazon MQ (ActiveMQ)
5. Reporting via Amazon QuickSight

**Special Considerations**:
- Sub-second latency requirements for auth
- Consider Amazon ElastiCache for auth decisions
- ML-based fraud detection enhancement opportunity

**Complexity**: HIGH
**Risk**: MEDIUM

---

## 5. Wave Dependency Matrix

```
Wave 0 (Foundation)
    │
    ▼
Wave 1 (Auth & Shared) ──────────────────────────────┐
    │                                                 │
    ├──────────────┬──────────────┐                  │
    ▼              ▼              ▼                  │
Wave 2         Wave 3         Wave 4                 │
(Reference)    (Account)      (Card)                 │
    │              │              │                  │
    └──────────────┴──────────────┘                  │
                   │                                  │
                   ▼                                  │
              Wave 5 ◄────────────────────────────────┘
         (Transactions)
                   │
                   ▼
              Wave 6
         (Auth & Reports)
```

---

## 6. Risk Assessment & Mitigation

### 6.1 High-Risk Items

| Risk | Impact | Mitigation |
|------|--------|------------|
| COACTUPC complexity (CC=373) | Data corruption | Extensive unit testing, parallel running |
| ACCTDATA concurrent access | Data inconsistency | Implement optimistic locking |
| Authentication cutover | System-wide outage | Feature flags, gradual rollout |
| IMS to DynamoDB migration | Auth latency | Performance testing, caching |
| Business logic extraction | Functional regression | Automated test generation |

### 6.2 Recommended Testing Strategy

1. **Unit Tests**: Generate from COBOL specifications using AWS Transform
2. **Integration Tests**: API contract testing between services
3. **Parallel Running**: Run legacy and modern side-by-side, compare results
4. **Performance Tests**: Ensure latency requirements met
5. **Chaos Engineering**: Test resilience with failure injection

---

## 7. Success Metrics

| Metric | Current (Estimated) | Target |
|--------|---------------------|--------|
| Deployment Frequency | Monthly | Daily |
| Lead Time for Changes | 4-6 weeks | < 1 week |
| Mean Time to Recovery | Hours | Minutes |
| Infrastructure Cost | $X/month | 60-90% reduction |
| Developer Productivity | Baseline | 3x improvement |
| Test Coverage | Unknown | > 80% |

---

## 8. Recommendations

### Immediate Actions

1. **Validate dependencies file**: Ensure `dependencies_20260121105446.json` is available for detailed program-level dependency mapping
2. **Engage AWS Transform**: Use AWS Transform for automated code analysis and refactoring
3. **Establish test baseline**: Generate automated tests from existing COBOL behavior
4. **Begin Wave 0**: Start cloud foundation setup immediately

### Architecture Decisions Needed

1. **Refactor vs Replatform**: Given high complexity, recommend automated refactoring via AWS Blu Age
2. **Database Choice**: Aurora PostgreSQL recommended for VSAM migration (ACID compliance)
3. **Event Bus**: Amazon EventBridge for domain events
4. **API Strategy**: REST APIs with OpenAPI specifications

### Team Structure

- **Platform Team**: AWS infrastructure, CI/CD, security
- **Domain Teams**: One per major domain (Account, Card, Transaction, Auth)
- **Data Team**: Migration, validation, quality assurance
- **QA Team**: Test automation, parallel validation

---

## Appendix A: Data Source to Domain Mapping

| Data Source | Primary Domain | Secondary Domains |
|-------------|----------------|-------------------|
| ACCTDATA | Account Management | Transaction, Auth, Reporting |
| CUSTDATA | Account Management | Auth, Reporting |
| CARDDATA | Credit Card Management | Transaction |
| CARDXREF | Credit Card Management | Account, Transaction |
| TRANSACT | Transaction Processing | Reporting |
| USRSEC | User Administration | All online |
| TRANSACTION_TYPE | Reference Data | Transaction |
| AUTHFRDS | Authorization | None |

## Appendix B: Program to Wave Mapping

| Wave | Programs |
|------|----------|
| 1 | COSGN00C, COMEN01C, COADM01C, CSUTLDTC, COBSWAIT |
| 2 | COTRTLIC, COTRTUPC, COBTUPDT, DISCGRP.jcl, TRANCATG.jcl |
| 3 | COACTVWC, COACTUPC, COACCT01, CBCUS01C, CBACT01C-03C |
| 4 | COCRDLIC, COCRDSLC, COCRDUPC |
| 5 | COTRN00C, COTRN01C, COTRN02C, COBIL00C, CBTRN01C-03C, CBACT04C |
| 6 | COPAUA0C, COPAUS0C-2C, CORPT00C, CBSTM03A/B |

---

*Document generated: 2026-01-21*
*Based on: Domain decomposition analysis, program_to_dsn.json, entrypoint JSON files, AWS re:Invent 2025 best practices*
