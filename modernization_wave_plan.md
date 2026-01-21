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

## 8. Domain Complexity vs. Business Value Matrix

Based on in-depth analysis of `dependencies_20260121105446.json`, this matrix provides strategic guidance for modernization prioritization by evaluating each domain's technical complexity against its business value.

### 8.1 Matrix Overview

```
                              BUSINESS VALUE
                    Low         Medium        High         Critical
              ┌─────────────┬─────────────┬─────────────┬─────────────┐
    Critical  │             │             │             │ Account     │
              │             │             │             │ Management  │
              │             │             │             │ (Wave 3)    │
              ├─────────────┼─────────────┼─────────────┼─────────────┤
    High      │             │             │ Credit Card │ Transaction │
              │             │             │ Management  │ Processing  │
              │             │             │ (Wave 4)    │ (Wave 5)    │
              │             │             │             │ Auth Proc.  │
              │             │             │             │ (Wave 6)    │
 COMPLEXITY   ├─────────────┼─────────────┼─────────────┼─────────────┤
    Medium    │ Data Mgmt   │ Reference   │ Customer    │             │
              │ & File Ops  │ Data Mgmt   │ Svc/Payments│             │
              │ (Wave N/A)  │ (Wave 2)    │ (Wave 5)    │             │
              │             │ Reporting   │             │             │
              │             │ (Wave 6)    │             │             │
              ├─────────────┼─────────────┼─────────────┼─────────────┤
    Low       │ System      │ Navigation  │ Shared      │ User Admin  │
              │ Admin/Infra │ & Interface │ Services    │ & Security  │
              │ (Wave 0)    │ (Wave 1)    │ (Wave 1)    │ (Wave 1)    │
              └─────────────┴─────────────┴─────────────┴─────────────┘
```

### 8.2 Detailed Domain Analysis

| Domain | Complexity Score | Value Score | Key Programs | Data Dependencies | Technology Mix | Recommendation |
|--------|-----------------|-------------|--------------|-------------------|----------------|----------------|
| **Account Management** | CRITICAL (9/10) | CRITICAL (10/10) | COACTUPC (CC=373), COACTVWC | ACCTDAT (R/W), CUSTDAT (R/W), CXACAIX | CICS, VSAM-KSDS, VSAM-PATH | Decompose carefully; highest risk |
| **Transaction Processing** | HIGH (8/10) | CRITICAL (10/10) | COTRN00C-02C, CBTRN01C-03C | TRANSACT (R/W), CCXREF, CXACAIX | CICS, VSAM-KSDS, GDG | Requires Account/Card first |
| **Authorization Processing** | HIGH (8/10) | CRITICAL (9/10) | COPAUS0C-2C, COPAUA0C | ACCTDAT, CUSTDAT, AUTHFRDS (DB2) | CICS, VSAM, DB2, IMS, MQ | Complex tech mix; performance-critical |
| **Credit Card Management** | HIGH (7/10) | HIGH (8/10) | COCRDLIC (CC=136), COCRDUPC (CC=170) | CARDDAT, CARDAIX, CXACAIX | CICS, VSAM-KSDS, VSAM-PATH | PCI considerations |
| **Customer Service/Payments** | MEDIUM (5/10) | HIGH (8/10) | COBIL00C | ACCTDAT (R/W), TRANSACT (W) | CICS, VSAM-KSDS | Well-bounded; customer-facing |
| **Reference Data Mgmt** | MEDIUM (5/10) | MEDIUM (5/10) | COTRTLIC (CC=221), COTRTUPC (CC=208) | TRANSACTION_TYPE (DB2) | CICS, DB2 | DB2 migration straightforward |
| **Reporting & Analytics** | MEDIUM (5/10) | MEDIUM (6/10) | CORPT00C, CBSTM03A/B | All master files (R) | CICS, VSAM, Batch | Read-only; batch replacement |
| **User Admin & Security** | LOW (3/10) | CRITICAL (9/10) | COSGN00C, COUSR01C-03C | USRSEC | CICS, VSAM-KSDS/ESDS/RRDS | Gateway; modernize first |
| **Shared Services** | LOW (2/10) | HIGH (7/10) | CSUTLDTC, COBSWAIT | None | Utilities | Foundation; enable reuse |
| **Navigation/Interface** | LOW (3/10) | MEDIUM (5/10) | COMEN01C, COADM01C | None (routing only) | CICS, BMS | Replace with SPA |
| **Data Mgmt & File Ops** | MEDIUM (4/10) | LOW (3/10) | CBEXPORT, CBIMPORT | All master files | Batch, VSAM | Operational; deferred |
| **System Admin/Infra** | LOW (2/10) | LOW (2/10) | JCL, PROCs, markers | Operational | JCL, System | CI/CD replacement |

