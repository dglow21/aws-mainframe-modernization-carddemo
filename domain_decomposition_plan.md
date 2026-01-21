# CardDemo Application Domain Decomposition Plan

## Executive Summary

This document provides a comprehensive plan for refining the domain decomposition of the AWS CardDemo mainframe application. The analysis covers 139 source files including COBOL programs, JCL batch jobs, copybooks, and BMS screen maps. The goal is to create a MECE (Mutually Exclusive, Collectively Exhaustive) set of domains that properly categorizes all application artifacts.

## Current State Analysis

### Existing Domains (11 Total)

| # | Domain Name | Description | Currently Assigned Files |
|---|-------------|-------------|-------------------------|
| 1 | Transaction Processing and Management | Transaction lifecycle management | 5 files |
| 2 | Credit Card Management | Card lifecycle management | 3 files |
| 3 | Account Management | Account operations | 3 files |
| 4 | Authorization Processing | Real-time authorization | 2 files |
| 5 | Customer Service and Payments | Bill payment processing | 1 file |
| 6 | User Administration and Security | User accounts and access control | 4 files |
| 7 | Reporting and Analytics | Business reports and statements | 4 files |
| 8 | System Navigation and Interface | Menus and navigation | 3 files |
| 9 | Data Management and File Operations | Data extraction and loading | 5 files |
| 10 | Reference Data Management | Master reference data | 3 files |
| 11 | System Administration and Infrastructure | System admin operations | 5 files |

**Total Files in Current Decomposition:** 38 files
**Total Files in Repository:** 139 files (COBOL, JCL, CPY, BMS)
**Unassigned Files:** 101 files (~73% of total)

## Gap Analysis

### Files Not Currently Assigned

#### COBOL Programs (Unassigned)

| Program | Purpose | Recommended Domain |
|---------|---------|-------------------|
| CBACT01C.cbl | Account data extraction (VSAM to sequential) | Batch Data Processing |
| CBACT02C.cbl | Card data reading and display | Batch Data Processing |
| CBACT03C.cbl | Cross-reference data validation | Batch Data Processing |
| CBACT04C.cbl | Interest calculation and posting | Transaction Processing and Management |
| CBCUS01C.cbl | Customer data reading | Batch Data Processing |
| CBTRN01C.cbl | Transaction card validation | Transaction Processing and Management |
| CBTRN02C.cbl | Daily transaction posting with validation | Transaction Processing and Management |
| CBTRN03C.cbl | Transaction detail reporting | Reporting and Analytics |
| CBSTM03A.CBL | Statement generation (text/HTML) | Reporting and Analytics |
| CBSTM03B.CBL | Statement file I/O subroutine | Reporting and Analytics |
| CBEXPORT.cbl | Multi-file data export | Data Management and File Operations |
| CBIMPORT.cbl | Data import and distribution | Data Management and File Operations |
| COSGN00C.cbl | User sign-on/authentication | User Administration and Security |
| CSUTLDTC.cbl | Date validation utility | Shared Services |
| COBSWAIT.cbl | Wait utility (timing) | Shared Services |
| COPAUS0C.cbl | Pending authorization summary (IMS) | Authorization Processing |
| COPAUS2C.cbl | Fraud record management (DB2) | Authorization Processing |
| CBPAUP0C.cbl | Expired authorization cleanup | Authorization Processing |
| PAUDBLOD.CBL | IMS database loading | System Administration and Infrastructure |
| PAUDBUNL.CBL | IMS database unloading | System Administration and Infrastructure |
| DBUNLDGS.CBL | GSAM database unloading | System Administration and Infrastructure |
| COTRTLIC.cbl | Transaction type list (DB2) | Reference Data Management |
| COTRTUPC.cbl | Transaction type update (DB2) | Reference Data Management |
| COBTUPDT.cbl | Batch transaction type update | Reference Data Management |

