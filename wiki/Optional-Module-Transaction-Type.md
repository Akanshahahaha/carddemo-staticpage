# Optional Module: Transaction Type Management (DB2)

Complete documentation for the Transaction Type Management optional module, which adds DB2-based transaction type maintenance capabilities.

## Table of Contents
- [Module Overview](#module-overview)
- [Features](#features)
- [Architecture](#architecture)
- [Installation](#installation)
- [Components](#components)
- [Data Models](#data-models)
- [User Guide](#user-guide)
- [Technical Details](#technical-details)
- [Integration](#integration)

## Module Overview

The Transaction Type Management module is an optional extension that demonstrates DB2 integration patterns using embedded static SQL. This module allows administrators to maintain transaction type reference data in DB2 tables, providing a more robust and relational approach to managing transaction metadata.

### Business Value

- **Centralized Reference Data**: Single source of truth for transaction types
- **Relational Data Management**: Leverage DB2 capabilities for data integrity
- **Online and Batch Maintenance**: Flexible data management options
- **Data Synchronization**: Extract DB2 data to VSAM for transaction processing
- **DB2 Integration Patterns**: Demonstrates various DB2 programming techniques

### Technologies Used

- **COBOL**: Business logic with embedded SQL
- **CICS**: Online transaction processing
- **DB2**: Relational database for transaction types
- **VSAM**: Synchronized reference data for transaction processing
- **JCL**: Batch data extraction and maintenance

## Features

### Online Transaction Type Management

1. **Transaction Type List (CTLI)**
   - Browse all transaction types
   - Forward and backward paging with cursors
   - Inline updates of descriptions
   - Delete transaction types
   - Referential integrity checking

2. **Transaction Type Add/Edit (CTTU)**
   - Add new transaction types
   - Edit existing transaction type descriptions
   - Validate transaction type codes
   - Associate with categories

### Batch Processing

1. **Database Creation (CREADB21)**
   - Create DB2 database and tables
   - Load initial transaction type data
   - Set up referential integrity constraints

2. **Data Extraction (TRANEXTR)**
   - Extract transaction types from DB2
   - Generate VSAM-compatible files
   - Synchronize with transaction processing system

3. **Batch Maintenance (MNTTRDB2)**
   - Bulk updates to transaction types
   - Data validation and error handling

### DB2 Integration Patterns

- **Static Embedded SQL**: SQL statements embedded in COBOL
- **Cursor Processing**: Forward and backward cursor navigation
- **CRUD Operations**: Complete Create, Read, Update, Delete
- **Transaction Management**: Commit/rollback handling
- **Error Handling**: SQLCA error checking
- **Host Variables**: COBOL-DB2 data exchange

## Architecture

### Component Architecture

```
┌──────────────────┐
│  Admin User      │
│  (CICS Terminal) │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ CICS Transactions│
│  CTLI / CTTU     │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ COBOL Programs   │
│ COTRTLIC/COTRTUPC│
│ (Embedded SQL)   │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ DB2 Subsystem    │
│ CARDDEMO Database│
└────────┬─────────┘
         │
         ├─→ TRANSACTION_TYPE Table
         │
         └─→ TRANSACTION_TYPE_CATEGORY Table

┌──────────────────┐
│ Batch Job        │
│ TRANEXTR         │
└────────┬─────────┘
         │
         ├─→ DB2 (Extract)
         │
         └─→ VSAM Files (Load)
```

### Data Flow

**Online Updates:**
1. User enters transaction via CTLI or CTTU
2. COBOL program executes embedded SQL
3. DB2 updates transaction type tables
4. Changes committed to database
5. Confirmation displayed to user

**Batch Synchronization:**
1. TRANEXTR job executes
2. Extracts data from DB2 tables
3. Formats data for VSAM
4. Loads TRANTYPE and TRANCATG VSAM files
5. Transaction processing uses updated reference data

## Installation

### Prerequisites

1. **Base CardDemo**: Installed and operational
2. **DB2 Subsystem**: Configured and accessible
3. **CICS with DB2**: CICS-DB2 attachment configured
4. **DB2 Precompiler**: Available for COBOL compilation

### Installation Steps

#### Step 1: Create DB2 Database and Tables

Submit the CREADB21 job to create the database and tables:

```
Submit: AWS.M2.CARDDEMO.JCL(CREADB21)
```

This job executes DDL to create:
- CARDDEMO database
- TRANSACTION_TYPE table
- TRANSACTION_TYPE_CATEGORY table
- Indexes
- Referential integrity constraints

**DDL Location**: `app/app-transaction-type-db2/ddl/`

#### Step 2: Upload Source Code

Upload module source code to mainframe datasets:

```
app/app-transaction-type-db2/cbl/     → CARDDEMO.CBL
app/app-transaction-type-db2/cpy/     → CARDDEMO.CPY
app/app-transaction-type-db2/bms/     → CARDDEMO.BMS
app/app-transaction-type-db2/jcl/     → CARDDEMO.JCL
app/app-transaction-type-db2/dcl/     → DB2 declarations
app/app-transaction-type-db2/ddl/     → DB2 DDL
```

#### Step 3: Precompile and Compile Programs

**Precompile COBOL programs with DB2:**

```jcl
//PRECOMP  EXEC PGM=DSNHPC,PARM='HOST(COBOL)'
//STEPLIB  DD DSN=DB2.SDSNLOAD,DISP=SHR
//DBRMLIB  DD DSN=CARDDEMO.DBRMLIB,DISP=SHR
//SYSIN    DD DSN=CARDDEMO.CBL(COTRTLIC),DISP=SHR
//SYSCIN   DD DSN=&&SYSCIN,DISP=(MOD,PASS)
```

**Compile precompiled source:**

```jcl
//COMPILE  EXEC PGM=IGYCRCTL
//STEPLIB  DD DSN=COBOL.SIGYCOMP,DISP=SHR
//SYSIN    DD DSN=&&SYSCIN,DISP=(OLD,DELETE)
//SYSLIN   DD DSN=&&LOADSET,DISP=(MOD,PASS)
```

**Bind DB2 Plan:**

```jcl
//BIND     EXEC PGM=IKJEFT01
//SYSTSPRT DD SYSOUT=*
//SYSTSIN  DD *
  DSN SYSTEM(DB2P)
  BIND PLAN(CARDDEMO) -
       MEMBER(COTRTLIC,COTRTUPC) -
       ACTION(REPLACE) -
       ISOLATION(CS) -
       VALIDATE(BIND) -
       RELEASE(COMMIT)
  END
/*
```

Programs to compile:
- **COTRTLIC**: Transaction type list/update/delete
- **COTRTUPC**: Transaction type add/edit
- **COBTUPDT**: Batch maintenance program

#### Step 4: Define CICS Resources

**Programs:**
```
CEDA DEFINE PROGRAM(COTRTLIC) GROUP(CARDDEMO) LANGUAGE(COBOL)
CEDA DEFINE PROGRAM(COTRTUPC) GROUP(CARDDEMO) LANGUAGE(COBOL)
```

**Mapsets:**
```
CEDA DEFINE MAPSET(COTRTLI) GROUP(CARDDEMO)
CEDA DEFINE MAPSET(COTRTUP) GROUP(CARDDEMO)
```

**Transactions:**
```
CEDA DEFINE TRANSACTION(CTLI) GROUP(CARDDEMO) PROGRAM(COTRTLIC)
CEDA DEFINE TRANSACTION(CTTU) GROUP(CARDDEMO) PROGRAM(COTRTUPC)
```

**DB2 Resources:**
```
CEDA DEFINE DB2CONN(DB2CONN1) GROUP(CARDDEMO) +
  DB2GROUPID(DB2GROUP) +
  CONNECTST(YES)

CEDA DEFINE DB2ENTRY(CARDDEMO) GROUP(CARDDEMO) +
  PLAN(CARDDEMO) +
  THREADLIMIT(50) +
  THREADWAIT(YES)

CEDA DEFINE DB2TRAN(CTLITRAN) GROUP(CARDDEMO) +
  ENTRY(CARDDEMO) +
  TRANSID(CTLI)

CEDA DEFINE DB2TRAN(CTTUTRAN) GROUP(CARDDEMO) +
  ENTRY(CARDDEMO) +
  TRANSID(CTTU)
```

**Install Resources:**
```
CEDA INSTALL GROUP(CARDDEMO)
```

#### Step 5: Extract Initial Data

Run TRANEXTR to extract DB2 data to VSAM:

```
Submit: AWS.M2.CARDDEMO.JCL(TRANEXTR)
```

This creates VSAM files that transaction processing uses.

#### Step 6: Enable Admin Menu Options

Options 5 and 6 will now be available in the Admin Menu (CA00):
- Option 5: Transaction Type List (CTLI)
- Option 6: Transaction Type Add/Edit (CTTU)

#### Step 7: Test Installation

1. Sign on as admin (ADMIN001/PASSWORD)
2. Access Admin Menu (CA00)
3. Select Option 5 (CTLI)
4. Verify transaction types display
5. Test update and delete functions
6. Select Option 6 (CTTU)
7. Test add and edit functions

## Components

### Programs

#### COTRTLIC - Transaction Type List/Update/Delete
- **Transaction**: CTLI
- **BMS Map**: COTRTLI
- **Function**: List, update, and delete transaction types
- **Database**: DB2 TRANSACTION_TYPE table
- **Features**:
  - Forward/backward cursor paging
  - Inline description updates
  - Delete with referential integrity check
  - Error handling with SQLCA

**Key SQL Operations:**
```cobol
* Declare cursor for paging
EXEC SQL
    DECLARE TRAN_TYPE_CURSOR CURSOR FOR
    SELECT TR_TYPE, TR_DESCRIPTION
    FROM CARDDEMO.TRANSACTION_TYPE
    ORDER BY TR_TYPE
END-EXEC.

* Open cursor
EXEC SQL
    OPEN TRAN_TYPE_CURSOR
END-EXEC.

* Fetch next row
EXEC SQL
    FETCH NEXT TRAN_TYPE_CURSOR
    INTO :WS-TR-TYPE, :WS-TR-DESCRIPTION
END-EXEC.

* Update transaction type
EXEC SQL
    UPDATE CARDDEMO.TRANSACTION_TYPE
    SET TR_DESCRIPTION = :WS-NEW-DESCRIPTION
    WHERE TR_TYPE = :WS-TR-TYPE
END-EXEC.

* Delete transaction type
EXEC SQL
    DELETE FROM CARDDEMO.TRANSACTION_TYPE
    WHERE TR_TYPE = :WS-TR-TYPE
END-EXEC.
```

#### COTRTUPC - Transaction Type Add/Edit
- **Transaction**: CTTU
- **BMS Map**: COTRTUP
- **Function**: Add new or edit existing transaction types
- **Database**: DB2 TRANSACTION_TYPE table
- **Features**:
  - Add new transaction types
  - Edit existing descriptions
  - Validate transaction type codes
  - Category association

**Key SQL Operations:**
```cobol
* Insert new transaction type
EXEC SQL
    INSERT INTO CARDDEMO.TRANSACTION_TYPE
    (TR_TYPE, TR_DESCRIPTION)
    VALUES (:WS-TR-TYPE, :WS-TR-DESCRIPTION)
END-EXEC.

* Select for update
EXEC SQL
    SELECT TR_TYPE, TR_DESCRIPTION
    INTO :WS-TR-TYPE, :WS-TR-DESCRIPTION
    FROM CARDDEMO.TRANSACTION_TYPE
    WHERE TR_TYPE = :WS-INPUT-TYPE
END-EXEC.

* Update transaction type
EXEC SQL
    UPDATE CARDDEMO.TRANSACTION_TYPE
    SET TR_DESCRIPTION = :WS-NEW-DESCRIPTION
    WHERE TR_TYPE = :WS-TR-TYPE
END-EXEC.
```

#### COBTUPDT - Batch Maintenance Program
- **Job**: MNTTRDB2
- **Function**: Batch updates to transaction types
- **Database**: DB2 TRANSACTION_TYPE table
- **Processing**:
  - Read input file with updates
  - Validate transaction type codes
  - Execute updates in DB2
  - Generate update report

### Copybooks

#### Transaction Type Copybooks
- **DCLTRAN**: DB2 table declarations
- **TRANTYPECPY**: Transaction type structure
- **TRANCATCPY**: Transaction category structure

### BMS Maps

#### COTRTLI - Transaction Type List Map
- Display area for transaction type list
- Action fields for update/delete
- PF keys for paging and navigation

#### COTRTUP - Transaction Type Add/Edit Map
- Input fields for transaction type code
- Input field for description
- Category selection
- Function keys for save/cancel

## Data Models

### DB2 Tables

#### TRANSACTION_TYPE Table

```sql
CREATE TABLE CARDDEMO.TRANSACTION_TYPE
(TR_TYPE              CHAR(2)      NOT NULL,
 TR_DESCRIPTION       VARCHAR(50)  NOT NULL,
 PRIMARY KEY (TR_TYPE))
IN CARDDEMO.CARDTS;
```

**Columns:**
- **TR_TYPE**: 2-character transaction type code (PK)
- **TR_DESCRIPTION**: Description of transaction type

**Sample Data:**
```
TR_TYPE  TR_DESCRIPTION
-------  ------------------
PUR      Purchase
PAY      Payment
ADJ      Adjustment
FEE      Fee
INT      Interest
REF      Refund
RET      Return
```

#### TRANSACTION_TYPE_CATEGORY Table

```sql
CREATE TABLE CARDDEMO.TRANSACTION_TYPE_CATEGORY
(TRC_TYPE_CODE        CHAR(2)      NOT NULL,
 TRC_TYPE_CATEGORY    CHAR(4)      NOT NULL,
 TRC_CAT_DATA         VARCHAR(50)  NOT NULL,
 PRIMARY KEY (TRC_TYPE_CODE, TRC_TYPE_CATEGORY),
 FOREIGN KEY (TRC_TYPE_CODE) 
   REFERENCES CARDDEMO.TRANSACTION_TYPE(TR_TYPE)
   ON DELETE RESTRICT)
IN CARDDEMO.CARDTS;
```

**Columns:**
- **TRC_TYPE_CODE**: Transaction type code (PK, FK)
- **TRC_TYPE_CATEGORY**: Category code (PK)
- **TRC_CAT_DATA**: Category description

**Referential Integrity:**
- Foreign key from TRC_TYPE_CODE to TR_TYPE
- DELETE RESTRICT prevents deletion of types with categories

**Sample Data:**
```
TRC_TYPE_CODE  TRC_TYPE_CATEGORY  TRC_CAT_DATA
-------------  -----------------  ----------------
PUR            5411               Grocery Stores
PUR            5812               Restaurants
PUR            5541               Gas Stations
PAY            0000               Payment
```

### Indexes

```sql
CREATE UNIQUE INDEX CARDDEMO.XTRANTYPE
ON CARDDEMO.TRANSACTION_TYPE (TR_TYPE ASC)
USING STOGROUP CARDDEMOSG;

CREATE UNIQUE INDEX CARDDEMO.XTRANTCAT
ON CARDDEMO.TRANSACTION_TYPE_CATEGORY 
(TRC_TYPE_CODE ASC, TRC_TYPE_CATEGORY ASC)
USING STOGROUP CARDDEMOSG;
```

## User Guide

### Listing Transaction Types (CTLI)

1. Sign on as admin
2. Access Admin Menu (CA00)
3. Select Option 5 or enter `CTLI`
4. Transaction type list displays

**Screen Layout:**
```
Type  Description
----  ---------------------
PUR   Purchase
PAY   Payment
ADJ   Adjustment
FEE   Fee
INT   Interest

Action: U=Update D=Delete
PF7=Backward PF8=Forward PF3=Exit
```

**Actions:**
- Enter 'U' next to a type to update description
- Enter 'D' next to a type to delete
- Press **PF7** for previous page
- Press **PF8** for next page
- Press **PF3** to exit

### Updating Transaction Type

1. From CTLI screen, enter 'U' next to transaction type
2. Modify the description field
3. Press **Enter** to save
4. Confirmation message displays

**Business Rules:**
- Description cannot be blank
- Changes are immediate
- No undo available

### Deleting Transaction Type

1. From CTLI screen, enter 'D' next to transaction type
2. Press **Enter**
3. Confirmation prompt displays
4. Enter 'Y' to confirm or 'N' to cancel
5. Deletion processes if no referential integrity violations

**Business Rules:**
- Cannot delete if categories exist
- Cannot delete if transactions exist with this type
- Deletion is permanent

### Adding Transaction Type (CTTU)

1. Access Admin Menu (CA00)
2. Select Option 6 or enter `CTTU`
3. Enter new transaction type code (2 characters)
4. Enter description
5. Press **Enter** to save

**Screen Layout:**
```
Add Transaction Type

Transaction Type Code: __
Description: _____________________________

PF3=Cancel PF5=Clear
```

**Business Rules:**
- Type code must be 2 characters
- Type code must be unique
- Description is required
- Type code cannot be changed after creation

### Editing Transaction Type (CTTU)

1. Enter `CTTU`
2. Enter existing transaction type code
3. Press **Enter** to retrieve
4. Modify description
5. Press **Enter** to save

**Business Rules:**
- Only description can be modified
- Type code cannot be changed

## Technical Details

### DB2 Precompiler Options

```
DSNHPC PARM='HOST(COBOL),APOSTSQL,STDSQL(YES)'
```

**Options:**
- **HOST(COBOL)**: COBOL host language
- **APOSTSQL**: Use apostrophes for SQL strings
- **STDSQL(YES)**: Use standard SQL

### SQLCA (SQL Communication Area)

All programs include SQLCA for error handling:

```cobol
EXEC SQL
    INCLUDE SQLCA
END-EXEC.

* Check SQL return code
IF SQLCODE NOT = 0
    PERFORM ERROR-HANDLING
END-IF.
```

**Common SQLCODE Values:**
- **0**: Successful execution
- **100**: No data found
- **-803**: Duplicate key violation
- **-911**: Deadlock or timeout
- **-913**: Resource unavailable

### Cursor Processing

**Forward Paging:**
```cobol
EXEC SQL
    FETCH NEXT TRAN_TYPE_CURSOR
    INTO :WS-TR-TYPE, :WS-TR-DESCRIPTION
END-EXEC.
```

**Backward Paging:**
```cobol
EXEC SQL
    FETCH PRIOR TRAN_TYPE_CURSOR
    INTO :WS-TR-TYPE, :WS-TR-DESCRIPTION
END-EXEC.
```

**Cursor Positioning:**
```cobol
EXEC SQL
    FETCH FIRST TRAN_TYPE_CURSOR
    INTO :WS-TR-TYPE, :WS-TR-DESCRIPTION
END-EXEC.

EXEC SQL
    FETCH LAST TRAN_TYPE_CURSOR
    INTO :WS-TR-TYPE, :WS-TR-DESCRIPTION
END-EXEC.
```

### Transaction Management

**Commit:**
```cobol
EXEC SQL
    COMMIT
END-EXEC.
```

**Rollback:**
```cobol
EXEC SQL
    ROLLBACK
END-EXEC.
```

**CICS Syncpoint:**
```cobol
EXEC CICS SYNCPOINT
END-EXEC.
```

### Host Variables

**Declaration:**
```cobol
01  WS-TR-TYPE              PIC X(02).
01  WS-TR-DESCRIPTION       PIC X(50).
```

**Usage in SQL:**
```cobol
EXEC SQL
    SELECT TR_TYPE, TR_DESCRIPTION
    INTO :WS-TR-TYPE, :WS-TR-DESCRIPTION
    FROM CARDDEMO.TRANSACTION_TYPE
    WHERE TR_TYPE = :WS-INPUT-TYPE
END-EXEC.
```

## Integration

### Integration with Base Application

**Data Synchronization:**

The TRANEXTR job extracts transaction types from DB2 and creates VSAM files:

```
DB2 TRANSACTION_TYPE → TRANTYPE.PS → TRANTYPE.VSAM.KSDS
DB2 TRANSACTION_TYPE_CATEGORY → TRANCATG.PS → TRANCATG.VSAM.KSDS
```

**Batch Job Flow:**
1. TRANEXTR extracts data from DB2
2. Formats data for VSAM
3. TRANTYPE job loads VSAM file
4. TRANCATG job loads VSAM file
5. Transaction processing uses VSAM files

**Why Dual Storage?**
- **DB2**: Administrative maintenance, relational integrity
- **VSAM**: High-performance transaction processing
- **Best of Both**: Flexibility and performance

### Admin Menu Integration

Options 5 and 6 are added to the Admin Menu (CA00):

```cobol
05  ADMIN-MENU-OPTIONS.
    10  OPTION-05          PIC X(50) VALUE
        '5. Transaction Type List'.
    10  OPTION-06          PIC X(50) VALUE
        '6. Transaction Type Add/Edit'.
```

### Batch Schedule Integration

TRANEXTR should run daily after transaction posting:

```
Daily Batch Sequence:
1. CLOSEFIL
2. TRANBKP
3. POSTTRAN
4. TRANEXTR  ← Extract DB2 data
5. TRANCATG  ← Load VSAM
6. TRANTYPE  ← Load VSAM
7. COMBTRAN
8. TRANIDX
9. OPENFIL
```

## Troubleshooting

### Common Issues

**Issue: SQLCODE -803 (Duplicate Key)**
- **Cause**: Attempting to insert duplicate transaction type
- **Solution**: Check if type already exists, use update instead

**Issue: SQLCODE -911 (Deadlock)**
- **Cause**: Multiple users updating same record
- **Solution**: Retry transaction, implement wait logic

**Issue: SQLCODE -913 (Resource Unavailable)**
- **Cause**: DB2 resource limit reached
- **Solution**: Check DB2 configuration, increase thread limit

**Issue: Cannot delete transaction type**
- **Cause**: Referential integrity constraint
- **Solution**: Delete dependent categories first, or keep type and mark inactive

**Issue: DB2 connection fails**
- **Cause**: DB2 subsystem not available or CICS-DB2 attachment issue
- **Solution**: Check DB2 status, verify CICS-DB2 connection

**Issue: Cursor not scrolling**
- **Cause**: Cursor not declared as scrollable
- **Solution**: Verify cursor declaration includes scrollable option

### Performance Considerations

**Cursor Performance:**
- Use appropriate fetch size
- Close cursors when done
- Consider result set size

**Index Usage:**
- Ensure indexes are used in queries
- Run RUNSTATS regularly
- Monitor access paths

**Connection Pooling:**
- Configure appropriate thread limits
- Monitor thread usage
- Adjust based on workload

---

**Navigation**: [Home](Home.md) | [Optional Modules](Home.md#optional-modules) | [Installation Guide](Installation-Guide.md) | [Technical Components](Technical-Components.md)
