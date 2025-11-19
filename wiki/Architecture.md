# CardDemo Architecture

This page provides a comprehensive overview of the CardDemo application architecture, including system design, component interactions, data flow, and technical patterns.

## Table of Contents
- [System Overview](#system-overview)
- [Application Layers](#application-layers)
- [Component Architecture](#component-architecture)
- [Data Architecture](#data-architecture)
- [Processing Flows](#processing-flows)
- [Security Architecture](#security-architecture)
- [Optional Module Integration](#optional-module-integration)

## System Overview

CardDemo is a multi-tier mainframe application built using traditional mainframe technologies. The architecture follows classic mainframe patterns with clear separation between online transaction processing (OLTP) and batch processing.

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    User Interface Layer                      │
│                    (CICS + BMS Screens)                      │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                  Application Logic Layer                     │
│                    (COBOL Programs)                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Online     │  │    Batch     │  │   Utility    │     │
│  │  Programs    │  │   Programs   │  │   Programs   │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Data Access Layer                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │   VSAM   │  │   DB2    │  │  IMS DB  │  │    MQ    │   │
│  │  (Base)  │  │(Optional)│  │(Optional)│  │(Optional)│   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### Key Architectural Principles

1. **Separation of Concerns**: Clear separation between presentation (BMS), business logic (COBOL), and data access (VSAM/DB2/IMS)
2. **Transaction Integrity**: ACID properties maintained through CICS transaction management
3. **Batch/Online Separation**: Distinct processing modes with file locking mechanisms
4. **Modular Design**: Core application with optional pluggable modules
5. **Data Independence**: Copybook-based data definitions for consistency

## Application Layers

### Presentation Layer (BMS Maps)

The presentation layer uses Basic Mapping Support (BMS) to define screen layouts and handle terminal I/O.

**Key Components:**
- **BMS Maps**: Screen definitions in assembler format
- **Symbolic Maps**: COBOL copybooks generated from BMS maps
- **Screen Flow**: Navigation between screens via PF keys and menu selections

**BMS Map Categories:**
- Sign-on and menu screens (COSGN00, COMEN01)
- Account management screens (COACTVW, COACTUP)
- Card management screens (COCRDLI, COCRDSL, COCRDUP)
- Transaction screens (COTRN00, COTRN01, COTRN02)
- Report screens (CORPT00)
- Bill payment screens (COBIL00)
- Admin screens (COADM01, COUSR00-03)

### Application Logic Layer (COBOL Programs)

The business logic layer implements all application functionality using COBOL programs.

**Program Categories:**

1. **Online Programs (CICS)**
   - Conversational programs for user interaction
   - Pseudo-conversational design for efficiency
   - COMMAREA for data passing between screens
   - CICS API calls for terminal I/O, file access, and program control

2. **Batch Programs (JCL)**
   - Sequential file processing
   - VSAM file maintenance
   - Report generation
   - Data transformation and loading

3. **Utility Programs (Assembler)**
   - MVSWAIT: Timer control for batch jobs
   - COBDATFT: Date format conversion utility

**Programming Patterns:**
- Structured COBOL with proper paragraph organization
- Error handling with CICS HANDLE CONDITION
- Copybook inclusion for data structures
- Standard naming conventions (CO* for online, CB* for batch)

### Data Access Layer

The data layer provides persistent storage using multiple technologies.

**Primary Storage (VSAM):**
- KSDS (Key-Sequenced Data Sets) for indexed access
- AIX (Alternate Index) for secondary key access
- ESDS/RRDS for specialized use cases

**Optional Storage:**
- DB2 for relational data (transaction types, fraud tracking)
- IMS DB for hierarchical data (authorization details)
- MQ for asynchronous messaging

## Component Architecture

### Core Application Components

#### User Management
- **Programs**: COSGN00C (sign-on), COUSR00C-03C (user CRUD)
- **Data**: USRSEC VSAM file
- **Security**: RACF integration for authentication

#### Account Management
- **Programs**: COACTVWC (view), COACTUPC (update)
- **Data**: ACCTDAT VSAM file
- **Features**: Account inquiry, balance viewing, credit limit management

#### Card Management
- **Programs**: COCRDLIC (list), COCRDSLC (view), COCRDUPC (update)
- **Data**: CARDDAT VSAM file, CARDXREF for cross-reference
- **Features**: Card issuance, status management, expiration tracking

#### Transaction Processing
- **Online Programs**: COTRN00C (list), COTRN01C (view), COTRN02C (add)
- **Batch Programs**: CBTRN02C (post transactions), CBTRN03C (reports)
- **Data**: TRANSACT VSAM file with AIX
- **Features**: Transaction entry, posting, reporting

#### Bill Payment
- **Programs**: COBIL00C
- **Data**: TRANSACT VSAM file
- **Features**: Payment processing, balance updates

#### Reporting
- **Programs**: CORPT00C (online), CBSTM03A (batch statements)
- **Data**: TRANSACT VSAM file
- **Features**: Transaction reports, account statements

### Optional Module Components

#### Credit Card Authorizations (IMS-DB2-MQ)
- **Programs**: COPAUA0C (MQ processor), COPAUS0C/1C (inquiry), CBPAUP0C (purge)
- **Data**: IMS HIDAM database, DB2 fraud table, MQ queues
- **Integration**: Real-time authorization via MQ, fraud detection with DB2

#### Transaction Type Management (DB2)
- **Programs**: COTRTUPC (add/edit), COTRTLIC (list/delete), COBTUPDT (batch)
- **Data**: DB2 transaction type tables
- **Integration**: DB2 embedded SQL with cursor processing

#### Account Extractions (VSAM-MQ)
- **Programs**: CODATE01 (date inquiry), COACCT01 (account inquiry)
- **Data**: VSAM files, MQ queues
- **Integration**: MQ request/response patterns

## Data Architecture

### VSAM File Organization

CardDemo uses VSAM as its primary data store with the following organization:

```
ACCTDAT (Account Master)
  Key: Account ID (11 digits)
  Contains: Account details, balances, credit limits

CARDDAT (Card Master)
  Key: Card Number (16 digits)
  Contains: Card details, status, expiration

CUSTDAT (Customer Master)
  Key: Customer ID (9 digits)
  Contains: Customer personal information

CARDXREF (Cross Reference)
  Key: Composite key
  Contains: Account-Card-Customer relationships

TRANSACT (Transaction File)
  Key: Transaction ID
  AIX: Card Number, Date
  Contains: Transaction details, amounts, merchants

USRSEC (User Security)
  Key: User ID
  Contains: User credentials, roles, permissions

TRANTYPE (Transaction Types)
  Key: Transaction Type Code
  Contains: Transaction type descriptions

TRANCATG (Transaction Categories)
  Key: Category Code
  Contains: Category descriptions
```

### Data Relationships

```
CUSTOMER (1) ──── (N) ACCOUNT (1) ──── (N) CARD
                                │
                                └──── (N) TRANSACTION
```

### Copybook Structure

All data structures are defined in copybooks located in `app/cpy/`:

- **CSUSR01Y**: User security record
- **CVACT01Y**: Account data record
- **CVACT02Y**: Card data record
- **CVCUS01Y**: Customer data record
- **CVACT03Y**: Card cross-reference record
- **CVTRA05Y**: Transaction record (online)
- **CVTRA06Y**: Transaction record (batch)

## Processing Flows

### Online Transaction Flow

```
1. User enters transaction (CC00 - Sign-on)
2. CICS receives terminal input
3. BMS map receives and formats data
4. COBOL program processes request
5. VSAM files accessed for data
6. Business logic applied
7. Response formatted via BMS
8. Screen displayed to user
```

### Batch Processing Flow

```
1. JCL job submitted
2. CLOSEFIL closes CICS files
3. Batch programs execute:
   - Read input files
   - Process records
   - Update VSAM files
   - Generate reports
4. OPENFIL reopens files for CICS
5. Job completion notification
```

### Authorization Flow (Optional Module)

```
1. POS system sends MQ message
2. MQ trigger starts CICS transaction (CP00)
3. COPAUA0C program processes request:
   - Validates card via VSAM
   - Retrieves customer data
   - Applies business rules
   - Stores in IMS DB
4. Response sent via MQ
5. User can view via CPVS/CPVD
6. Fraud marking updates DB2
```

## Security Architecture

### Authentication
- User credentials stored in USRSEC VSAM file
- Password validation at sign-on (COSGN00C)
- Session management via CICS

### Authorization
- Role-based access control (User vs Admin)
- Transaction-level security via CICS
- RACF integration for system-level security

### Data Security
- Encrypted password storage
- Audit trail in transaction records
- File-level access control via VSAM

## Optional Module Integration

### Integration Points

1. **Admin Menu Extension**: Optional modules add menu items to CA00
2. **Data Synchronization**: TRANEXTR job extracts DB2 data to VSAM
3. **Message Queue Integration**: MQ queues for asynchronous processing
4. **Database Integration**: DB2 and IMS DB for specialized storage

### Module Independence

Each optional module is designed to be independently installable:
- Core application functions without optional modules
- Optional modules enhance but don't replace core functionality
- Shared copybooks ensure data compatibility
- Separate CICS resource groups for easy management

## Performance Considerations

### Online Performance
- Pseudo-conversational programming for CICS efficiency
- Indexed VSAM access for fast lookups
- Minimal COMMAREA size for reduced overhead
- Efficient screen navigation

### Batch Performance
- Sequential file processing for throughput
- Batch window management with CLOSEFIL/OPENFIL
- Sort utilities for data organization
- GDG for backup and recovery

## Scalability

CardDemo demonstrates scalability patterns:
- CICS transaction routing for workload distribution
- VSAM file sharing across regions
- Batch job parallelization opportunities
- Optional module architecture for feature growth

---

**Navigation**: [Home](Home.md) | [Installation Guide](Installation-Guide.md) | [Technical Components](Technical-Components.md) | [Data Models](Data-Models.md)