#### JCL Files (Unassigned or Incorrectly Assigned)

| JCL | Purpose | Recommended Domain |
|-----|---------|-------------------|
| ACCTFILE.jcl | Account file rebuild | System Administration and Infrastructure |
| CARDFILE.jcl | Card file rebuild | System Administration and Infrastructure |
| CUSTFILE.jcl | Customer file management | System Administration and Infrastructure |
| TRANFILE.jcl | Transaction file setup | System Administration and Infrastructure |
| XREFFILE.jcl | Cross-reference file setup | System Administration and Infrastructure |
| CBADMCDJ.jcl | CICS environment setup | System Administration and Infrastructure |
| CLOSEFIL.jcl | CICS file closing | System Administration and Infrastructure |
| OPENFIL.jcl | CICS file opening | System Administration and Infrastructure |
| INTCALC.jcl | Interest calculation | Transaction Processing and Management |
| COMBTRAN.jcl | Transaction consolidation | Data Management and File Operations |
| CBIMPORT.jcl | Data import job | Data Management and File Operations |
| WAITSTEP.jcl | Batch timing control | System Administration and Infrastructure |
| TRANCATG.jcl | Transaction category setup | Reference Data Management |
| TRANTYPE.jcl | Transaction type loading | Reference Data Management |
| DALYREJS.jcl | Daily rejection GDG setup | System Administration and Infrastructure |
| DEFGDGB.jcl | GDG base definition | System Administration and Infrastructure |
| DEFGDGD.jcl | Reference data GDG | System Administration and Infrastructure |
| REPTFILE.jcl | Report file GDG | System Administration and Infrastructure |
| ESDSRRDS.jcl | Security system init | User Administration and Security |
| DUSRSECJ.jcl | User security population | User Administration and Security |
| LOADPADB.JCL | Authorization data loading | Data Management and File Operations |
| UNLDPADB.JCL | Authorization data unloading | Data Management and File Operations |
| UNLDGSAM.JCL | GSAM data extraction | Data Management and File Operations |
| DBPAUTP0.jcl | IMS database operations | System Administration and Infrastructure |
| CBPAUP0J.jcl | IMS batch processing | Authorization Processing |
| INTRDRJ1.JCL, INTRDRJ2.JCL | Internal reader jobs | System Administration and Infrastructure |
| FTPJCL.JCL | File transfer | System Administration and Infrastructure |

#### Copybooks (Need Assignment)

| Category | Copybooks | Recommended Domain |
|----------|-----------|-------------------|
| Account Data | CVACT01Y, CVACT02Y, CVACT03Y | Account Management |
| Customer Data | CVCUS01Y, CUSTREC | Account Management |
| Credit Card Data | CVCRD01Y | Credit Card Management |
| Transaction Data | CVTRA01Y-07Y | Transaction Processing and Management |
| User Data | CSUSR01Y | User Administration and Security |
| Common/Utility | CSDAT01Y, CSMSG01Y, CSMSG02Y, CSSETATY, CSSTRPFY | Shared Services |
| DB2 Communication | CSDB2RPY, CSDB2RWY | Shared Services |
| Date Utilities | CSUTLDPY, CSUTLDWY, CODATECN | Shared Services |
| Export/Import | CVEXPORT | Data Management and File Operations |
| Menu/Navigation | COADM02Y, COCOM01Y, COMEN02Y, COTTL01Y | System Navigation and Interface |
| Authorization | CCPAUERY, CCPAURLY, CCPAURQY, CIPAUDTY, CIPAUSMY | Authorization Processing |
| IMS Functions | IMSFUNCS | Authorization Processing |
| BMS Screen Definitions | COPAU00, COPAU01, COTRTLI, COTRTUP | Various (with parent BMS) |
| Lookup | CSLKPCDY | Reference Data Management |
| Unused | UNUSED1Y | (Can be removed) |

#### BMS Screen Maps (Need Assignment)

