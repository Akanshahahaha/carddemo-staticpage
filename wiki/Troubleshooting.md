# Troubleshooting Guide

Comprehensive troubleshooting guide for CardDemo issues, errors, and common problems.

## Table of Contents
- [General Troubleshooting Approach](#general-troubleshooting-approach)
- [Installation Issues](#installation-issues)
- [CICS Issues](#cics-issues)
- [VSAM File Issues](#vsam-file-issues)
- [Batch Job Issues](#batch-job-issues)
- [User Access Issues](#user-access-issues)
- [Transaction Issues](#transaction-issues)
- [Optional Module Issues](#optional-module-issues)
- [Performance Issues](#performance-issues)
- [Error Code Reference](#error-code-reference)

## General Troubleshooting Approach

### Systematic Debugging Process

1. **Identify the Problem**
   - What is the error message?
   - When does it occur?
   - Can you reproduce it?
   - What changed recently?

2. **Gather Information**
   - Check CICS logs
   - Review job output
   - Check file status
   - Review system messages

3. **Analyze the Data**
   - Look for patterns
   - Check timestamps
   - Review related components
   - Verify configurations

4. **Implement Solution**
   - Test in development first
   - Document the fix
   - Verify resolution
   - Monitor for recurrence

5. **Prevent Recurrence**
   - Update documentation
   - Implement monitoring
   - Train team members
   - Review procedures

### Key Log Locations

**CICS Logs:**
- CICS message log (MSGUSR)
- CICS transaction dump
- CICS trace

**Batch Logs:**
- JES job output (SDSF)
- SYSOUT datasets
- Job return codes

**System Logs:**
- z/OS system log
- VSAM error messages
- DB2 logs (if applicable)
- IMS logs (if applicable)

## Installation Issues

### Issue: Dataset Creation Fails

**Symptoms:**
- Cannot allocate dataset
- Space allocation error
- Catalog error

**Possible Causes:**
- Insufficient DASD space
- Invalid dataset name
- Catalog full
- Duplicate dataset name

**Solutions:**
```
1. Check available space:
   LISTCAT SPACE

2. Verify dataset name format:
   - Must follow naming conventions
   - Check HLQ permissions

3. Delete existing dataset if duplicate:
   DELETE 'AWS.M2.CARDDEMO.JCL'

4. Increase space allocation:
   SPACE=(CYL,(20,10)) instead of (10,5)
```

### Issue: File Transfer Fails

**Symptoms:**
- Files not uploaded
- Incorrect record format
- Data corruption

**Possible Causes:**
- Wrong transfer mode (binary vs text)
- Incorrect LRECL
- EBCDIC conversion issues
- Network problems

**Solutions:**
```
1. Use correct transfer mode:
   - Text mode for source code
   - Binary mode for data files

2. Verify LRECL matches:
   quote site recfm=fb lrecl=80

3. Check file contents after transfer:
   Browse dataset in ISPF

4. Retry transfer if network issue
```

### Issue: JCL Job Fails During Initialization

**Symptoms:**
- DUSRSECJ fails
- ACCTFILE fails
- VSAM file not created

**Possible Causes:**
- Incorrect dataset names in JCL
- VSAM catalog issues
- Insufficient permissions
- Missing input data

**Solutions:**
```
1. Update HLQ in JCL:
   Edit all occurrences of AWS.M2 to your HLQ

2. Check VSAM catalog:
   LISTCAT ENTRIES('AWS.M2.CARDDEMO.*')

3. Verify input data exists:
   Check AWS.M2.CARDDEMO.USRSEC.PS

4. Review job output for specific error
```

### Issue: Program Compilation Fails

**Symptoms:**
- Compilation errors
- Missing copybooks
- Link edit fails

**Possible Causes:**
- Copybook not found
- Syntax errors in source
- Missing libraries
- Incorrect compiler options

**Solutions:**
```
1. Verify SYSLIB includes copybook library:
   //SYSLIB DD DSN=AWS.M2.CARDDEMO.CPY,DISP=SHR

2. Check for syntax errors:
   Review compiler output

3. Verify all copybooks exist:
   Check CPY dataset

4. Check compiler options:
   PARM='LIB,OBJECT,MAP'
```

## CICS Issues

### Issue: Transaction ABEND AFCF

**Symptoms:**
- Transaction fails with AFCF
- File not found error

**Possible Causes:**
- File not defined in CICS
- File not open
- Incorrect file name
- File not cataloged

**Solutions:**
```
1. Check file definition:
   CEMT INQUIRE FILE(ACCTDAT)

2. Open file if closed:
   CEMT SET FILE(ACCTDAT) OPEN

3. Verify file name in program matches CICS definition

4. Check VSAM file is cataloged:
   LISTCAT ENT('AWS.M2.CARDDEMO.ACCTDAT.VSAM.KSDS')
```

### Issue: Transaction ABEND ASRA

**Symptoms:**
- Program check
- Data exception
- Protection exception

**Possible Causes:**
- Invalid data in field
- Array bounds exceeded
- Null pointer reference
- Division by zero

**Solutions:**
```
1. Review transaction dump:
   CEMT INQUIRE TASK
   Look for PSW and registers

2. Check data values:
   Review COMMAREA contents

3. Verify array subscripts:
   Check OCCURS DEPENDING ON

4. Add data validation:
   IF field IS NUMERIC
```

### Issue: Transaction ABEND AEIV

**Symptoms:**
- Invalid request
- CICS command error

**Possible Causes:**
- Invalid CICS command
- Wrong command syntax
- Resource not available
- Invalid parameter

**Solutions:**
```
1. Review CICS command:
   Check EXEC CICS syntax

2. Verify resource exists:
   CEMT INQ FILE/PROG/TRAN

3. Check RESP code:
   Add RESP(WS-RESP) to command

4. Review CICS manual for correct syntax
```

### Issue: Cannot Access Transaction

**Symptoms:**
- Transaction not found
- TRANSIDERR

**Possible Causes:**
- Transaction not defined
- Transaction not installed
- Transaction disabled
- Security restriction

**Solutions:**
```
1. Check transaction definition:
   CEMT INQUIRE TRANSACTION(CC00)

2. Install transaction if not installed:
   CEDA INSTALL TRANSACTION(CC00) GROUP(CARDDEMO)

3. Enable transaction if disabled:
   CEMT SET TRANSACTION(CC00) ENABLED

4. Check RACF permissions
```

### Issue: Program Not Found (PGMIDERR)

**Symptoms:**
- Program not found error
- ABEND APCT

**Possible Causes:**
- Program not defined
- Program not in load library
- NEWCOPY needed
- Wrong program name

**Solutions:**
```
1. Check program definition:
   CEMT INQUIRE PROGRAM(COSGN00C)

2. Verify program in load library:
   Check DFHRPL concatenation

3. Execute NEWCOPY:
   CEMT SET PROGRAM(COSGN00C) NEWCOPY

4. Verify program name spelling
```

## VSAM File Issues

### Issue: File Not Found (S013)

**Symptoms:**
- JCL fails with S013
- File not cataloged

**Possible Causes:**
- File not created
- File deleted
- Wrong dataset name
- Catalog issue

**Solutions:**
```
1. Check if file exists:
   LISTCAT ENT('AWS.M2.CARDDEMO.ACCTDAT.VSAM.KSDS')

2. Create file if missing:
   Submit ACCTFILE job

3. Verify dataset name in JCL

4. Recatalog if needed:
   DEFINE NONVSAM (...)
```

### Issue: File In Use (S213)

**Symptoms:**
- Cannot open file
- File already open
- Enqueue failure

**Possible Causes:**
- File open in CICS
- Batch job has file open
- Previous job didn't close file
- VSAM lock

**Solutions:**
```
1. Check CICS file status:
   CEMT INQUIRE FILE(ACCTDAT)

2. Close file in CICS:
   CEMT SET FILE(ACCTDAT) CLOSED

3. Check for batch jobs:
   Display active jobs in SDSF

4. Force close if necessary:
   IDCAMS VERIFY
```

### Issue: Out of Space (SB37, SD37, SE37)

**Symptoms:**
- File full
- Cannot extend
- Space allocation error

**Possible Causes:**
- Primary space exhausted
- Secondary extents exhausted
- Volume full
- No space on volume

**Solutions:**
```
1. Check space usage:
   LISTCAT ENT('file.name') ALL

2. Delete and redefine with more space:
   DELETE file.name
   DEFINE CLUSTER (... CYL(50,10))

3. Move to larger volume

4. Compress file if KSDS:
   REPRO INFILE(...) OUTFILE(...)
```

### Issue: VSAM Logic Error

**Symptoms:**
- Invalid request
- Sequence error
- Duplicate key

**Possible Causes:**
- Writing out of sequence
- Duplicate key insert
- Invalid key
- File not properly opened

**Solutions:**
```
1. Check key sequence:
   Verify records are in order

2. Check for duplicate keys:
   Verify key uniqueness

3. Verify file opened correctly:
   Check OPEN mode (INPUT/OUTPUT/I-O)

4. Review program logic
```

## Batch Job Issues

### Issue: Job Fails with S806

**Symptoms:**
- Program not found
- Load module missing

**Possible Causes:**
- Program not compiled
- Wrong load library
- Program name misspelled
- STEPLIB missing

**Solutions:**
```
1. Verify program exists:
   Check load library for member

2. Add STEPLIB if missing:
   //STEPLIB DD DSN=AWS.M2.CARDDEMO.LOADLIB,DISP=SHR

3. Recompile program if missing

4. Check program name spelling in JCL
```

### Issue: Job Fails with S0C7

**Symptoms:**
- Data exception
- Invalid numeric data

**Possible Causes:**
- Non-numeric data in numeric field
- Uninitialized field
- Data corruption
- Wrong data format

**Solutions:**
```
1. Check input data:
   Browse input file for invalid data

2. Initialize fields:
   INITIALIZE WS-NUMERIC-FIELD

3. Validate data before use:
   IF field IS NUMERIC

4. Check data conversion:
   Verify COMP-3 fields
```

### Issue: Job Runs Too Long

**Symptoms:**
- Job exceeds time limit
- S322 abend
- Performance degradation

**Possible Causes:**
- Inefficient program logic
- Large data volume
- VSAM file fragmentation
- Insufficient buffers

**Solutions:**
```
1. Increase TIME parameter:
   //JOBNAME JOB ...,TIME=30

2. Optimize program:
   Review logic for inefficiencies

3. Reorganize VSAM files:
   REPRO to defragment

4. Increase buffer allocation:
   AMP=('BUFNI=20,BUFND=40')
```

### Issue: Sort Fails

**Symptoms:**
- Sort step fails
- Insufficient sort work space

**Possible Causes:**
- Not enough SORTWK files
- SORTWK space too small
- Invalid sort keys
- Memory issues

**Solutions:**
```
1. Add more SORTWK files:
   //SORTWK01-04 DD ...

2. Increase SORTWK space:
   SPACE=(CYL,(100,20))

3. Verify sort keys:
   Check SORT FIELDS parameter

4. Increase region size:
   REGION=64M
```

## User Access Issues

### Issue: Cannot Sign On

**Symptoms:**
- Invalid user ID or password
- Sign-on rejected

**Possible Causes:**
- Wrong credentials
- Account locked
- Account inactive
- USRSEC file issue

**Solutions:**
```
1. Verify credentials:
   Check user ID and password

2. Check account status:
   Browse USRSEC file

3. Unlock account:
   Use CU02 to update status

4. Verify USRSEC file is open:
   CEMT INQ FILE(USRSEC)
```

### Issue: Access Denied to Function

**Symptoms:**
- Cannot access admin functions
- Transaction not authorized

**Possible Causes:**
- User not admin
- Wrong user type
- Security restriction
- Transaction disabled

**Solutions:**
```
1. Check user type:
   Browse USRSEC record

2. Update user type if needed:
   Use CU02 to change to ADMIN

3. Check transaction security:
   CEMT INQ TRAN(CA00)

4. Verify RACF permissions
```

## Transaction Issues

### Issue: Transaction Not Posting

**Symptoms:**
- Transaction entered but not saved
- Balance not updated

**Possible Causes:**
- Validation error
- File update failed
- Insufficient credit
- Program logic error

**Solutions:**
```
1. Check validation messages:
   Review screen for errors

2. Verify credit limit:
   Check available credit

3. Check file status:
   CEMT INQ FILE(TRANSACT)

4. Review program logic:
   Check COTRN02C program
```

### Issue: Duplicate Transaction ID

**Symptoms:**
- Transaction ID already exists
- Duplicate key error

**Possible Causes:**
- ID generation logic error
- Concurrent transactions
- System clock issue

**Solutions:**
```
1. Review ID generation:
   Check transaction ID logic

2. Add timestamp to ID:
   Include microseconds

3. Implement locking:
   Use ENQ/DEQ

4. Retry with new ID
```

## Optional Module Issues

### IMS-DB2-MQ Authorization Module

**Issue: Authorization not processing**
```
1. Check MQ queues:
   DISPLAY QUEUE(AWS.M2.CARDDEMO.PAUTH.REQUEST)

2. Verify IMS DB is available:
   /DIS DB DBPAUTP0

3. Check DB2 connection:
   CEMT INQ DB2CONN

4. Review CP00 transaction status:
   CEMT INQ TRAN(CP00)
```

### DB2 Transaction Type Module

**Issue: Cannot access transaction types**
```
1. Check DB2 connection:
   CEMT INQ DB2CONN

2. Verify DB2 plan bound:
   DISPLAY PLAN(CARDDEMO)

3. Check table exists:
   SELECT * FROM CARDDEMO.TRANSACTION_TYPE

4. Verify DB2ENTRY defined:
   CEMT INQ DB2ENTRY(CARDDEMO)
```

### VSAM-MQ Module

**Issue: MQ messages not processing**
```
1. Check MQ connection:
   CEMT INQ MQCONN

2. Verify queues exist:
   DISPLAY QUEUE(CARDDEMO.REQUEST.QUEUE)

3. Check queue depth:
   DISPLAY QUEUE(...) CURDEPTH

4. Verify CICS-MQ bridge:
   Check MQCONN resource
```

## Performance Issues

### Issue: Slow Response Time

**Symptoms:**
- Transactions take too long
- Screen updates slow
- Timeouts occurring

**Possible Causes:**
- VSAM file fragmentation
- Insufficient CICS resources
- Network latency
- High system load

**Solutions:**
```
1. Reorganize VSAM files:
   Submit reorganization jobs

2. Increase CICS resources:
   Adjust MXT, MAXTASK

3. Check system load:
   Review RMF reports

4. Optimize programs:
   Review program efficiency
```

### Issue: High CPU Usage

**Symptoms:**
- CPU utilization high
- System slow
- Batch jobs delayed

**Possible Causes:**
- Inefficient programs
- Looping code
- Large data volumes
- Insufficient tuning

**Solutions:**
```
1. Identify CPU-intensive programs:
   Review SMF records

2. Optimize program logic:
   Eliminate unnecessary loops

3. Tune VSAM buffers:
   Adjust BUFNI/BUFND

4. Schedule batch appropriately:
   Avoid peak hours
```

## Error Code Reference

### CICS ABEND Codes

| Code | Description | Common Cause |
|:-----|:------------|:-------------|
| AFCF | File control error | File not open or not found |
| ASRA | Program check | Data exception, protection exception |
| AEIV | Invalid request | Wrong CICS command syntax |
| APCT | Program control error | Program not found |
| AKCP | Task control error | Invalid task operation |

### JCL Return Codes

| Code | Description | Action |
|:-----|:------------|:-------|
| 0000 | Successful | None |
| 0004 | Warning | Review output |
| 0008 | Error | Fix and rerun |
| 0012 | Severe error | Investigate cause |
| S806 | Program not found | Check STEPLIB |
| S013 | File not found | Check dataset name |
| S213 | File in use | Close file |
| S222 | Operator cancel | Check with operations |
| S322 | Time limit exceeded | Increase TIME |
| SB37 | Out of space | Increase allocation |
| S0C7 | Data exception | Check numeric data |

### VSAM Return Codes

| Code | Description | Solution |
|:-----|:------------|:---------|
| 0 | Successful | None |
| 8 | Duplicate key | Check for duplicates |
| 16 | Record not found | Verify key |
| 24 | Out of space | Increase space |
| 28 | File not open | Open file |
| 92 | Logic error | Check program logic |

### DB2 SQL Codes

| Code | Description | Solution |
|:-----|:------------|:---------|
| 0 | Successful | None |
| 100 | No data found | Check query |
| -803 | Duplicate key | Check uniqueness |
| -911 | Deadlock | Retry transaction |
| -913 | Resource unavailable | Wait and retry |

---

**Navigation**: [Home](Home.md) | [Installation Guide](Installation-Guide.md) | [FAQ](FAQ.md) | [Technical Components](Technical-Components.md)
