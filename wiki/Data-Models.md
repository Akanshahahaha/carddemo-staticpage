# Data Models

Comprehensive documentation of all data structures, file layouts, and relationships in CardDemo.

## Table of Contents
- [Data Model Overview](#data-model-overview)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [VSAM File Structures](#vsam-file-structures)
- [Record Layouts](#record-layouts)
- [Data Relationships](#data-relationships)
- [Data Validation Rules](#data-validation-rules)
- [Sample Data](#sample-data)

## Data Model Overview

CardDemo uses a relational-style data model implemented in VSAM files. The model supports:
- Customer information management
- Account and card relationships
- Transaction processing and history
- User security and access control
- Reference data for transaction types and categories

### Data Storage Technologies

**Primary Storage (VSAM):**
- Customer Master (CUSTDAT)
- Account Master (ACCTDAT)
- Card Master (CARDDAT)
- Card Cross Reference (CARDXREF)
- Transaction File (TRANSACT)
- User Security (USRSEC)
- Reference Files (TRANTYPE, TRANCATG, DISCGRP, TCATBALF)

**Optional Storage:**
- DB2: Transaction types, fraud tracking
- IMS DB: Authorization details
- MQ: Message queuing

## Entity Relationship Diagram

```
┌─────────────┐
│  CUSTOMER   │
│ (CUSTDAT)   │
└──────┬──────┘
       │ 1
       │
       │ N
┌──────┴──────┐
│   ACCOUNT   │
│  (ACCTDAT)  │
└──────┬──────┘
       │ 1
       │
       │ N
┌──────┴──────┐         ┌─────────────┐
│    CARD     │────N────│ TRANSACTION │
│ (CARDDAT)   │    1    │ (TRANSACT)  │
└─────────────┘         └─────────────┘

┌─────────────┐
│  CARDXREF   │  (Cross Reference)
│ Account-Card│
│  -Customer  │
└─────────────┘

┌─────────────┐
│   USRSEC    │  (User Security)
└─────────────┘

┌─────────────┐
│  TRANTYPE   │  (Transaction Types)
└─────────────┘

┌─────────────┐
│  TRANCATG   │  (Transaction Categories)
└─────────────┘
```

### Cardinality

- One CUSTOMER can have many ACCOUNTs (1:N)
- One ACCOUNT can have many CARDs (1:N)
- One CARD can have many TRANSACTIONs (1:N)
- CARDXREF maintains the many-to-many relationships

## VSAM File Structures

### CUSTDAT - Customer Master

**File Organization:** KSDS (Key-Sequenced Data Set)

**Key Structure:**
- Primary Key: Customer ID (9 bytes, position 1-9)
- Key Type: Numeric
- Uniqueness: Unique

**File Attributes:**
- Record Length: 500 bytes (Fixed)
- CI Size: 4096 bytes
- CA Size: 1 cylinder
- SHAREOPTIONS: (2,3)

**Copybook:** CVCUS01Y

**Record Layout:**
```
01  CUSTOMER-RECORD.
    05  CUST-ID                    PIC 9(09).
    05  CUST-FIRST-NAME            PIC X(25).
    05  CUST-MIDDLE-NAME           PIC X(25).
    05  CUST-LAST-NAME             PIC X(25).
    05  CUST-ADDR-LINE-1           PIC X(50).
    05  CUST-ADDR-LINE-2           PIC X(50).
    05  CUST-ADDR-LINE-3           PIC X(50).
    05  CUST-ADDR-STATE-CD         PIC X(02).
    05  CUST-ADDR-COUNTRY-CD       PIC X(03).
    05  CUST-ADDR-ZIP              PIC X(10).
    05  CUST-PHONE-NUM-1           PIC X(15).
    05  CUST-PHONE-NUM-2           PIC X(15).
    05  CUST-SSN                   PIC 9(09).
    05  CUST-GOVT-ISSUED-ID        PIC X(20).
    05  CUST-DOB-YYYY-MM-DD        PIC X(10).
    05  CUST-EFT-ACCOUNT-ID        PIC X(10).
    05  CUST-PRI-CARD-HOLDER-IND   PIC X(01).
    05  CUST-FICO-CREDIT-SCORE     PIC 9(03).
    05  FILLER                     PIC X(168).
```

### ACCTDAT - Account Master

**File Organization:** KSDS

**Key Structure:**
- Primary Key: Account ID (11 bytes, position 1-11)
- Key Type: Numeric
- Uniqueness: Unique

**File Attributes:**
- Record Length: 300 bytes (Fixed)
- CI Size: 4096 bytes
- SHAREOPTIONS: (2,3)

**Copybook:** CVACT01Y

**Record Layout:**
```
01  ACCOUNT-RECORD.
    05  ACCT-ID                    PIC 9(11).
    05  ACCT-ACTIVE-STATUS         PIC X(01).
        88  ACCT-ACTIVE            VALUE 'Y'.
        88  ACCT-INACTIVE          VALUE 'N'.
    05  ACCT-CURR-BAL              PIC S9(10)V99 COMP-3.
    05  ACCT-CREDIT-LIMIT          PIC S9(10)V99 COMP-3.
    05  ACCT-CASH-CREDIT-LIMIT     PIC S9(10)V99 COMP-3.
    05  ACCT-OPEN-DATE             PIC X(10).
    05  ACCT-EXPIRATION-DATE       PIC X(10).
    05  ACCT-REISSUE-DATE          PIC X(10).
    05  ACCT-CURR-CYC-CREDIT       PIC S9(10)V99 COMP-3.
    05  ACCT-CURR-CYC-DEBIT        PIC S9(10)V99 COMP-3.
    05  ACCT-ADDR-ZIP              PIC X(10).
    05  ACCT-GROUP-ID              PIC X(10).
    05  FILLER                     PIC X(178).
```

### CARDDAT - Card Master

**File Organization:** KSDS

**Key Structure:**
- Primary Key: Card Number (16 bytes, position 1-16)
- Key Type: Numeric
- Uniqueness: Unique

**File Attributes:**
- Record Length: 150 bytes (Fixed)
- CI Size: 4096 bytes
- SHAREOPTIONS: (2,3)

**Copybook:** CVACT02Y

**Record Layout:**
```
01  CARD-RECORD.
    05  CARD-NUM                   PIC 9(16).
    05  CARD-ACCT-ID               PIC 9(11).
    05  CARD-CVV-CD                PIC 9(03).
    05  CARD-EMBOSSED-NAME         PIC X(50).
    05  CARD-EXPIRATION-DATE       PIC X(10).
    05  CARD-ACTIVE-STATUS         PIC X(01).
        88  CARD-ACTIVE            VALUE 'Y'.
        88  CARD-INACTIVE          VALUE 'N'.
    05  FILLER                     PIC X(59).
```

### CARDXREF - Card Cross Reference

**File Organization:** KSDS

**Key Structure:**
- Primary Key: Composite (36 bytes)
  - Account ID (11 bytes)
  - Card Number (16 bytes)
  - Customer ID (9 bytes)
- Uniqueness: Unique

**File Attributes:**
- Record Length: 50 bytes (Fixed)
- CI Size: 4096 bytes
- SHAREOPTIONS: (2,3)

**Copybook:** CVACT03Y

**Record Layout:**
```
01  CARD-XREF-RECORD.
    05  XREF-ACCT-ID               PIC 9(11).
    05  XREF-CARD-NUM              PIC 9(16).
    05  XREF-CUST-ID               PIC 9(09).
    05  FILLER                     PIC X(14).
```

### TRANSACT - Transaction File

**File Organization:** KSDS with AIX (Alternate Index)

**Primary Key:**
- Transaction ID (15 bytes, position 1-15)
- Key Type: Alphanumeric
- Uniqueness: Unique

**Alternate Index:**
- Card Number (16 bytes) + Transaction Date (10 bytes)
- Allows efficient retrieval by card number

**File Attributes:**
- Record Length: 350 bytes (Fixed)
- CI Size: 8192 bytes
- SHAREOPTIONS: (2,3)

**Copybook:** CVTRA05Y (Online), CVTRA06Y (Batch)

**Record Layout:**
```
01  TRANSACTION-RECORD.
    05  TRAN-ID                    PIC X(16).
    05  TRAN-TYPE-CD               PIC X(02).
    05  TRAN-CAT-CD                PIC 9(04).
    05  TRAN-SOURCE                PIC X(10).
    05  TRAN-DESC                  PIC X(100).
    05  TRAN-AMT                   PIC S9(09)V99 COMP-3.
    05  TRAN-CARD-NUM              PIC 9(16).
    05  TRAN-MERCHANT-ID           PIC 9(09).
    05  TRAN-MERCHANT-NAME         PIC X(50).
    05  TRAN-MERCHANT-CITY         PIC X(50).
    05  TRAN-MERCHANT-ZIP          PIC X(10).
    05  TRAN-ORIG-TS               PIC X(26).
    05  TRAN-PROC-TS               PIC X(26).
    05  FILLER                     PIC X(??).
```

### USRSEC - User Security

**File Organization:** KSDS

**Key Structure:**
- Primary Key: User ID (8 bytes, position 1-8)
- Key Type: Alphanumeric
- Uniqueness: Unique

**File Attributes:**
- Record Length: 80 bytes (Fixed)
- CI Size: 4096 bytes
- SHAREOPTIONS: (2,3)

**Copybook:** CSUSR01Y

**Record Layout:**
```
01  USER-SECURITY-RECORD.
    05  SEC-USR-ID                 PIC X(08).
    05  SEC-USR-FNAME              PIC X(20).
    05  SEC-USR-LNAME              PIC X(20).
    05  SEC-USR-PWD                PIC X(08).
    05  SEC-USR-TYPE               PIC X(01).
        88  SEC-USR-TYPE-ADMIN     VALUE 'A'.
        88  SEC-USR-TYPE-USER      VALUE 'U'.
    05  SEC-USR-FILLER             PIC X(23).
```

## Record Layouts

### Data Types Used

**COBOL Data Types:**
- **PIC 9(n)**: Numeric (display)
- **PIC X(n)**: Alphanumeric
- **PIC S9(n)V99**: Signed numeric with 2 decimal places
- **COMP-3**: Packed decimal (efficient storage)
- **COMP**: Binary

**Date Formats:**
- Standard: YYYY-MM-DD (10 bytes)
- Timestamp: YYYY-MM-DD-HH.MM.SS.NNNNNN (26 bytes)

**Amount Formats:**
- Currency: S9(10)V99 COMP-3 (up to $99,999,999.99)
- Percentages: S9(03)V99 (up to 999.99%)

### Field Naming Conventions

**Prefixes:**
- CUST-: Customer fields
- ACCT-: Account fields
- CARD-: Card fields
- TRAN-: Transaction fields
- SEC-: Security fields
- XREF-: Cross-reference fields

**Suffixes:**
- -ID: Identifier
- -NUM: Number
- -CD: Code
- -IND: Indicator
- -TS: Timestamp
- -AMT: Amount
- -DATE: Date field

## Data Relationships

### Customer to Account

**Relationship Type:** One-to-Many

**Implementation:**
- Customer ID stored in CARDXREF
- Account ID stored in CARDXREF
- Join via CARDXREF to link customers to accounts

**Business Rules:**
- One customer can have multiple accounts
- Each account belongs to one primary customer
- Joint accounts supported via multiple CARDXREF records

### Account to Card

**Relationship Type:** One-to-Many

**Implementation:**
- Account ID stored in CARD-ACCT-ID field
- Direct relationship via account ID

**Business Rules:**
- One account can have multiple cards
- Each card belongs to exactly one account
- Card number is globally unique

### Card to Transaction

**Relationship Type:** One-to-Many

**Implementation:**
- Card Number stored in TRAN-CARD-NUM field
- Alternate index on card number for efficient access

**Business Rules:**
- One card can have many transactions
- Each transaction belongs to exactly one card
- Transaction ID is globally unique

### Cross-Reference Relationships

**CARDXREF Purpose:**
- Links Account, Card, and Customer
- Supports multiple cards per account
- Supports multiple customers per account (joint accounts)

**Query Patterns:**
- Find all cards for an account
- Find all accounts for a customer
- Find customer for a card

## Data Validation Rules

### Customer Data

**Customer ID:**
- Format: 9 digits
- Range: 000000001 - 999999999
- Uniqueness: Must be unique

**Name Fields:**
- First Name: 1-25 characters, required
- Last Name: 1-25 characters, required
- Middle Name: 0-25 characters, optional

**Address:**
- Address lines: Up to 50 characters each
- State: 2-character code
- Country: 3-character code
- ZIP: Up to 10 characters

**SSN:**
- Format: 9 digits
- Uniqueness: Should be unique (not enforced)

**FICO Score:**
- Range: 300-850
- Default: 0 (not scored)

### Account Data

**Account ID:**
- Format: 11 digits
- Range: 00000000001 - 99999999999
- Uniqueness: Must be unique

**Balances:**
- Current Balance: -$99,999,999.99 to $99,999,999.99
- Credit Limit: $0.00 to $99,999,999.99
- Cash Credit Limit: $0.00 to Credit Limit

**Status:**
- Values: 'Y' (Active), 'N' (Inactive)
- Default: 'Y'

**Dates:**
- Open Date: Required, format YYYY-MM-DD
- Expiration Date: Required, must be > Open Date
- Reissue Date: Optional

### Card Data

**Card Number:**
- Format: 16 digits
- Validation: Luhn algorithm (checksum)
- Uniqueness: Must be unique

**CVV:**
- Format: 3 digits
- Range: 000-999

**Expiration Date:**
- Format: MM/YY or YYYY-MM-DD
- Must be future date

**Status:**
- Values: 'Y' (Active), 'N' (Inactive), 'B' (Blocked), 'E' (Expired)
- Default: 'Y'

### Transaction Data

**Transaction ID:**
- Format: 16 characters alphanumeric
- Uniqueness: Must be unique
- Generation: System-generated

**Transaction Type:**
- Format: 2 characters
- Valid values: PUR, PAY, ADJ, FEE, INT, etc.
- Must exist in TRANTYPE file

**Amount:**
- Range: -$999,999.99 to $999,999.99
- Precision: 2 decimal places
- Positive: Charges (purchases, fees)
- Negative: Credits (payments, adjustments)

**Merchant Information:**
- Merchant ID: 9 digits
- Merchant Name: Up to 50 characters
- Merchant City: Up to 50 characters
- Merchant ZIP: Up to 10 characters

## Sample Data

### Sample Customer

```
Customer ID: 000000001
First Name: John
Middle Name: Michael
Last Name: Doe
Address Line 1: 123 Main Street
Address Line 2: Apt 4B
City: New York
State: NY
ZIP: 10001
Phone: 212-555-1234
SSN: 123456789
DOB: 1980-05-15
FICO Score: 720
```

### Sample Account

```
Account ID: 00000000001
Status: Active (Y)
Current Balance: $1,234.56
Credit Limit: $5,000.00
Cash Credit Limit: $1,000.00
Open Date: 2020-01-15
Expiration Date: 2025-01-15
Current Cycle Credit: $0.00
Current Cycle Debit: $234.56
```

### Sample Card

```
Card Number: 4000123456789010
Account ID: 00000000001
CVV: 123
Embossed Name: JOHN M DOE
Expiration Date: 2025-12-31
Status: Active (Y)
```

### Sample Transaction

```
Transaction ID: T000000000000001
Type: PUR (Purchase)
Category: 5411 (Grocery)
Description: GROCERY STORE PURCHASE
Amount: $45.67
Card Number: 4000123456789010
Merchant ID: 123456789
Merchant Name: ABC Grocery Store
Merchant City: New York
Merchant ZIP: 10001
Transaction Date: 2025-11-15
Processing Date: 2025-11-15
```

### Sample User

```
User ID: USER0001
First Name: John
Last Name: Doe
Password: ******** (encrypted)
User Type: U (User)
Status: Active
Last Login: 2025-11-15
```

## Data Volume Estimates

**Typical Production Volumes:**
- Customers: 100,000 - 1,000,000
- Accounts: 150,000 - 1,500,000
- Cards: 200,000 - 2,000,000
- Transactions: 1,000,000 - 10,000,000 per month
- Users: 100 - 1,000

**Storage Requirements:**
- CUSTDAT: ~500 MB per 1M customers
- ACCTDAT: ~300 MB per 1M accounts
- CARDDAT: ~150 MB per 1M cards
- TRANSACT: ~3.5 GB per 10M transactions

---

**Navigation**: [Home](Home.md) | [Architecture](Architecture.md) | [Technical Components](Technical-Components.md) | [API Reference](API-Reference.md)
