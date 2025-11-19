# Batch Processing Guide

Comprehensive guide to CardDemo batch processing, including job execution, scheduling, and troubleshooting.

## Table of Contents
- [Batch Processing Overview](#batch-processing-overview)
- [Batch Job Catalog](#batch-job-catalog)
- [Daily Processing](#daily-processing)
- [Monthly Processing](#monthly-processing)
- [Job Execution Guide](#job-execution-guide)
- [Job Dependencies](#job-dependencies)
- [Monitoring and Troubleshooting](#monitoring-and-troubleshooting)
- [Performance Optimization](#performance-optimization)

## Batch Processing Overview

CardDemo uses batch processing for:
- Transaction posting and reconciliation
- Interest calculations
- Statement generation
- Data maintenance and backup
- File reorganization
- Report generation

### Batch vs Online Processing

**Online Processing (CICS):**
- Real-time transaction processing
- Interactive user interface
- Immediate response required
- Concurrent file access

**Batch Processing (JCL):**
- Scheduled background processing
- Large volume data processing
- No user interaction
- Exclusive file access

### Batch Window

Batch jobs typically run during off-peak hours when online users are not active.

**Recommended Schedule:**
- **Daily Jobs**: 2:00 AM - 6:00 AM
- **Monthly Jobs**: First Sunday of month, 1:00 AM - 8:00 AM

## Batch Job Catalog

### Initialization Jobs

| Job Name | Program | Purpose | Frequency |
|:---------|:--------|:--------|:----------|
| DUSRSECJ | IEBGENER | Initial load of user security file | One-time / As needed |
| DEFGDGB | IDCAMS | Setup GDG bases | One-time |
| DEFGDGD | IDCAMS | Setup GDG bases for DB2 | One-time (if using DB2) |
| CLOSEFIL | IEFBR14 | Close VSAM files in CICS | Before batch processing |
| OPENFIL | IEFBR14 | Open VSAM files in CICS | After batch processing |

### Data Loading Jobs

| Job Name | Program | Purpose | Frequency |
|:---------|:--------|:--------|:----------|
| ACCTFILE | IDCAMS | Load/refresh Account master | Initial / As needed |
| CARDFILE | IDCAMS | Load/refresh Card master | Initial / As needed |
| CUSTFILE | IDCAMS | Load/refresh Customer master | Initial / As needed |
| XREFFILE | IDCAMS | Load account-card cross reference | Initial / As needed |
| TRANFILE | IDCAMS | Load transaction master file | Initial / As needed |
| DISCGRP | IDCAMS | Load disclosure group file | Initial / As needed |
| TCATBALF | IDCAMS | Load transaction category balance | Initial / As needed |
| TRANCATG | IDCAMS | Load transaction category types | Initial / As needed |
| TRANTYPE | IDCAMS | Load transaction type file | Initial / As needed |

### Daily Processing Jobs

| Job Name | Program | Purpose | Frequency |
|:---------|:--------|:--------|:----------|
| CLOSEFIL | IEFBR14 | Close files for batch | Daily (before batch) |
| TRANBKP | IDCAMS | Backup transaction database | Daily |
| POSTTRAN | CBTRN02C | Post daily transactions | Daily |
| COMBTRAN | SORT | Combine transaction files | Daily |
| TRANIDX | IDCAMS | Rebuild transaction alternate index | Daily |
| OPENFIL | IEFBR14 | Open files for CICS | Daily (after batch) |

### Monthly Processing Jobs

| Job Name | Program | Purpose | Frequency |
|:---------|:--------|:--------|:----------|
| INTCALC | CBACT04C | Calculate interest charges | Monthly |
| CREASTMT | CBSTM03A | Generate account statements | Monthly |

### Optional Module Jobs

| Job Name | Program | Purpose | Module | Frequency |
|:---------|:--------|:--------|:-------|:----------|
| CREADB21 | DSNTEP4 | Create DB2 database and load tables | DB2 Transaction Type | One-time |
| TRANEXTR | DSNTIAUL | Extract DB2 data for transaction types | DB2 Transaction Type | Daily |
| MNTTRDB2 | COBTUPDT | Maintain transaction type table | DB2 Transaction Type | As needed |
| CBPAUP0J | CBPAUP0C | Purge expired authorizations | IMS-DB2-MQ Auth | Daily |

### Utility Jobs

| Job Name | Program | Purpose | Frequency |
|:---------|:--------|:--------|:----------|
| WAITSTEP | COBSWAIT | Wait for specified time | As needed |
| TRANREPT | CBTRN03C | Generate transaction report | On demand |
| ESDSRRDS | IDCAMS | Create ESDS and RRDS VSAM files | As needed |

## Daily Processing

### Daily Batch Sequence

Execute jobs in this order for daily processing:

```
1. CLOSEFIL   - Close files (2:00 AM)
2. TRANBKP    - Backup transactions (2:05 AM)
3. POSTTRAN   - Post transactions (2:15 AM)
4. COMBTRAN   - Combine transactions (3:00 AM)
5. TRANIDX    - Rebuild index (3:30 AM)
6. TRANEXTR   - Extract DB2 data (4:00 AM) [if using DB2 module]
7. TRANCATG   - Update transaction categories (4:15 AM)
8. TRANTYPE   - Update transaction types (4:20 AM)
9. CBPAUP0J   - Purge authorizations (4:30 AM) [if using Auth module]
10. OPENFIL   - Open files (5:00 AM)
```

### Job Details

#### CLOSEFIL - Close Files

**Purpose**: Closes all VSAM files to allow exclusive batch access.

**JCL:**
```jcl
//CLOSEFIL JOB ...
//STEP1    EXEC PGM=IEFBR14
//ACCTDAT  DD DSN=AWS.M2.CARDDEMO.ACCTDAT.VSAM.KSDS,
//            DISP=OLD
//CARDDAT  DD DSN=AWS.M2.CARDDEMO.CARDDAT.VSAM.KSDS,
//            DISP=OLD
// ... (all VSAM files)
```

**Expected RC**: 0

**Notes**: Must complete before any other batch jobs access VSAM files.

#### TRANBKP - Backup Transactions

**Purpose**: Creates backup copy of transaction file.

**Program**: IDCAMS

**Input**: TRANSACT.VSAM.KSDS

**Output**: TRANSACT.BACKUP.GDG(+1)

**Expected RC**: 0

**Notes**: Backup is kept for 7 generations.

#### POSTTRAN - Post Transactions

**Purpose**: Posts daily transactions to account balances.

**Program**: CBTRN02C (COBOL)

**Input Files:**
- DALYTRAN.PS (daily transactions)
- ACCTDAT.VSAM.KSDS (account master)
- TRANSACT.VSAM.KSDS (transaction file)

**Output Files:**
- Updated ACCTDAT.VSAM.KSDS
- Updated TRANSACT.VSAM.KSDS
- POSTTRAN.REPORT (posting report)

**Processing Logic:**
1. Read daily transaction file
2. Validate each transaction
3. Update account balance
4. Write to transaction history
5. Generate posting report

**Expected RC**: 0

**Error Handling:**
- Invalid transactions written to error file
- Processing continues for valid transactions
- Summary report shows counts and totals

#### COMBTRAN - Combine Transactions

**Purpose**: Combines and sorts transaction files.

**Program**: SORT

**Input Files:**
- TRANSACT.VSAM.KSDS
- DALYTRAN.PS

**Output**: Sorted combined transaction file

**Sort Keys:**
- Primary: Card Number
- Secondary: Transaction Date (descending)

**Expected RC**: 0

#### TRANIDX - Rebuild Transaction Index

**Purpose**: Rebuilds alternate index on transaction file.

**Program**: IDCAMS

**Commands:**
```
DELETE AWS.M2.CARDDEMO.TRANSACT.AIX
DEFINE AIX (...)
BLDINDEX INDATASET(AWS.M2.CARDDEMO.TRANSACT.VSAM.KSDS) ...
```

**Expected RC**: 0

**Notes**: Improves query performance for card number searches.

#### OPENFIL - Open Files

**Purpose**: Reopens VSAM files for CICS access.

**JCL:**
```jcl
//OPENFIL  JOB ...
//STEP1    EXEC PGM=IEFBR14
//ACCTDAT  DD DSN=AWS.M2.CARDDEMO.ACCTDAT.VSAM.KSDS,
//            DISP=SHR
// ... (all VSAM files)
```

**Expected RC**: 0

**Notes**: Must complete before CICS users can access files.

## Monthly Processing

### Monthly Batch Sequence

Execute these jobs in addition to daily jobs on the first of each month:

```
1. CLOSEFIL   - Close files
2. TRANBKP    - Backup transactions
3. POSTTRAN   - Post transactions
4. INTCALC    - Calculate interest (NEW)
5. CREASTMT   - Generate statements (NEW)
6. COMBTRAN   - Combine transactions
7. TRANIDX    - Rebuild index
8. OPENFIL    - Open files
```

### Job Details

#### INTCALC - Interest Calculation

**Purpose**: Calculates and posts monthly interest charges.

**Program**: CBACT04C (COBOL)

**Input Files:**
- ACCTDAT.VSAM.KSDS (account master)
- TRANSACT.VSAM.KSDS (transaction file)

**Output Files:**
- Updated ACCTDAT.VSAM.KSDS (with interest charges)
- Updated TRANSACT.VSAM.KSDS (interest transactions)
- INTCALC.REPORT (interest calculation report)

**Processing Logic:**
1. Read all active accounts
2. Calculate average daily balance
3. Apply interest rate based on account type
4. Post interest transaction
5. Update account balance
6. Generate summary report

**Interest Calculation:**
```
Interest = (Average Daily Balance × Annual Rate) / 12
```

**Expected RC**: 0

**Report Contents:**
- Account count processed
- Total interest charged
- Account-by-account detail

#### CREASTMT - Create Statements

**Purpose**: Generates monthly account statements.

**Program**: CBSTM03A (COBOL)

**Input Files:**
- ACCTDAT.VSAM.KSDS (account master)
- CARDDAT.VSAM.KSDS (card master)
- CUSTDAT.VSAM.KSDS (customer master)
- TRANSACT.VSAM.KSDS (transaction file)

**Output Files:**
- STATEMENT.GDG(+1) (statement file)
- CREASTMT.REPORT (generation report)

**Processing Logic:**
1. Read all active accounts
2. Retrieve customer information
3. Retrieve all transactions for billing period
4. Calculate totals and balances
5. Format statement
6. Write to statement file

**Statement Contents:**
- Account summary
- Previous balance
- Payments and credits
- Purchases and debits
- Interest charges
- New balance
- Minimum payment due
- Payment due date
- Transaction detail

**Expected RC**: 0

## Job Execution Guide

### Submitting Jobs

**Using TSO/ISPF:**
```
1. TSO ISPF 3.4
2. Navigate to AWS.M2.CARDDEMO.JCL
3. Select job member
4. Enter 'S' (Submit)
5. Job submitted to JES
```

**Using JCL:**
```
//SUBMIT   EXEC PGM=IEBGENER
//SYSUT1   DD DSN=AWS.M2.CARDDEMO.JCL(POSTTRAN),DISP=SHR
//SYSUT2   DD SYSOUT=(A,INTRDR)
//SYSPRINT DD SYSOUT=*
```

### Monitoring Job Execution

**Check Job Status:**
```
1. Enter SDSF or equivalent
2. Filter by job name
3. Check status (ACTIVE, OUTPUT, ABEND)
4. View return code
```

**View Job Output:**
```
1. Select job in SDSF
2. Enter 'S' to view output
3. Review SYSOUT datasets
4. Check for error messages
```

### Verifying Job Completion

**Success Indicators:**
- Return Code: 0000
- No error messages in SYSOUT
- Expected output files created
- Record counts match expectations

**Check Return Codes:**
```
RC=0000  - Successful completion
RC=0004  - Warning (review output)
RC=0008  - Error (job failed)
RC=0012  - Severe error
RC=Sxxx  - System abend
RC=Uxxx  - User abend
```

## Job Dependencies

### Dependency Chart

```
CLOSEFIL
    ↓
TRANBKP
    ↓
POSTTRAN ──→ COMBTRAN ──→ TRANIDX
    ↓            ↓
INTCALC      TRANEXTR
    ↓            ↓
CREASTMT     TRANCATG
                 ↓
             TRANTYPE
                 ↓
             CBPAUP0J
                 ↓
             OPENFIL
```

### Critical Dependencies

1. **CLOSEFIL must complete** before any file access
2. **TRANBKP must complete** before POSTTRAN
3. **POSTTRAN must complete** before INTCALC
4. **INTCALC must complete** before CREASTMT
5. **All processing must complete** before OPENFIL

### Parallel Execution

These jobs can run in parallel (after POSTTRAN):
- COMBTRAN
- TRANEXTR
- INTCALC (if not dependent on COMBTRAN)

## Monitoring and Troubleshooting

### Common Issues

**Issue: Job Abends with S806**
- **Cause**: Program not found
- **Solution**: Verify load library in STEPLIB

**Issue: Job Abends with S013**
- **Cause**: File not found or not cataloged
- **Solution**: Verify dataset exists and is cataloged

**Issue: Job Abends with S213**
- **Cause**: File already open or in use
- **Solution**: Verify CLOSEFIL completed, check for CICS file locks

**Issue: Job Abends with S222**
- **Cause**: Operator cancel
- **Solution**: Check with operations, resubmit if appropriate

**Issue: Job Abends with S322**
- **Cause**: CPU time limit exceeded
- **Solution**: Increase TIME parameter in JCL

**Issue: Job Abends with SB37**
- **Cause**: Out of space
- **Solution**: Increase SPACE allocation or delete old datasets

### Performance Issues

**Slow Transaction Posting:**
- Check VSAM file CI/CA splits
- Consider file reorganization
- Review buffer allocations

**Long-Running Jobs:**
- Review program logic
- Check for unnecessary I/O
- Consider parallel processing

### Recovery Procedures

**If POSTTRAN Fails:**
1. Review error messages
2. Correct data issues
3. Restore from TRANBKP if needed
4. Rerun POSTTRAN
5. Continue with remaining jobs

**If INTCALC Fails:**
1. Review calculation errors
2. Correct account data
3. Rerun INTCALC
4. Verify balances before CREASTMT

**If OPENFIL Fails:**
1. Check file status
2. Manually open files via CEMT
3. Investigate root cause

## Performance Optimization

### Best Practices

1. **Schedule Appropriately**: Run during off-peak hours
2. **Monitor Resources**: Track CPU, I/O, and elapsed time
3. **Optimize File Access**: Use appropriate buffer sizes
4. **Reorganize Files**: Periodically reorganize VSAM files
5. **Parallel Processing**: Run independent jobs in parallel
6. **Archive Old Data**: Move historical data to archive files

### Tuning Parameters

**VSAM Buffers:**
```
//ACCTDAT  DD DSN=...,DISP=SHR,
//            AMP=('BUFNI=10,BUFND=20')
```

**Sort Work Files:**
```
//SORTWK01 DD UNIT=SYSDA,SPACE=(CYL,(50,10))
//SORTWK02 DD UNIT=SYSDA,SPACE=(CYL,(50,10))
//SORTWK03 DD UNIT=SYSDA,SPACE=(CYL,(50,10))
```

**Region Size:**
```
//JOBNAME  JOB ...,REGION=32M
```

---

**Navigation**: [Home](Home.md) | [Installation Guide](Installation-Guide.md) | [Technical Components](Technical-Components.md) | [Troubleshooting](Troubleshooting.md)