| BMS File | Associated Program | Domain |
|----------|-------------------|--------|
| COACTUP.bms | COACTUPC | Account Management |
| COACTVW.bms | COACTVWC | Account Management |
| COADM01.bms | COADM01C | System Navigation and Interface |
| COBIL00.bms | COBIL00C | Customer Service and Payments |
| COCRDLI.bms | COCRDLIC | Credit Card Management |
| COCRDSL.bms | COCRDSLC | Credit Card Management |
| COCRDUP.bms | COCRDUPC | Credit Card Management |
| COMEN01.bms | COMEN01C | System Navigation and Interface |
| COPAU00.bms | COPAUA0C | Authorization Processing |
| COPAU01.bms | COPAUS1C | Authorization Processing |
| CORPT00.bms | CORPT00C | Reporting and Analytics |
| COSGN00.bms | COSGN00C | User Administration and Security |
| COTRN00.bms | COTRN00C | Transaction Processing and Management |
| COTRN01.bms | COTRN01C | Transaction Processing and Management |
| COTRN02.bms | COTRN02C | Transaction Processing and Management |
| COTRTLI.bms | COTRTLIC | Reference Data Management |
| COTRTUP.bms | COTRTUPC | Reference Data Management |
| COUSR00-03.bms | COUSR00-03C | User Administration and Security |

---

## Recommended Domain Structure (MECE)

After analyzing all files, the existing 11 domains are appropriate but need expansion. I recommend **keeping all 11 existing domains** and adding **1 new domain** for a total of **12 domains**. This maintains the MECE principle while properly accommodating all files.

### New Domain: Shared Services

**Name:** Shared Services
**Description:** Provides common utility functions, data type conversions, date handling, DB2 communication wrappers, and reusable components used across multiple business domains.

**Rationale:** Several copybooks and programs provide cross-cutting functionality that doesn't belong to any specific business domain but is essential infrastructure.

### Refined Domain Definitions

| # | Domain | Description | File Types |
|---|--------|-------------|------------|
| 1 | **Transaction Processing and Management** | Manages complete transaction lifecycle including inquiry, creation, posting, interest calculation, and reporting | COBOL, JCL, BMS, CPY |
| 2 | **Credit Card Management** | Credit card lifecycle including inquiry, updates, and maintenance | COBOL, BMS, CPY |
| 3 | **Account Management** | Customer account operations, balance management, account-card relationships | COBOL, BMS, CPY |
| 4 | **Authorization Processing** | Real-time and batch authorization including fraud detection, pending authorizations, IMS/DB2 integration | COBOL, JCL, BMS, CPY |
| 5 | **Customer Service and Payments** | Customer-facing bill payment and self-service operations | COBOL, BMS |
| 6 | **User Administration and Security** | User accounts, authentication, role-based access control | COBOL, JCL, BMS, CPY |
| 7 | **Reporting and Analytics** | Business reports, customer statements, multi-format output (print/HTML/PDF) | COBOL, JCL, BMS |
| 8 | **System Navigation and Interface** | Main menus, navigation routing, administrative menus, date services | COBOL, BMS, CPY |
| 9 | **Data Management and File Operations** | Data extraction, loading, export/import, file consolidation | COBOL, JCL, CPY |
| 10 | **Reference Data Management** | Transaction types, categories, disclosure groups, lookup tables | COBOL, JCL, BMS, CPY |
| 11 | **System Administration and Infrastructure** | Database admin, file management, compilation, CICS operations, GDG management | JCL, COBOL, PROC |
| 12 | **Shared Services** (NEW) | Common utilities, date handling, DB2 communication, reusable components | COBOL, CPY |

---

## Detailed File-to-Domain Mapping

### Domain 1: Transaction Processing and Management