### 8.3 Dependency Analysis Insights

#### Data Coupling Analysis (from dependencies_20260121105446.json)

| Data Source | Programs with Read | Programs with Write | Coupling Level |
|-------------|-------------------|---------------------|----------------|
| **ACCTDAT** (Account Master) | COACTVWC, COBIL00C, COPAUS0C, CBSTM03B, CBEXPORT, CBTRN01C | COACTUPC, COBIL00C, CBACT04C, CBTRN02C | **CRITICAL** |
| **CUSTDAT** (Customer Master) | COACTVWC, COPAUS0C, CBSTM03B, CBEXPORT | COACTUPC | **CRITICAL** |
| **TRANSACT** (Transactions) | COTRN00C, COTRN01C, CBSTM03B | COTRN02C, COBIL00C, CBTRN02C | **HIGH** |
| **CARDDAT** (Card Master) | COCRDLIC, COCRDSLC, CBSTM03B | COCRDUPC | **HIGH** |
| **CCXREF/CXACAIX** (Cross-Ref) | 12+ programs | None (read-only) | **HIGH** |
| **USRSEC** (User Security) | COSGN00C, COUSR01C-03C | COUSR01C-03C | **HIGH** |
| **AUTHFRDS** (Fraud - DB2) | None | COPAUS2C | **MEDIUM** |
| **TRANSACTION_TYPE** (DB2) | COTRTLIC | COTRTUPC, COBTUPDT | **LOW** |

#### Program Call Chain Analysis

```
Entry Points and Call Chains:

1. AUTHENTICATION FLOW (Gateway - Must modernize first)
   COSGN00C ──┬──> COMEN01C (regular users)
              └──> COADM01C (admin users)

   Dependencies: USRSEC file, all downstream programs

2. ACCOUNT/CUSTOMER FLOW (Highest complexity)
   COACTVWC ──> ACCTDAT, CUSTDAT (read-only view)
   COACTUPC ──> ACCTDAT (R/W), CUSTDAT (R/W), CXACAIX, COMEN01C

   Dependency chain: COSGN00C → COMEN01C → COACTVWC/COACTUPC

3. CREDIT CARD FLOW
   COCRDLIC ──> CARDDAT, COCRDSLC, COCRDUPC, COMEN01C
   COCRDSLC ──> CARDDAT, CARDAIX, COCRDLIC
   COCRDUPC ──> CARDDAT, CXACAIX, COCRDLIC, COMEN01C

   Dependencies: Account must exist for card operations

4. TRANSACTION FLOW
   COTRN00C ──> TRANSACT (browse), COTRN01C, COMEN01C, COSGN00C
   COTRN01C ──> TRANSACT (read), COTRN00C, COMEN01C, COSGN00C
   COTRN02C ──> TRANSACT (write), CCXREF, CXACAIX, CSUTLDTC

   Dependencies: Account + Card services required

5. AUTHORIZATION FLOW (Complex tech mix: CICS + IMS + DB2 + MQ)
   COPAUA0C ──> ACCTDAT, CUSTDAT, MQGET/MQPUT (async via MQ)
   COPAUS0C ──> ACCTDAT, CUSTDAT, CXACAIX, COPAUS1C, COSGN00C
   COPAUS1C ──> COPAUS0C, COPAUS2C
   COPAUS2C ──> AUTHFRDS (DB2 fraud table)

   Technology bridge: MQ to Amazon MQ; IMS to DynamoDB

6. BILL PAYMENT FLOW
   COBIL00C ──> ACCTDAT (R/W), TRANSACT (write), CXACAIX

   Dependencies: Account + Transaction services
```

### 8.4 Strategic Quadrant Analysis

Based on the complexity vs. value matrix, we derive four strategic quadrants:

#### Quadrant 1: HIGH VALUE, LOW COMPLEXITY (Quick Wins)
**Recommendation: Modernize Early**

