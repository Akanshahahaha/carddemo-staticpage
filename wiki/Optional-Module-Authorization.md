# Optional Module: Credit Card Authorizations (IMS-DB2-MQ)

Complete documentation for the Credit Card Authorizations optional module, which adds real-time authorization processing using IMS DB, DB2, and MQ.

## Table of Contents
- [Module Overview](#module-overview)
- [Features](#features)
- [Architecture](#architecture)
- [Installation](#installation)
- [Components](#components)
- [Data Models](#data-models)
- [User Guide](#user-guide)
- [Technical Details](#technical-details)
- [Troubleshooting](#troubleshooting)

## Module Overview

The Credit Card Authorizations module enhances CardDemo with comprehensive real-time authorization processing capabilities. This module simulates real-world credit card authorization flows from initial merchant request to approval/decline decisions, including fraud detection.

### Business Value

- **Real-time Processing**: Immediate authorization decisions via MQ messaging
- **Fraud Detection**: Identify and flag suspicious transactions
- **Authorization History**: Complete audit trail of all authorization requests
- **Multi-Database Integration**: Demonstrates IMS DB and DB2 integration patterns
- **Asynchronous Processing**: MQ-based request/response architecture

### Technologies Used

- **COBOL**: Business logic implementation
- **CICS**: Transaction processing and screen management
- **IMS DB**: Hierarchical database for authorization storage (HIDAM)
- **DB2**: Relational database for fraud analytics
- **MQ**: Message queuing for authorization requests/responses
- **VSAM**: Account and customer data access

## Features

### Authorization Processing

1. **Real-time Authorization Requests**
   - Receive authorization requests via MQ
   - Validate card and account status
   - Apply business rules for approval/decline
   - Send response via MQ
   - Store authorization details in IMS DB

2. **Authorization Inquiry**
   - View pending authorizations by account
   - Display authorization summary
   - View detailed authorization information
   - Navigate through multiple authorizations

3. **Fraud Detection**
   - Mark suspicious transactions as fraudulent
   - Store fraud cases in DB2 for analytics
   - Track fraud patterns and trends

4. **Batch Processing**
   - Daily purge of expired authorizations
   - Adjust available credit for unmatched authorizations
   - Generate authorization reports

## Architecture

### Authorization Flow

```
┌──────────────┐
│ POS System / │
│ Cloud Client │
└──────┬───────┘
       │ MQ Request
       ▼
┌──────────────────┐
│ MQ Request Queue │
│ AWS.M2.CARDDEMO. │
│ PAUTH.REQUEST    │
└──────┬───────────┘
       │ Trigger
       ▼
┌──────────────────┐
│ CICS Transaction │
│ CP00 (COPAUA0C)  │
└──────┬───────────┘
       │
       ├─→ VSAM (Card/Account Validation)
       │
       ├─→ IMS DB (Store Authorization)
       │
       └─→ MQ Response Queue
           │
           ▼
       ┌──────────────────┐
       │ MQ Response Queue│
       │ AWS.M2.CARDDEMO. │
       │ PAUTH.REPLY      │
       └──────────────────┘

┌──────────────────┐
│ User Inquiry     │
│ CPVS/CPVD        │
└──────┬───────────┘
       │
       ├─→ IMS DB (Read Authorizations)
       │
       └─→ DB2 (Fraud Marking)
```

### Component Integration

**MQ Integration:**
- Request queue for incoming authorization requests
- Reply queue for authorization responses
- Trigger-based CICS transaction initiation

**IMS DB Integration:**
- HIDAM database structure
- Root segment: Authorization Summary
- Child segment: Authorization Details
- BMP and online PSBs

**DB2 Integration:**
- AUTHFRDS table for fraud tracking
- Embedded SQL in COBOL
- Two-phase commit with IMS

**VSAM Integration:**
- Read card and account data
- Validate credit limits
- Check card status

## Installation

### Prerequisites

1. **Base CardDemo**: Must be installed and operational
2. **IMS DB**: IMS database subsystem configured
3. **DB2**: DB2 subsystem configured
4. **MQ**: IBM MQ configured and accessible from CICS
5. **CICS**: CICS with IMS DB, DB2, and MQ support

### Installation Steps

#### Step 1: IMS Database Setup

Work with your IMS database administrator to create the necessary databases.

**Create DBDs (Database Definitions):**

Located in `app/app-authorization-ims-db2-mq/ims/`:
- **DBPAUTP0**: HIDAM primary database
- **DBPAUTX0**: HIDAM index database

**Create PSBs (Program Specification Blocks):**
- **PSBPAUTB**: BMP PSB for batch processing
- **PSBPAUTL**: PSB for online processing

**Allocate Datasets:**
```
DDPAUTP0 - Primary database dataset
DDPAUTX0 - Index database dataset
```

#### Step 2: DB2 Table Setup

Execute the following DDL to create the fraud tracking table:

```sql
CREATE TABLE CARDDEMO.AUTHFRDS                   
(CARD_NUM              CHAR(16)    NOT NULL,
 AUTH_TS                TIMESTAMP   NOT NULL,
 AUTH_TYPE              CHAR(4)             ,
 CARD_EXPIRY_DATE       CHAR(4)             ,
 MESSAGE_TYPE           CHAR(6)             ,
 MESSAGE_SOURCE         CHAR(6)             ,
 AUTH_ID_CODE           CHAR(6)             ,
 AUTH_RESP_CODE         CHAR(2)             ,
 AUTH_RESP_REASON       CHAR(4)             ,
 PROCESSING_CODE        CHAR(6)             ,
 TRANSACTION_AMT        DECIMAL(12,2)       ,
 APPROVED_AMT           DECIMAL(12,2)       ,
 MERCHANT_CATAGORY_CODE CHAR(4)             ,
 ACQR_COUNTRY_CODE      CHAR(3)             ,
 POS_ENTRY_MODE         SMALLINT            ,
 MERCHANT_ID            CHAR(15)            ,
 MERCHANT_NAME          VARCHAR(22)         ,
 MERCHANT_CITY          CHAR(13)            ,
 MERCHANT_STATE         CHAR(02)            ,
 MERCHANT_ZIP           CHAR(09)            ,
 TRANSACTION_ID         CHAR(15)            ,
 MATCH_STATUS           CHAR(1)             ,
 AUTH_FRAUD             CHAR(1)             ,
 FRAUD_RPT_DATE         DATE                ,
 ACCT_ID                DECIMAL(11)         ,
 CUST_ID                DECIMAL(9)          ,
 PRIMARY KEY(CARD_NUM,AUTH_TS))
IN AWSTSFRD.AWSTSFRD;

CREATE UNIQUE INDEX CARDDEMO.XAUTHFRD           
ON CARDDEMO.AUTHFRDS                         
(CARD_NUM ASC, AUTH_TS DESC)             
USING STOGROUP AWSTSSG                     
COPY YES;
```

**Important**: Update the DB2 schema name in program COPAUS2C.cbl to match your environment.

#### Step 3: MQ Configuration

Define MQ queues for authorization processing:

```
DEFINE QLOCAL('AWS.M2.CARDDEMO.PAUTH.REQUEST') REPLACE +
  DESCR('CardDemo Authorization Request Queue') +
  DEFPSIST(YES) +
  MAXDEPTH(5000)

DEFINE QLOCAL('AWS.M2.CARDDEMO.PAUTH.REPLY') REPLACE +
  DESCR('CardDemo Authorization Reply Queue') +
  DEFPSIST(YES) +
  MAXDEPTH(5000)
```

**Configure MQ Trigger:**
```
DEFINE PROCESS('CARDDEMO.AUTH.PROCESS') REPLACE +
  DESCR('CardDemo Authorization Process') +
  APPLTYPE(CICS) +
  APPLICID('CP00')

ALTER QLOCAL('AWS.M2.CARDDEMO.PAUTH.REQUEST') +
  PROCESS('CARDDEMO.AUTH.PROCESS') +
  TRIGGER +
  TRIGTYPE(FIRST) +
  INITQ('SYSTEM.CICS.INITQUEUE')
```

#### Step 4: Upload Source Code

Upload the module source code to mainframe datasets:

```
app/app-authorization-ims-db2-mq/cbl/    → CARDDEMO.CBL
app/app-authorization-ims-db2-mq/cpy/    → CARDDEMO.CPY
app/app-authorization-ims-db2-mq/bms/    → CARDDEMO.BMS
app/app-authorization-ims-db2-mq/jcl/    → CARDDEMO.JCL
app/app-authorization-ims-db2-mq/ddl/    → DB2 DDL
app/app-authorization-ims-db2-mq/ims/    → IMS DBD/PSB
```

#### Step 5: Compile Programs

Compile programs with appropriate options:

**CICS-IMS Programs:**
- COPAUA0C (MQ trigger program)
- COPAUS0C (Authorization summary)
- COPAUS1C (Authorization details)

**CICS-DB2 Programs:**
- COPAUS2C (Fraud marking)

**Batch-IMS Programs:**
- CBPAUP0C (Authorization purge)

Use compilation JCL templates provided in the module.

#### Step 6: Define CICS Resources

**Programs:**
```
CEDA DEFINE PROGRAM(COPAUA0C) GROUP(CARDDEMO) LANGUAGE(COBOL)
CEDA DEFINE PROGRAM(COPAUS0C) GROUP(CARDDEMO) LANGUAGE(COBOL)
CEDA DEFINE PROGRAM(COPAUS1C) GROUP(CARDDEMO) LANGUAGE(COBOL)
CEDA DEFINE PROGRAM(COPAUS2C) GROUP(CARDDEMO) LANGUAGE(COBOL)
```

**Mapsets:**
```
CEDA DEFINE MAPSET(COPAU00) GROUP(CARDDEMO)
CEDA DEFINE MAPSET(COPAU01) GROUP(CARDDEMO)
```

**Transactions:**
```
CEDA DEFINE TRANSACTION(CP00) GROUP(CARDDEMO) PROGRAM(COPAUA0C)
CEDA DEFINE TRANSACTION(CPVS) GROUP(CARDDEMO) PROGRAM(COPAUS0C)
CEDA DEFINE TRANSACTION(CPVD) GROUP(CARDDEMO) PROGRAM(COPAUS1C)
```

**DB2 Resources:**
```
CEDA DEFINE DB2ENTRY(DB201PLN) GROUP(CARDDEMO) PLAN(CARDDEMO)
CEDA DEFINE DB2TRAN(CPVDTRAN) ENTRY(DB201PLN) TRANSID(CPVD) GROUP(CARDDEMO)
```

**Install Resources:**
```
CEDA INSTALL GROUP(CARDDEMO)
```

#### Step 7: Enable Main Menu Option

The Pending Authorizations option (Option 11) will now be available in the Main Menu.

#### Step 8: Test Installation

1. Send a test authorization request via MQ
2. Verify authorization is processed
3. Check IMS DB for stored authorization
4. View authorization via CPVS transaction
5. Test fraud marking functionality

## Components

### Programs

#### COPAUA0C - Authorization Request Processor
- **Type**: CICS Online (MQ Trigger)
- **Transaction**: CP00
- **Function**: Process authorization requests from MQ
- **Databases**: IMS DB (write), VSAM (read)
- **Processing**:
  1. Read authorization request from MQ
  2. Validate card number and expiration
  3. Retrieve account and customer data from VSAM
  4. Apply business rules for approval/decline
  5. Store authorization in IMS DB
  6. Send response to MQ reply queue

#### COPAUS0C - Authorization Summary
- **Type**: CICS Online
- **Transaction**: CPVS
- **BMS Map**: COPAU00
- **Function**: Display pending authorizations for an account
- **Databases**: IMS DB (read), VSAM (read)
- **Features**: Paging, selection for details

#### COPAUS1C - Authorization Details
- **Type**: CICS Online
- **Transaction**: CPVD
- **BMS Map**: COPAU01
- **Function**: Display detailed authorization information
- **Databases**: IMS DB (read)
- **Features**: Fraud marking (PF5)

#### COPAUS2C - Fraud Marking
- **Type**: CICS Online (Called)
- **Function**: Mark authorization as fraudulent
- **Databases**: DB2 (write)
- **Processing**: Insert fraud record into AUTHFRDS table

#### CBPAUP0C - Authorization Purge
- **Type**: Batch (BMP)
- **Job**: CBPAUP0J
- **Function**: Purge expired authorizations
- **Databases**: IMS DB (delete)
- **Processing**:
  1. Read all authorizations
  2. Identify expired authorizations (> 30 days)
  3. Delete expired records
  4. Adjust available credit if unmatched

### Copybooks

#### CIPAUSMY - Authorization Summary Segment
- IMS segment definition for authorization summary
- Root segment in HIDAM database

#### CIPAUDTY - Authorization Details Segment
- IMS segment definition for authorization details
- Child segment in HIDAM database

#### CCPAURQY - Authorization Request Structure
- MQ message format for authorization requests
- CSV format parsing

#### CCPAURLY - Authorization Response Structure
- MQ message format for authorization responses
- CSV format generation

## Data Models

### IMS DB Structure

**Database**: DBPAUTP0 (HIDAM)

**Segments:**

**PAUTSUM0 (Root Segment):**
```
01  PEND-AUTH-SUMMARY.
    05  PAUTH-ACCT-ID              PIC 9(11).
    05  PAUTH-CARD-NUM             PIC 9(16).
    05  PAUTH-COUNT                PIC 9(03).
    05  PAUTH-LAST-AUTH-TS         PIC X(26).
```

**PAUTDTL1 (Child Segment):**
```
01  PEND-AUTH-DETAIL.
    05  PAUTH-DTL-AUTH-TS          PIC X(26).
    05  PAUTH-DTL-AUTH-TYPE        PIC X(04).
    05  PAUTH-DTL-CARD-EXPIRY      PIC X(04).
    05  PAUTH-DTL-MSG-TYPE         PIC X(06).
    05  PAUTH-DTL-MSG-SOURCE       PIC X(06).
    05  PAUTH-DTL-PROC-CODE        PIC X(06).
    05  PAUTH-DTL-TRAN-AMT         PIC S9(10)V99 COMP-3.
    05  PAUTH-DTL-MERCHANT-CAT     PIC X(04).
    05  PAUTH-DTL-ACQR-COUNTRY     PIC X(03).
    05  PAUTH-DTL-POS-ENTRY        PIC 9(04).
    05  PAUTH-DTL-MERCHANT-ID      PIC X(15).
    05  PAUTH-DTL-MERCHANT-NAME    PIC X(22).
    05  PAUTH-DTL-MERCHANT-CITY    PIC X(13).
    05  PAUTH-DTL-MERCHANT-STATE   PIC X(02).
    05  PAUTH-DTL-MERCHANT-ZIP     PIC X(09).
    05  PAUTH-DTL-TRAN-ID          PIC X(15).
    05  PAUTH-DTL-AUTH-ID          PIC X(06).
    05  PAUTH-DTL-AUTH-RESP-CD     PIC X(02).
    05  PAUTH-DTL-AUTH-RESP-REASON PIC X(04).
    05  PAUTH-DTL-APPROVED-AMT     PIC S9(10)V99 COMP-3.
```

### MQ Message Formats

**Authorization Request (CSV Format):**
```
AUTH-DATE,AUTH-TIME,CARD-NUM,AUTH-TYPE,CARD-EXPIRY-DATE,
MESSAGE-TYPE,MESSAGE-SOURCE,PROCESSING-CODE,TRANSACTION-AMT,
MERCHANT-CATEGORY-CODE,ACQR-COUNTRY-CODE,POS-ENTRY-MODE,
MERCHANT-ID,MERCHANT-NAME,MERCHANT-CITY,MERCHANT-STATE,
MERCHANT-ZIP,TRANSACTION-ID
```

**Authorization Response (CSV Format):**
```
CARD-NUM,TRANSACTION-ID,AUTH-ID-CODE,AUTH-RESP-CODE,
AUTH-RESP-REASON,APPROVED-AMT
```

**Response Codes:**
- **00**: Approved
- **05**: Declined - Insufficient Funds
- **14**: Declined - Invalid Card
- **54**: Declined - Expired Card
- **57**: Declined - Transaction Not Permitted

## User Guide

### Viewing Pending Authorizations

1. From Main Menu, select Option 11 (Pending Authorizations)
2. Or enter transaction: `CPVS`
3. Enter Account Number
4. Press **Enter**
5. List of pending authorizations displays

**Screen Fields:**
- Authorization ID
- Card Number
- Authorization Date/Time
- Merchant Name
- Amount
- Status

**Actions:**
- Enter 'S' next to authorization to view details
- Use PF7/PF8 to page through list

### Viewing Authorization Details

1. Select authorization from summary screen
2. Or enter transaction: `CPVD` with authorization ID
3. Complete authorization details display

**Information Shown:**
- Authorization timestamp
- Card and account information
- Transaction amount
- Merchant details (name, city, state, ZIP)
- Authorization response code
- Approval/decline reason

**Fraud Marking:**
- Press **PF5** to mark as fraudulent
- Confirmation prompt displays
- Record inserted into DB2 AUTHFRDS table
- Fraud indicator set in IMS DB

### Batch Purge Process

**Job**: CBPAUP0J

**Schedule**: Daily (recommended)

**Processing:**
1. Reads all authorizations from IMS DB
2. Identifies authorizations older than 30 days
3. Deletes expired authorizations
4. Adjusts available credit for unmatched authorizations
5. Generates purge report

**Submit:**
```
Submit: AWS.M2.CARDDEMO.JCL(CBPAUP0J)
```

## Technical Details

### Business Rules

**Authorization Approval Logic:**

1. **Card Validation**:
   - Card must exist in CARDDAT
   - Card must be active
   - Card must not be expired

2. **Credit Limit Check**:
   - Transaction amount + current balance ≤ credit limit
   - Available credit must be sufficient

3. **Fraud Checks**:
   - Unusual transaction patterns
   - High-risk merchant categories
   - Geographic anomalies

**Approval**: All checks pass → Response code 00
**Decline**: Any check fails → Appropriate decline code

### IMS DB Operations

**Insert Authorization:**
```cobol
CALL 'CBLTDLI' USING DLI-ISRT
                     PSBPAUTL
                     PAUTSUM0-SEGMENT
                     PAUTSUM0-DATA.
```

**Read Authorization:**
```cobol
CALL 'CBLTDLI' USING DLI-GU
                     PSBPAUTL
                     PAUTSUM0-SEGMENT
                     PAUTSUM0-DATA
                     PAUTSUM0-SSA.
```

**Delete Authorization:**
```cobol
CALL 'CBLTDLI' USING DLI-DLET
                     PSBPAUTB
                     PAUTSUM0-SEGMENT.
```

### DB2 Operations

**Insert Fraud Record:**
```cobol
EXEC SQL
    INSERT INTO CARDDEMO.AUTHFRDS
    (CARD_NUM, AUTH_TS, AUTH_TYPE, ...)
    VALUES
    (:WS-CARD-NUM, :WS-AUTH-TS, :WS-AUTH-TYPE, ...)
END-EXEC.
```

### MQ Operations

**Read Request:**
```cobol
EXEC CICS READQ TS
    QUEUE('AWS.M2.CARDDEMO.PAUTH.REQUEST')
    INTO(WS-REQUEST-MSG)
    LENGTH(WS-MSG-LEN)
END-EXEC.
```

**Write Response:**
```cobol
EXEC CICS WRITEQ TS
    QUEUE('AWS.M2.CARDDEMO.PAUTH.REPLY')
    FROM(WS-RESPONSE-MSG)
    LENGTH(WS-MSG-LEN)
END-EXEC.
```

## Troubleshooting

### Common Issues

**Issue: Authorization requests not processing**
- Check MQ trigger is active
- Verify CP00 transaction is enabled
- Check IMS DB is available
- Review CICS logs for errors

**Issue: Cannot view authorizations (CPVS)**
- Verify IMS DB is open
- Check PSB is allocated
- Verify account has authorizations

**Issue: Fraud marking fails**
- Check DB2 connection
- Verify AUTHFRDS table exists
- Check DB2 plan is bound
- Review SQLCA for error codes

**Issue: Batch purge fails**
- Verify IMS DB is available for batch
- Check BMP region is configured
- Review job output for IMS status codes

### Error Codes

**IMS Status Codes:**
- **GE**: Segment not found
- **GB**: End of database
- **II**: Invalid SSA
- **DA**: Key out of sequence

**DB2 SQL Codes:**
- **-803**: Duplicate key
- **-911**: Deadlock
- **-913**: Resource unavailable

---

**Navigation**: [Home](Home.md) | [Optional Modules](Home.md#optional-modules) | [Installation Guide](Installation-Guide.md) | [Technical Components](Technical-Components.md)