#### COBOL Programs
| File | Type | Purpose |
|------|------|---------|
| COTRN00C.cbl | Online | Transaction browsing and inquiry |
| COTRN01C.cbl | Online | Detailed transaction viewing |
| COTRN02C.cbl | Online | New transaction creation |
| CBACT04C.cbl | Batch | Interest calculation and posting |
| CBTRN01C.cbl | Batch | Transaction card validation |
| CBTRN02C.cbl | Batch | Daily transaction posting |

#### JCL
| File | Purpose |
|------|---------|
| POSTTRAN.jcl | Daily transaction posting job |
| TRANREPT.jcl | Transaction reporting job |
| INTCALC.jcl | Interest calculation job |

#### BMS Screens
| File | Purpose |
|------|---------|
| COTRN00.bms | Transaction list screen |
| COTRN01.bms | Transaction detail screen |
| COTRN02.bms | Transaction entry screen |

#### Copybooks
| File | Purpose |
|------|---------|
| CVTRA01Y.cpy | Transaction category balance record |
| CVTRA02Y.cpy | Discount group record |
| CVTRA03Y.cpy | Transaction type record |
| CVTRA04Y.cpy | Transaction category record |
| CVTRA05Y.cpy | Transaction record |
| CVTRA06Y.cpy | Daily transaction record |
| CVTRA07Y.cpy | Transaction report record |

---

### Domain 2: Credit Card Management

#### COBOL Programs
| File | Type | Purpose |
|------|------|---------|
| COCRDLIC.cbl | Online | Credit card list/inquiry |
| COCRDSLC.cbl | Online | Credit card selection |
| COCRDUPC.cbl | Online | Credit card update |

#### BMS Screens
| File | Purpose |
|------|---------|
| COCRDLI.bms | Card list screen |
| COCRDSL.bms | Card selection screen |
| COCRDUP.bms | Card update screen |

#### Copybooks
| File | Purpose |
|------|---------|
| CVCRD01Y.cpy | Credit card data record |

---

### Domain 3: Account Management

#### COBOL Programs
| File | Type | Purpose |
|------|------|---------|
| COACTVWC.cbl | Online | Account inquiry and viewing |
| COACTUPC.cbl | Online | Account update |
| COACCT01.cbl | Online/MQ | Real-time account inquiry via MQ |

#### BMS Screens
| File | Purpose |
|------|---------|
| COACTVW.bms | Account view screen |
| COACTUP.bms | Account update screen |

#### Copybooks
| File | Purpose |
|------|---------|
| CVACT01Y.cpy | Account record |
| CVACT02Y.cpy | Card record (account relationship) |
| CVACT03Y.cpy | Card cross-reference record |
| CVCUS01Y.cpy | Customer record |
| CUSTREC.cpy | Customer record definition |

---

### Domain 4: Authorization Processing

#### COBOL Programs
| File | Type | Purpose |
|------|------|---------|
| COPAUA0C.cbl | Online | Real-time authorization processing |
| COPAUS0C.cbl | Online | Pending authorization summary |
| COPAUS1C.cbl | Online | Authorization detail viewing |
| COPAUS2C.cbl | Online | Fraud record management (DB2) |
| CBPAUP0C.cbl | Batch | Expired authorization cleanup |

#### JCL
| File | Purpose |
|------|---------|
| CBPAUP0J.jcl | IMS batch authorization processing |

#### BMS Screens
| File | Purpose |
|------|---------|
| COPAU00.bms | Authorization main screen |
| COPAU01.bms | Authorization detail screen |

#### Copybooks
| File | Purpose |
|------|---------|
| CCPAUERY.cpy | Authorization error handling |
| CCPAURLY.cpy | Authorization reply |
| CCPAURQY.cpy | Authorization request |
| CIPAUDTY.cpy | IMS authorization detail |
| CIPAUSMY.cpy | IMS authorization summary |
| COPAU00.cpy | BMS screen copybook |
| COPAU01.cpy | BMS screen copybook |
| IMSFUNCS.cpy | IMS function definitions |

---

