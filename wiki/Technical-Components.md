# Technical Components

Comprehensive reference for all CardDemo technical components including programs, files, copybooks, and resources.

## Table of Contents
- [Online Programs](#online-programs)
- [Batch Programs](#batch-programs)
- [Utility Programs](#utility-programs)
- [BMS Maps](#bms-maps)
- [Copybooks](#copybooks)
- [VSAM Files](#vsam-files)
- [CICS Transactions](#cics-transactions)
- [JCL Jobs](#jcl-jobs)
- [Program Specifications](#program-specifications)

## Online Programs

### Sign-On and Menu Programs

#### COSGN00C - Sign-On Program
- **Transaction**: CC00
- **BMS Map**: COSGN00
- **Function**: User authentication and sign-on
- **Files Accessed**: USRSEC (User Security)
- **Processing**:
  - Validates user credentials
  - Checks account status
  - Establishes user session
  - Routes to main menu on success
- **Error Handling**: Invalid credentials, locked accounts, system errors

#### COMEN01C - Main Menu Program
- **Transaction**: CM00
- **BMS Map**: COMEN01
- **Function**: Main menu display and navigation
- **Processing**:
  - Displays menu options based on user type
  - Routes to selected function
  - Maintains session context
- **User Types**: Regular users see options 1-11, Admin users see option 12

### Account Management Programs

#### COACTVWC - Account View Program
- **Transaction**: CAVW
- **BMS Map**: COACTVW
- **Function**: Display account details
- **Files Accessed**: ACCTDAT, CUSTDAT
- **Input**: Account Number (11 digits)
- **Output**: Account details, balances, customer information
- **Processing**:
  - Validates account number
  - Retrieves account record
  - Retrieves customer record
  - Formats and displays data

#### COACTUPC - Account Update Program
- **Transaction**: CAUP
- **BMS Map**: COACTUP
- **Function**: Update account information
- **Files Accessed**: ACCTDAT (update)
- **Updatable Fields**: Credit limit, cash credit limit, status
- **Processing**:
  - Retrieves account for update
  - Validates changes
  - Updates account record
  - Logs changes
- **Business Rules**: Credit limit validation, status change rules

### Card Management Programs

#### COCRDLIC - Card List Program
- **Transaction**: CCLI
- **BMS Map**: COCRDLI
- **Function**: List cards for an account
- **Files Accessed**: CARDDAT, CARDXREF
- **Input**: Account Number
- **Output**: List of cards with status
- **Processing**:
  - Retrieves cross-reference records
  - Retrieves card records
  - Formats list display
  - Supports paging (PF7/PF8)

#### COCRDSLC - Card View Program
- **Transaction**: CCDL
- **BMS Map**: COCRDSL
- **Function**: Display card details
- **Files Accessed**: CARDDAT
- **Input**: Card Number (16 digits)
- **Output**: Complete card information
- **Processing**:
  - Validates card number
  - Retrieves card record
  - Formats and displays data

#### COCRDUPC - Card Update Program
- **Transaction**: CCUP
- **BMS Map**: COCRDUP
- **Function**: Update card information
- **Files Accessed**: CARDDAT (update)
- **Updatable Fields**: Status, expiration date, cardholder name
- **Processing**:
  - Retrieves card for update
  - Validates changes
  - Updates card record
  - Logs changes

### Transaction Management Programs

#### COTRN00C - Transaction List Program
- **Transaction**: CT00
- **BMS Map**: COTRN00
- **Function**: List transactions for a card
- **Files Accessed**: TRANSACT (via AIX)
- **Input**: Card Number, optional date range
- **Output**: Transaction list
- **Processing**:
  - Uses alternate index for card number access
  - Filters by date range if specified
  - Supports paging
  - Sorts by date descending

#### COTRN01C - Transaction View Program
- **Transaction**: CT01
- **BMS Map**: COTRN01
- **Function**: Display transaction details
- **Files Accessed**: TRANSACT
- **Input**: Transaction ID
- **Output**: Complete transaction information
- **Processing**:
  - Retrieves transaction record
  - Formats merchant information
  - Displays authorization details

#### COTRN02C - Transaction Add Program
- **Transaction**: CT02
- **BMS Map**: COTRN02
- **Function**: Add new transaction
- **Files Accessed**: TRANSACT (add), CARDDAT (read), ACCTDAT (update)
- **Input**: Transaction details
- **Processing**:
  - Validates card status
  - Checks available credit
  - Generates transaction ID
  - Writes transaction record
  - Updates account balance
- **Business Rules**: Credit limit checking, card status validation

### Bill Payment and Report Programs

#### COBIL00C - Bill Payment Program
- **Transaction**: CB00
- **BMS Map**: COBIL00
- **Function**: Process bill payments
- **Files Accessed**: ACCTDAT (update), TRANSACT (add)
- **Input**: Account, card, payment amount
- **Processing**:
  - Validates payment amount
  - Creates payment transaction
  - Updates account balance
  - Updates available credit
- **Business Rules**: Payment validation, balance checking

#### CORPT00C - Report Program
- **Transaction**: CR00
- **BMS Map**: CORPT00
- **Function**: Generate transaction reports
- **Files Accessed**: TRANSACT, ACCTDAT
- **Report Types**: Account summary, transaction detail, category summary
- **Processing**:
  - Filters transactions by criteria
  - Aggregates data
  - Formats report output
  - Supports paging

### Admin Programs

#### COADM01C - Admin Menu Program
- **Transaction**: CA00
- **BMS Map**: COADM01
- **Function**: Admin menu display
- **Processing**:
  - Verifies admin privileges
  - Displays admin options
  - Routes to admin functions

#### COUSR00C - User List Program
- **Transaction**: CU00
- **BMS Map**: COUSR00
- **Function**: List all users
- **Files Accessed**: USRSEC
- **Output**: User list with details
- **Processing**:
  - Retrieves all user records
  - Formats list display
  - Supports paging

#### COUSR01C - User Add Program
- **Transaction**: CU01
- **BMS Map**: COUSR01
- **Function**: Create new user
- **Files Accessed**: USRSEC (add)
- **Input**: User details
- **Processing**:
  - Validates user ID uniqueness
  - Validates password requirements
  - Creates user record
  - Logs creation

#### COUSR02C - User Update Program
- **Transaction**: CU02
- **BMS Map**: COUSR02
- **Function**: Update user information
- **Files Accessed**: USRSEC (update)
- **Processing**:
  - Retrieves user for update
  - Validates changes
  - Updates user record
  - Supports password reset

#### COUSR03C - User Delete Program
- **Transaction**: CU03
- **BMS Map**: COUSR03
- **Function**: Delete user account
- **Files Accessed**: USRSEC (delete)
- **Processing**:
  - Validates deletion rules
  - Confirms deletion
  - Deletes user record
  - Logs deletion

## Batch Programs

### Transaction Processing Programs

#### CBTRN02C - Transaction Posting Program
- **Job**: POSTTRAN
- **Function**: Post daily transactions to accounts
- **Input Files**: DALYTRAN.PS, ACCTDAT, TRANSACT
- **Output Files**: Updated ACCTDAT, Updated TRANSACT, POSTTRAN.REPORT
- **Processing Logic**:
  1. Read daily transaction file
  2. Validate each transaction
  3. Update account balance
  4. Write to transaction history
  5. Generate posting report
- **Error Handling**: Invalid transactions logged, processing continues

#### CBTRN03C - Transaction Report Program
- **Job**: TRANREPT
- **Function**: Generate transaction reports
- **Input Files**: TRANSACT, ACCTDAT
- **Output Files**: TRANREPT.REPORT
- **Processing**: Aggregates and formats transaction data

### Account Processing Programs

#### CBACT04C - Interest Calculation Program
- **Job**: INTCALC
- **Function**: Calculate and post monthly interest
- **Input Files**: ACCTDAT, TRANSACT
- **Output Files**: Updated ACCTDAT, Updated TRANSACT, INTCALC.REPORT
- **Processing Logic**:
  1. Calculate average daily balance
  2. Apply interest rate
  3. Post interest transaction
  4. Update account balance
- **Interest Formula**: (Average Daily Balance × Annual Rate) / 12

#### CBSTM03A - Statement Generation Program
- **Job**: CREASTMT
- **Function**: Generate monthly statements
- **Input Files**: ACCTDAT, CARDDAT, CUSTDAT, TRANSACT
- **Output Files**: STATEMENT.GDG(+1), CREASTMT.REPORT
- **Processing Logic**:
  1. Read all active accounts
  2. Retrieve transactions for period
  3. Calculate totals
  4. Format statement
  5. Write to output file

## Utility Programs

### Assembler Utilities

#### MVSWAIT - Timer Control Utility
- **Language**: Assembler
- **Function**: Wait for specified time period
- **Usage**: Called from batch jobs to introduce delays
- **Parameters**: Wait time in seconds
- **Purpose**: Coordinate batch job timing

#### COBDATFT - Date Format Conversion Utility
- **Language**: Assembler
- **Function**: Convert date formats
- **Input**: Date in various formats
- **Output**: Standardized date format
- **Usage**: Called from COBOL programs for date conversion

## BMS Maps

### Map Inventory

| Map Name | Program | Transaction | Description |
|:---------|:--------|:------------|:------------|
| COSGN00 | COSGN00C | CC00 | Sign-on screen |
| COMEN01 | COMEN01C | CM00 | Main menu |
| COACTVW | COACTVWC | CAVW | Account view |
| COACTUP | COACTUPC | CAUP | Account update |
| COCRDLI | COCRDLIC | CCLI | Card list |
| COCRDSL | COCRDSLC | CCDL | Card view |
| COCRDUP | COCRDUPC | CCUP | Card update |
| COTRN00 | COTRN00C | CT00 | Transaction list |
| COTRN01 | COTRN01C | CT01 | Transaction view |
| COTRN02 | COTRN02C | CT02 | Transaction add |
| CORPT00 | CORPT00C | CR00 | Reports |
| COBIL00 | COBIL00C | CB00 | Bill payment |
| COADM01 | COADM01C | CA00 | Admin menu |
| COUSR00 | COUSR00C | CU00 | User list |
| COUSR01 | COUSR01C | CU01 | User add |
| COUSR02 | COUSR02C | CU02 | User update |
| COUSR03 | COUSR03C | CU03 | User delete |

### Map Characteristics

**Standard Features:**
- 24x80 character display
- Function key support (PF3, PF4, PF7, PF8, PF12)
- Field attributes (protected, unprotected, numeric)
- Color support (where available)
- Cursor positioning

## Copybooks

### Data Structure Copybooks

#### CSUSR01Y - User Security Record
- **File**: USRSEC
- **Length**: 80 bytes
- **Key**: User ID (8 bytes)
- **Fields**: User ID, Password, User Type, Status, Last Login

#### CVACT01Y - Account Data Record
- **File**: ACCTDAT
- **Length**: 300 bytes
- **Key**: Account ID (11 bytes)
- **Fields**: Account details, balances, credit limits, dates

#### CVACT02Y - Card Data Record
- **File**: CARDDAT
- **Length**: 150 bytes
- **Key**: Card Number (16 bytes)
- **Fields**: Card details, status, expiration, cardholder name

#### CVCUS01Y - Customer Data Record
- **File**: CUSTDAT
- **Length**: 500 bytes
- **Key**: Customer ID (9 bytes)
- **Fields**: Customer personal information, address, contact

#### CVACT03Y - Card Cross Reference Record
- **File**: CARDXREF
- **Length**: 50 bytes
- **Key**: Composite (Account + Card + Customer)
- **Fields**: Account ID, Card Number, Customer ID

#### CVTRA05Y - Transaction Record (Online)
- **File**: TRANSACT
- **Length**: 350 bytes
- **Key**: Transaction ID
- **Fields**: Transaction details, amounts, merchant info, dates

#### CVTRA06Y - Transaction Record (Batch)
- **File**: DALYTRAN
- **Length**: 350 bytes
- **Fields**: Same as CVTRA05Y, used for batch input

### BMS Copybooks

Generated from BMS maps, located in `app/cpy-bms/`:
- Symbolic map definitions
- Field definitions
- Attribute byte definitions

### Working Storage Copybooks

Common working storage structures:
- Date/time formats
- Error messages
- Constants
- Common calculations

## VSAM Files

### File Specifications

#### ACCTDAT - Account Master
- **Organization**: KSDS (Key-Sequenced Data Set)
- **Key**: Account ID (11 bytes, position 1)
- **Record Length**: 300 bytes
- **Access**: Random and sequential
- **Usage**: Online and batch

#### CARDDAT - Card Master
- **Organization**: KSDS
- **Key**: Card Number (16 bytes, position 1)
- **Record Length**: 150 bytes
- **Access**: Random
- **Usage**: Online and batch

#### CUSTDAT - Customer Master
- **Organization**: KSDS
- **Key**: Customer ID (9 bytes, position 1)
- **Record Length**: 500 bytes
- **Access**: Random
- **Usage**: Online and batch

#### CARDXREF - Card Cross Reference
- **Organization**: KSDS
- **Key**: Composite key (36 bytes)
- **Record Length**: 50 bytes
- **Access**: Random and sequential
- **Usage**: Online

#### TRANSACT - Transaction File
- **Organization**: KSDS with AIX
- **Primary Key**: Transaction ID (15 bytes)
- **Alternate Key**: Card Number + Date
- **Record Length**: 350 bytes
- **Access**: Random and sequential
- **Usage**: Online and batch

#### USRSEC - User Security
- **Organization**: KSDS
- **Key**: User ID (8 bytes, position 1)
- **Record Length**: 80 bytes
- **Access**: Random
- **Usage**: Online

### File Attributes

**Buffer Specifications:**
- BUFNI: Number of index buffers
- BUFND: Number of data buffers
- Typical: BUFNI=10, BUFND=20 for online access

**Sharing:**
- SHAREOPTIONS(2,3) for CICS access
- Exclusive access for batch

## CICS Transactions

### Transaction Definitions

| Transaction | Program | Description | Priority |
|:------------|:--------|:------------|:---------|
| CC00 | COSGN00C | Sign-on | High |
| CM00 | COMEN01C | Main menu | High |
| CAVW | COACTVWC | Account view | Normal |
| CAUP | COACTUPC | Account update | Normal |
| CCLI | COCRDLIC | Card list | Normal |
| CCDL | COCRDSLC | Card view | Normal |
| CCUP | COCRDUPC | Card update | Normal |
| CT00 | COTRN00C | Transaction list | Normal |
| CT01 | COTRN01C | Transaction view | Normal |
| CT02 | COTRN02C | Transaction add | Normal |
| CR00 | CORPT00C | Reports | Low |
| CB00 | COBIL00C | Bill payment | Normal |
| CA00 | COADM01C | Admin menu | High |
| CU00 | COUSR00C | User list | Normal |
| CU01 | COUSR01C | User add | Normal |
| CU02 | COUSR02C | User update | Normal |
| CU03 | COUSR03C | User delete | Normal |

### Transaction Attributes

**Standard Attributes:**
- TASKDATALOC(ANY)
- TASKDATAKEY(USER)
- RUNAWAY(SYSTEM)
- SPURGE(YES)
- TPURGE(YES)

## JCL Jobs

### Job Categories

**Initialization Jobs:**
- DUSRSECJ, DEFGDGB, DEFGDGD, CLOSEFIL, OPENFIL

**Data Loading Jobs:**
- ACCTFILE, CARDFILE, CUSTFILE, XREFFILE, TRANFILE, DISCGRP, TCATBALF, TRANCATG, TRANTYPE

**Daily Processing Jobs:**
- TRANBKP, POSTTRAN, COMBTRAN, TRANIDX

**Monthly Processing Jobs:**
- INTCALC, CREASTMT

**Utility Jobs:**
- WAITSTEP, TRANREPT, ESDSRRDS

See [Batch Processing](Batch-Processing.md) for detailed job information.

## Program Specifications

### Naming Conventions

**Online Programs:**
- Prefix: CO (CICS Online)
- Example: COSGN00C, COACTVWC

**Batch Programs:**
- Prefix: CB (COBOL Batch)
- Example: CBTRN02C, CBACT04C

**BMS Maps:**
- Prefix: CO
- Example: COSGN00, COACTVW

**Copybooks:**
- Suffix: Y
- Example: CSUSR01Y, CVACT01Y

### Programming Standards

**COBOL Standards:**
- COBOL 85 or later
- Structured programming
- Proper error handling
- Copybook usage for data structures
- Standard paragraph naming

**CICS Standards:**
- Pseudo-conversational design
- COMMAREA for data passing
- HANDLE CONDITION for error handling
- Proper resource management

**Documentation:**
- Program header with purpose
- Paragraph descriptions
- Complex logic commented
- Change history maintained

---

**Navigation**: [Home](Home.md) | [Architecture](Architecture.md) | [Data Models](Data-Models.md) | [API Reference](API-Reference.md)
