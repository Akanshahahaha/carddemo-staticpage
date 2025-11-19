# Quick Start Guide

Get up and running with CardDemo quickly. This guide provides the fastest path to a working installation for evaluation and testing purposes.

## Prerequisites Check

Before starting, verify you have:
- [ ] z/OS mainframe environment with CICS
- [ ] TSO/ISPF access
- [ ] Authority to create datasets and submit jobs
- [ ] File transfer capability (FTP/SFTP)
- [ ] 30-60 minutes for installation

## Quick Installation Steps

### 1. Clone Repository (5 minutes)

```bash
git clone https://github.com/aws-samples/aws-mainframe-modernization-carddemo.git
cd aws-mainframe-modernization-carddemo
```

### 2. Create Datasets (10 minutes)

Create mainframe datasets with your HLQ (e.g., `AWS.M2`):

**Source Datasets** (FB, LRECL=80):
- AWS.M2.CARDDEMO.JCL
- AWS.M2.CARDDEMO.CBL
- AWS.M2.CARDDEMO.CPY
- AWS.M2.CARDDEMO.BMS
- AWS.M2.CARDDEMO.ASM

**Data Datasets** (Various LRECL):
- AWS.M2.CARDDEMO.USRSEC.PS (LRECL=80)
- AWS.M2.CARDDEMO.ACCTDATA.PS (LRECL=300)
- AWS.M2.CARDDEMO.CARDDATA.PS (LRECL=150)
- AWS.M2.CARDDEMO.CUSTDATA.PS (LRECL=500)

See [Installation Guide](Installation-Guide.md#step-2-dataset-creation) for complete list.

### 3. Upload Files (10 minutes)

**Upload source code** (text mode):
```bash
ftp mainframe.example.com
quote site recfm=fb lrecl=80
cd 'AWS.M2.CARDDEMO.CBL'
mput app/cbl/*.cbl
# Repeat for CPY, BMS, JCL, ASM
```

**Upload sample data** (binary mode):
```bash
binary
put app/data/EBCDIC/USRSEC.txt 'AWS.M2.CARDDEMO.USRSEC.PS'
# Repeat for all data files
```

### 4. Initialize Environment (15 minutes)

Submit initialization jobs in order:

```
1. DUSRSECJ  - User security setup
2. CLOSEFIL  - Close CICS files
3. ACCTFILE  - Load accounts
4. CARDFILE  - Load cards
5. CUSTFILE  - Load customers
6. XREFFILE  - Load cross-reference
7. TRANFILE  - Load transactions
8. DISCGRP   - Load disclosure groups
9. TCATBALF  - Load category balance
10. TRANCATG - Load transaction categories
11. TRANTYPE - Load transaction types
12. OPENFIL  - Open CICS files
13. DEFGDGB  - Define GDG base
```

**Verify**: All jobs should complete with RC=0000

### 5. Compile Programs (10 minutes)

```
1. Submit: ASMCOMP  - Compile assembler utilities
2. Submit: BMSCOMP  - Compile BMS maps
3. Submit: CBLCOMP  - Compile COBOL programs
```

**Verify**: Check load library for compiled modules

### 6. Configure CICS (10 minutes)

**Option A - Automated** (Recommended):
```
Submit: DFHCSDUP job from CSD folder
```

**Option B - Manual**:
```
CEDA DEFINE GROUP(CARDDEMO)
CEDA DEFINE PROGRAM(COSGN00C) GROUP(CARDDEMO)
CEDA DEFINE TRANSACTION(CC00) GROUP(CARDDEMO) PROGRAM(COSGN00C)
# Repeat for all programs/transactions/files
CEDA INSTALL GROUP(CARDDEMO)
```

### 7. Test Access (5 minutes)

1. Clear CICS terminal
2. Enter: `CC00`
3. Login with:
   - User: `ADMIN001`
   - Password: `PASSWORD`

**Success!** You should see the CardDemo main menu.

## Quick Test Scenarios

### Test User Functions

1. **View Account** (CAVW)
   - Enter account: `0000000001`
   - View balance and details

2. **List Cards** (CCLI)
   - View all cards for account
   - Select a card to view details

3. **View Transactions** (CT00)
   - Enter card number
   - Browse transaction history

4. **Add Transaction** (CT02)
   - Enter transaction details
   - Submit for processing

### Test Admin Functions

1. **Admin Menu** (CA00)
   - Login as ADMIN001
   - Access admin functions

2. **List Users** (CU00)
   - View all system users

3. **Add User** (CU01)
   - Create a new user account

### Test Batch Processing

Submit a batch job:
```
Submit: POSTTRAN
```

Check output for successful transaction posting.

## Default Test Accounts

### User Accounts
- **USER0001** / PASSWORD - Regular user
- **USER0002** / PASSWORD - Regular user

### Admin Accounts
- **ADMIN001** / PASSWORD - Administrator

### Test Credit Cards
- **4000123456789010** - Active card
- **4000123456789011** - Active card

### Test Accounts
- **0000000001** - Sample account with transactions
- **0000000002** - Sample account with transactions

## Common Quick Start Issues

### Issue: Can't Access CC00 Transaction
**Solution**: 
- Verify CICS resources installed: `CEMT INQ TRAN(CC00)`
- Check program loaded: `CEMT INQ PROG(COSGN00C)`

### Issue: File Not Found Error
**Solution**:
- Verify OPENFIL job completed successfully
- Check file status: `CEMT INQ FILE(ACCTDAT)`

### Issue: Login Fails
**Solution**:
- Verify DUSRSECJ job completed
- Check USRSEC file is open
- Confirm user ID and password are correct

### Issue: Compilation Errors
**Solution**:
- Verify copybook libraries in SYSLIB
- Check JCL for correct dataset names
- Review compiler output for specific errors

## Next Steps

Now that CardDemo is running:

1. **Explore Features**: Try all menu options and transactions
2. **Read Documentation**: 
   - [User Guide](User-Guide.md) - Learn all user functions
   - [Admin Guide](Admin-Guide.md) - Understand admin capabilities
   - [Batch Processing](Batch-Processing.md) - Run batch jobs
3. **Install Optional Modules**:
   - [Credit Card Authorizations](Optional-Module-Authorization.md)
   - [Transaction Type Management](Optional-Module-Transaction-Type.md)
   - [Account Extractions](Optional-Module-VSAM-MQ.md)
4. **Customize**: Modify for your specific use case

## Getting Help

- **Detailed Installation**: See [Installation Guide](Installation-Guide.md)
- **Troubleshooting**: See [Troubleshooting](Troubleshooting.md)
- **FAQ**: See [FAQ](FAQ.md)
- **Issues**: Report in GitHub repository

---

**Navigation**: [Home](Home.md) | [Installation Guide](Installation-Guide.md) | [User Guide](User-Guide.md) | [Troubleshooting](Troubleshooting.md)