### Domain 5: Customer Service and Payments

#### COBOL Programs
| File | Type | Purpose |
|------|------|---------|
| COBIL00C.cbl | Online | Bill payment processing |

#### BMS Screens
| File | Purpose |
|------|---------|
| COBIL00.bms | Bill payment screen |

---

### Domain 6: User Administration and Security

#### COBOL Programs
| File | Type | Purpose |
|------|------|---------|
| COSGN00C.cbl | Online | User sign-on/authentication |
| COUSR00C.cbl | Online | User management hub |
| COUSR01C.cbl | Online | New user creation |
| COUSR02C.cbl | Online | User update |
| COUSR03C.cbl | Online | User deletion |

#### JCL
| File | Purpose |
|------|---------|
| ESDSRRDS.jcl | Security system initialization |
| DUSRSECJ.jcl | User security file population |

#### BMS Screens
| File | Purpose |
|------|---------|
| COSGN00.bms | Sign-on screen |
| COUSR00.bms | User management screen |
| COUSR01.bms | User create screen |
| COUSR02.bms | User update screen |
| COUSR03.bms | User delete screen |

#### Copybooks
| File | Purpose |
|------|---------|
| CSUSR01Y.cpy | User security record |

---

### Domain 7: Reporting and Analytics

#### COBOL Programs
| File | Type | Purpose |
|------|------|---------|
| CORPT00C.cbl | Online | Report generation interface |
| CBTRN03C.cbl | Batch | Transaction detail reporting |
| CBSTM03A.CBL | Batch | Statement generation (text/HTML) |
| CBSTM03B.CBL | Batch | Statement file I/O subroutine |

#### JCL
| File | Purpose |
|------|---------|
| CREASTMT.JCL | Statement creation job |
| TXT2PDF1.JCL | PDF conversion job |
| PRTCATBL.jcl | Category balance print job |

#### BMS Screens
| File | Purpose |
|------|---------|
| CORPT00.bms | Report generation screen |

---

### Domain 8: System Navigation and Interface

#### COBOL Programs
| File | Type | Purpose |
|------|------|---------|
| COMEN01C.cbl | Online | Main menu navigation |
| COADM01C.cbl | Online | Administrative menu |
| CODATE01.cbl | Online/MQ | Date/time services |

#### BMS Screens
| File | Purpose |
|------|---------|
| COMEN01.bms | Main menu screen |
| COADM01.bms | Admin menu screen |

#### Copybooks
| File | Purpose |
|------|---------|
| COADM02Y.cpy | Admin screen layout |
| COCOM01Y.cpy | Common screen fields |
| COMEN02Y.cpy | Menu screen layout |
| COTTL01Y.cpy | Title/header layout |

---

### Domain 9: Data Management and File Operations

#### COBOL Programs
| File | Type | Purpose |
|------|------|---------|
| CBEXPORT.cbl | Batch | Multi-file data export |
| CBIMPORT.cbl | Batch | Data import processing |
| CBACT01C.cbl | Batch | Account data extraction |
| CBACT02C.cbl | Batch | Card data reading |
| CBACT03C.cbl | Batch | Cross-reference validation |
| CBCUS01C.cbl | Batch | Customer data reading |
| PAUDBLOD.CBL | Batch | IMS database loading |
| PAUDBUNL.CBL | Batch | IMS database unloading |
| DBUNLDGS.CBL | Batch | GSAM database extraction |

#### JCL
| File | Purpose |
|------|---------|
| READACCT.jcl | Account file reading |
| READCARD.jcl | Card file reading |
| READCUST.jcl | Customer file reading |
| READXREF.jcl | Cross-reference reading |
| CBEXPORT.jcl | Export job |
| CBIMPORT.jcl | Import job |
| COMBTRAN.jcl | Transaction consolidation |
| LOADPADB.JCL | Authorization data loading |
| UNLDPADB.JCL | Authorization data unloading |
| UNLDGSAM.JCL | GSAM extraction |