| Domain | Rationale | Wave |
|--------|-----------|------|
| User Admin & Security | Gateway dependency, enables all subsequent work | 1 |
| Shared Services | Foundation utilities, high reuse | 1 |
| Navigation/Interface | UI replacement, React SPA | 1 |

**Dependencies resolved by completing Quadrant 1:**
- Authentication for all online programs
- Date/time utilities
- Application shell and routing

#### Quadrant 2: HIGH VALUE, HIGH COMPLEXITY (Strategic Investments)
**Recommendation: Invest Carefully with Risk Mitigation**

| Domain | Rationale | Risk Mitigation | Wave |
|--------|-----------|-----------------|------|
| Account Management | Highest complexity (CC=373), core entity | Decompose into Account + Customer services; extensive parallel testing | 3 |
| Transaction Processing | Highest volume, revenue | Event-driven architecture; Step Functions for batch | 5 |
| Authorization Processing | Performance-critical, complex tech mix | Cache layer; MQ bridge; gradual traffic shift | 6 |
| Credit Card Management | PCI compliance, moderate complexity | Security review; card tokenization | 4 |
| Customer Payments | Direct customer impact | Well-bounded; test extensively | 5 |

#### Quadrant 3: LOW VALUE, LOW COMPLEXITY (Fill-ins)
**Recommendation: Modernize Opportunistically**

| Domain | Rationale | Wave |
|--------|-----------|------|
| System Admin/Infrastructure | Replace JCL with CI/CD | 0 (Foundation) |

#### Quadrant 4: LOW VALUE, HIGH COMPLEXITY (Reconsider)
**Recommendation: Evaluate Rebuild vs. Maintain**

| Domain | Rationale | Decision |
|--------|-----------|----------|
| Data Mgmt & File Ops | Operational utilities, not customer-facing | Defer; replace with cloud-native tools |

### 8.5 Modernization Value Scores

| Domain | Complexity Penalty | Business Value | Net Score | Priority Rank |
|--------|-------------------|----------------|-----------|---------------|
| User Admin & Security | -3 | 90 | **87** | 1 |
| Shared Services | -2 | 70 | **68** | 2 |
| Reference Data Mgmt | -5 | 50 | **45** | 3 |
| Account Management | -9 | 100 | **91** (but HIGH RISK) | 4 |
| Credit Card Management | -7 | 80 | **73** | 5 |
| Transaction Processing | -8 | 100 | **92** (but HIGH RISK) | 6 |
| Customer Payments | -5 | 80 | **75** | 7 |
| Authorization Processing | -8 | 90 | **82** (but HIGH RISK) | 8 |
| Reporting & Analytics | -5 | 60 | **55** | 9 |
| Navigation/Interface | -3 | 50 | **47** | 10 |

### 8.6 Key Insights from Dependency Data

1. **COACTUPC is the Achilles' Heel**: With CC=373 and writes to both ACCTDAT and CUSTDAT, this single program represents the highest modernization risk. Recommend:
   - Split into separate Account Service and Customer Service
   - Implement saga pattern for cross-entity updates
   - Parallel running for minimum 3 cycles

2. **COSGN00C is the Gateway**: All online paths flow through sign-on. Modernizing authentication first with Amazon Cognito:
   - Unlocks parallel development for all other domains
   - Enables feature flags for gradual rollout
   - Reduces blast radius of subsequent changes

3. **Cross-Reference Tables (CXACAIX, CCXREF) are Read-Only**: These can be migrated early and accessed via APIs without write-contention concerns.

4. **Authorization has Unique Tech Mix**: COPAUS* programs use CICS + IMS + DB2 + MQ. Requires:
   - Amazon MQ for message bridging
   - DynamoDB for IMS data (high throughput)
   - Aurora PostgreSQL for fraud data

5. **Batch Programs Have Lower Coupling**: CBACT*, CBTRN*, CBSTM* programs have well-defined inputs/outputs, making them ideal for Step Functions migration.

---

## 9. Updated Recommendations

### Immediate Actions

1. ~~**Validate dependencies file**~~: ✓ Completed - `dependencies_20260121105446.json` analysis integrated
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
*Last updated: 2026-01-21 (Added Complexity vs. Value Matrix based on dependencies_20260121105446.json analysis)*
*Based on: Domain decomposition analysis, program_to_dsn.json, entrypoint JSON files, dependencies_20260121105446.json, AWS re:Invent 2025 best practices*