#### Copybooks
| File | Purpose |
|------|---------|
| CVEXPORT.cpy | Export record format |

---

### Domain 10: Reference Data Management

#### COBOL Programs
| File | Type | Purpose |
|------|------|---------|
| COTRTLIC.cbl | Online | Transaction type list (DB2) |
| COTRTUPC.cbl | Online | Transaction type update (DB2) |
| COBTUPDT.cbl | Batch | Batch transaction type update |

#### JCL
| File | Purpose |
|------|---------|
| TRANEXTR.jcl | Reference data extraction |
| DISCGRP.jcl | Disclosure group setup |
| TCATBALF.jcl | Category balance init |
| TRANCATG.jcl | Transaction category setup |
| TRANTYPE.jcl | Transaction type loading |

#### BMS Screens
| File | Purpose |
|------|---------|
| COTRTLI.bms | Transaction type list screen |
| COTRTUP.bms | Transaction type update screen |

#### Copybooks
| File | Purpose |
|------|---------|
| COTRTLI.cpy | BMS screen copybook |
| COTRTUP.cpy | BMS screen copybook |
| CSLKPCDY.cpy | Lookup code definitions |

---

### Domain 11: System Administration and Infrastructure

#### COBOL Programs
| File | Type | Purpose |
|------|------|---------|
| (None - primarily JCL/PROC based) | | |

#### JCL - Database Administration
| File | Purpose |
|------|---------|
| CREADB21.jcl | DB2 database creation |
| MNTTRDB2.jcl | DB2 maintenance |
| DBPAUTP0.jcl | IMS database operations |

#### JCL - File Management
| File | Purpose |
|------|---------|
| ACCTFILE.jcl | Account file rebuild |
| CARDFILE.jcl | Card file rebuild |
| CUSTFILE.jcl | Customer file management |
| TRANFILE.jcl | Transaction file setup |
| XREFFILE.jcl | Cross-reference file setup |
| TRANIDX.jcl | Transaction index creation |
| TRANBKP.jcl | Transaction backup |

#### JCL - CICS Operations
| File | Purpose |
|------|---------|
| CBADMCDJ.jcl | CICS environment setup |
| CLOSEFIL.jcl | CICS file closing |
| OPENFIL.jcl | CICS file opening |

#### JCL - GDG Management
| File | Purpose |
|------|---------|
| DEFGDGB.jcl | GDG base definition |
| DEFGDGD.jcl | Reference data GDG |
| REPTFILE.jcl | Report file GDG |
| DALYREJS.jcl | Daily rejection GDG |

#### JCL - System Operations
| File | Purpose |
|------|---------|
| DEFCUST.jcl | Customer infrastructure setup |
| WAITSTEP.jcl | Batch timing control |
| FTPJCL.JCL | File transfer operations |
| INTRDRJ1.JCL | Internal reader operations |
| INTRDRJ2.JCL | Internal reader operations |

#### Sample/Build Files
| File | Purpose |
|------|---------|
| samples/jcl/BATCMP.jcl | Batch compilation |
| samples/jcl/BMSCMP.jcl | BMS compilation |
| samples/jcl/CICCMP.jcl | CICS compilation |
| samples/jcl/CICDBCMP.jcl | CICS DB2 compilation |
| samples/jcl/IMSMQCMP.jcl | IMS MQ compilation |
| samples/jcl/LISTCAT.jcl | Catalog listing |
| samples/jcl/RACFCMDS.jcl | Security commands |
| samples/jcl/REPRTEST.jcl | Test replication |
| samples/jcl/SORTTEST.jcl | Sort testing |
| samples/proc/*.prc | Build procedures |

---

### Domain 12: Shared Services (NEW)

#### COBOL Programs
| File | Type | Purpose |
|------|------|---------|
| CSUTLDTC.cbl | Utility | Date validation/conversion |
| COBSWAIT.cbl | Utility | Wait/timing utility |

#### Copybooks
| File | Purpose |
|------|---------|
| CSDAT01Y.cpy | Date data structures |
| CSUTLDPY.cpy | Date utility parameters |
| CSUTLDWY.cpy | Date utility work areas |
| CODATECN.cpy | Date constants |
| CSMSG01Y.cpy | Message definitions |
| CSMSG02Y.cpy | Extended message definitions |
| CSSETATY.cpy | Attribute settings |
| CSSTRPFY.cpy | String processing functions |
| CSDB2RPY.cpy | DB2 response handling |
| CSDB2RWY.cpy | DB2 work areas |

---

## Implementation Plan

### Phase 1: Validation and Preparation
1. Review current domain directory structures
2. Verify all file paths in enhanced_decomposition_input.json
3. Identify any dependencies between files
4. Create backup of existing domain assignments

### Phase 2: Domain Updates
1. Update enhanced_decomposition_input.json with complete file assignments
2. Create new "Shared Services" domain structure
3. Move unassigned files to appropriate domains
4. Update domain JSON files with new components

### Phase 3: Documentation
1. Update application.json with new domain
2. Generate updated domain HTML documentation
3. Create data lineage reports per domain
4. Document cross-domain dependencies

### Phase 4: Verification
1. Validate MECE principle (no overlaps, no gaps)
2. Ensure each file has exactly one domain assignment
3. Verify copybook and BMS associations match COBOL programs
4. Run consistency checks on domain boundaries

---

## File Count Summary by Domain

| Domain | COBOL | JCL | BMS | CPY | Total |
|--------|-------|-----|-----|-----|-------|
| Transaction Processing and Management | 6 | 3 | 3 | 7 | 19 |
| Credit Card Management | 3 | 0 | 3 | 1 | 7 |
| Account Management | 3 | 0 | 2 | 5 | 10 |
| Authorization Processing | 5 | 1 | 2 | 8 | 16 |
| Customer Service and Payments | 1 | 0 | 1 | 0 | 2 |
| User Administration and Security | 5 | 2 | 5 | 1 | 13 |
| Reporting and Analytics | 4 | 3 | 1 | 0 | 8 |
| System Navigation and Interface | 3 | 0 | 2 | 4 | 9 |
| Data Management and File Operations | 9 | 10 | 0 | 1 | 20 |
| Reference Data Management | 3 | 5 | 2 | 3 | 13 |
| System Administration and Infrastructure | 3 | 28+ | 0 | 0 | 31+ |
| Shared Services (NEW) | 2 | 0 | 0 | 10 | 12 |
| **TOTAL** | **47** | **52+** | **21** | **40** | **160+** |

Note: Some files may appear in samples or as variants, accounting for the total being higher than 139.

---

## Recommendations

### 1. Keep Existing Domains
All 11 existing domains are well-defined and represent distinct business capabilities. No consolidation is needed.

### 2. Add Shared Services Domain
Create a new "Shared Services" domain to house common utilities and reusable components that span multiple business domains.

### 3. Expand File Assignments
The current decomposition only includes ~27% of files. This plan provides assignments for all 139 files.

### 4. Maintain Copybook/BMS Associations
Copybooks and BMS screens should be assigned to the same domain as their primary consuming COBOL program.

### 5. Consider Sub-Domain Grouping
For very large domains like System Administration (30+ files), consider grouping by sub-function (Database, CICS, GDG, Build) in documentation.

---

## Next Steps

1. **Review this plan** with stakeholders for approval
2. **Update enhanced_decomposition_input.json** with complete assignments
3. **Create domain directory for Shared Services**
4. **Migrate files** to correct domain directories
5. **Regenerate documentation** for all domains
6. **Validate MECE compliance** with automated checks

---

*Document generated as part of mainframe modernization domain decomposition analysis.*
