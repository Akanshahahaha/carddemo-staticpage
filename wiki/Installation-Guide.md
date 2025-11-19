# Installation Guide

This comprehensive guide walks you through the complete installation process for CardDemo, from prerequisites to final verification.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Installation Overview](#installation-overview)
- [Step 1: Environment Preparation](#step-1-environment-preparation)
- [Step 2: Dataset Creation](#step-2-dataset-creation)
- [Step 3: Source Code Upload](#step-3-source-code-upload)
- [Step 4: Sample Data Upload](#step-4-sample-data-upload)
- [Step 5: Environment Initialization](#step-5-environment-initialization)
- [Step 6: Program Compilation](#step-6-program-compilation)
- [Step 7: CICS Configuration](#step-7-cics-configuration)
- [Step 8: Verification](#step-8-verification)
- [Optional Module Installation](#optional-module-installation)
- [Troubleshooting Installation](#troubleshooting-installation)

## Prerequisites

Before beginning the installation, ensure you have the following:

### Mainframe Environment
- z/OS operating system
- CICS Transaction Server
- VSAM support
- JCL execution capability
- File transfer capability (FTP, SFTP, or similar)

### Optional Components
For optional modules, you'll also need:
- **DB2**: For Transaction Type Management module
- **IMS DB**: For Credit Card Authorizations module
- **MQ**: For Authorization and Account Extraction modules

### Access Requirements
- TSO/ISPF access
- Authority to create datasets
- Authority to define CICS resources
- Authority to submit JCL jobs
- File transfer access between local and mainframe

### Skills Required
- Basic mainframe navigation
- Understanding of TSO/ISPF
- Familiarity with JCL
- Basic CICS administration knowledge

## Installation Overview

The installation process follows these major phases:

1. **Preparation**: Clone repository and prepare environment
2. **Dataset Setup**: Create mainframe datasets
3. **Code Upload**: Transfer source code to mainframe
4. **Data Upload**: Transfer sample data files
5. **Initialization**: Execute setup JCL jobs
6. **Compilation**: Compile COBOL programs
7. **CICS Setup**: Configure CICS resources
8. **Verification**: Test the installation

**Estimated Time**: 2-4 hours for base installation

## Step 1: Environment Preparation

### 1.1 Clone the Repository

Clone the CardDemo repository to your local development environment:

```bash
git clone https://github.com/aws-samples/aws-mainframe-modernization-carddemo.git
cd aws-mainframe-modernization-carddemo
```

### 1.2 Review the Structure

Familiarize yourself with the repository structure:

```
app/
├── asm/        # Assembler programs
├── bms/        # BMS map definitions
├── cbl/        # COBOL programs
├── cpy/        # Copybooks
├── cpy-bms/    # BMS copybooks
├── csd/        # CICS resource definitions
├── data/       # Sample data files
├── jcl/        # JCL jobs
├── maclib/     # Macro libraries
└── proc/       # JCL procedures
```

### 1.3 Define Your High Level Qualifier (HLQ)

Choose a High Level Qualifier for your datasets. This guide uses `AWS.M2` as an example, but you should use your organization's naming convention.

**Example HLQ**: `AWS.M2`

## Step 2: Dataset Creation

Create the following datasets on your mainframe system. Replace `AWS.M2` with your chosen HLQ.

### 2.1 Source Code Datasets

Create these datasets with **FB (Fixed Block)** format and **LRECL=80**:

| Dataset Name | Description | Format | LRECL |
|:-------------|:------------|:-------|------:|
| AWS.M2.CARDDEMO.JCL | JCL jobs | FB | 80 |
| AWS.M2.CARDDEMO.PROC | JCL procedures | FB | 80 |
| AWS.M2.CARDDEMO.CBL | COBOL programs | FB | 80 |
| AWS.M2.CARDDEMO.CPY | Copybooks | FB | 80 |
| AWS.M2.CARDDEMO.BMS | BMS maps | FB | 80 |
| AWS.M2.CARDDEMO.ASM | Assembler programs | FB | 80 |
| AWS.M2.CARDDEMO.MACLIB | Macro library | FB | 80 |

### 2.2 Sample Dataset Allocation Commands

Using ISPF 3.2 (Dataset Utility):

```
ALLOCATE DATASET('AWS.M2.CARDDEMO.JCL')
  SPACE(10,5) TRACKS
  RECFM(F,B) LRECL(80) BLKSIZE(27920)
  DSORG(PO) DSNTYPE(LIBRARY)
```

Or using JCL:

```jcl
//ALLOC    EXEC PGM=IEFBR14
//DD1      DD DSN=AWS.M2.CARDDEMO.JCL,
//            DISP=(NEW,CATLG,DELETE),
//            SPACE=(TRK,(10,5,20)),
//            DCB=(RECFM=FB,LRECL=80,BLKSIZE=27920),
//            DSNTYPE=LIBRARY
```

### 2.3 Data File Datasets

Create these datasets for sample data:

| Dataset Name | Description | Copybook | Format | LRECL |
|:-------------|:------------|:---------|:-------|------:|
| AWS.M2.CARDDEMO.USRSEC.PS | User Security | CSUSR01Y | FB | 80 |
| AWS.M2.CARDDEMO.ACCTDATA.PS | Account Data | CVACT01Y | FB | 300 |
| AWS.M2.CARDDEMO.CARDDATA.PS | Card Data | CVACT02Y | FB | 150 |
| AWS.M2.CARDDEMO.CUSTDATA.PS | Customer Data | CVCUS01Y | FB | 500 |
| AWS.M2.CARDDEMO.CARDXREF.PS | Card Cross Reference | CVACT03Y | FB | 50 |
| AWS.M2.CARDDEMO.DALYTRAN.PS.INIT | Transaction Init | CVTRA06Y | FB | 350 |
| AWS.M2.CARDDEMO.DALYTRAN.PS | Transaction Data | CVTRA06Y | FB | 350 |
| AWS.M2.CARDDEMO.DISCGRP.PS | Disclosure Groups | CVTRA02Y | FB | 50 |
| AWS.M2.CARDDEMO.TRANCATG.PS | Transaction Categories | CVTRA04Y | FB | 60 |
| AWS.M2.CARDDEMO.TRANTYPE.PS | Transaction Types | CVTRA03Y | FB | 60 |
| AWS.M2.CARDDEMO.TCATBALF.PS | Category Balance | CVTRA01Y | FB | 50 |

## Step 3: Source Code Upload

### 3.1 Upload Source Files

Transfer the source code from your local repository to the mainframe datasets:

**Using FTP:**

```bash
ftp mainframe.example.com
quote site recfm=fb lrecl=80 blksize=27920
cd 'AWS.M2.CARDDEMO.CBL'
mput app/cbl/*.cbl
cd 'AWS.M2.CARDDEMO.CPY'
mput app/cpy/*.cpy
cd 'AWS.M2.CARDDEMO.BMS'
mput app/bms/*.bms
cd 'AWS.M2.CARDDEMO.JCL'
mput app/jcl/*.jcl
cd 'AWS.M2.CARDDEMO.ASM'
mput app/asm/*.asm
```

**Using SFTP or File Transfer Tool:**

Use your organization's preferred file transfer method, ensuring:
- Text mode transfer for source code
- Proper EBCDIC conversion
- Correct record format preservation

### 3.2 Verify Upload

Check that all files were transferred successfully:

```
TSO ISPF 3.4
Enter: AWS.M2.CARDDEMO.*
```

Verify member counts:
- CBL: 29 members
- JCL: 33 members
- CPY: Multiple copybooks
- BMS: Multiple map definitions

## Step 4: Sample Data Upload

### 4.1 Upload Sample Data Files

Transfer sample data from `app/data/EBCDIC/` to the mainframe:

**Important**: Use **binary transfer mode** to preserve data integrity.

```bash
ftp mainframe.example.com
binary
put app/data/EBCDIC/USRSEC.txt 'AWS.M2.CARDDEMO.USRSEC.PS'
put app/data/EBCDIC/ACCTDATA.txt 'AWS.M2.CARDDEMO.ACCTDATA.PS'
put app/data/EBCDIC/CARDDATA.txt 'AWS.M2.CARDDEMO.CARDDATA.PS'
put app/data/EBCDIC/CUSTDATA.txt 'AWS.M2.CARDDEMO.CUSTDATA.PS'
put app/data/EBCDIC/CARDXREF.txt 'AWS.M2.CARDDEMO.CARDXREF.PS'
put app/data/EBCDIC/DALYTRAN.txt 'AWS.M2.CARDDEMO.DALYTRAN.PS.INIT'
put app/data/EBCDIC/DISCGRP.txt 'AWS.M2.CARDDEMO.DISCGRP.PS'
put app/data/EBCDIC/TRANCATG.txt 'AWS.M2.CARDDEMO.TRANCATG.PS'
put app/data/EBCDIC/TRANTYPE.txt 'AWS.M2.CARDDEMO.TRANTYPE.PS'
put app/data/EBCDIC/TCATBALF.txt 'AWS.M2.CARDDEMO.TCATBALF.PS'
```

### 4.2 Verify Data Upload

Browse the datasets to verify data was uploaded correctly:

```
TSO ISPF 1 (Browse)
Enter: 'AWS.M2.CARDDEMO.USRSEC.PS'
```

Check for readable data in EBCDIC format.

## Step 5: Environment Initialization

Execute the initialization JCL jobs in the specified sequence. Each job must complete successfully before proceeding to the next.

### 5.1 Update JCL Parameters

Before submitting jobs, update the HLQ in each JCL:

1. Edit each JCL member
2. Change all occurrences of the HLQ to match your environment
3. Update CICS region name if different
4. Update VSAM catalog name if required

### 5.2 Execute Initialization Jobs

Submit the following jobs in order:

| Order | Jobname | Purpose | Expected RC |
|:------|:--------|:--------|:------------|
| 1 | DUSRSECJ | Sets up user security VSAM file | 0 |
| 2 | CLOSEFIL | Closes files opened by CICS | 0 |
| 3 | ACCTFILE | Loads Account database | 0 |
| 4 | CARDFILE | Loads Card database | 0 |
| 5 | CUSTFILE | Creates customer database | 0 |
| 6 | XREFFILE | Loads cross reference | 0 |
| 7 | TRANFILE | Copies initial Transaction file | 0 |
| 8 | DISCGRP | Copies Disclosure Group file | 0 |
| 9 | TCATBALF | Copies TCATBALF file | 0 |
| 10 | TRANCATG | Copies transaction category file | 0 |
| 11 | TRANTYPE | Copies transaction type file | 0 |
| 12 | OPENFIL | Makes files available to CICS | 0 |
| 13 | DEFGDGB | Defines GDG Base | 0 |

### 5.3 Verify Job Completion

For each job:
1. Check the job output for RC=0000
2. Review SYSOUT for any error messages
3. Verify VSAM files were created successfully

**Check VSAM Files:**

```
TSO ISPF 3.4
Enter: AWS.M2.CARDDEMO.*.VSAM.*
```

You should see:
- ACCTDAT.VSAM.KSDS
- CARDDAT.VSAM.KSDS
- CUSTDAT.VSAM.KSDS
- CARDXREF.VSAM.KSDS
- TRANSACT.VSAM.KSDS
- USRSEC.VSAM.KSDS
- And others

## Step 6: Program Compilation

### 6.1 Compilation Overview

CardDemo programs must be compiled in the following order:
1. Assembler programs (utilities)
2. BMS maps
3. COBOL programs

### 6.2 Compile Assembler Programs

Submit compilation JCL for assembler programs:

```
Submit: AWS.M2.CARDDEMO.JCL(ASMCOMP)
```

Programs to compile:
- MVSWAIT: Timer control utility
- COBDATFT: Date format conversion utility

### 6.3 Compile BMS Maps

Submit BMS compilation JCL:

```
Submit: AWS.M2.CARDDEMO.JCL(BMSCOMP)
```

This compiles all BMS maps and generates symbolic copybooks.

### 6.4 Compile COBOL Programs

Submit COBOL compilation JCL for each program or use batch compilation:

```
Submit: AWS.M2.CARDDEMO.JCL(CBLCOMP)
```

**Online Programs to Compile:**
- COSGN00C (Sign-on)
- COMEN01C (Main Menu)
- COACTVWC (Account View)
- COACTUPC (Account Update)
- COCRDLIC (Card List)
- COCRDSLC (Card View)
- COCRDUPC (Card Update)
- COTRN00C (Transaction List)
- COTRN01C (Transaction View)
- COTRN02C (Transaction Add)
- CORPT00C (Reports)
- COBIL00C (Bill Payment)
- COADM01C (Admin Menu)
- COUSR00C-03C (User Management)

**Batch Programs to Compile:**
- CBTRN02C (Transaction Posting)
- CBACT04C (Interest Calculation)
- CBSTM03A (Statement Generation)
- CBTRN03C (Transaction Report)

### 6.5 Verify Compilation

Check that all programs compiled successfully:

```
TSO ISPF 3.4
Enter: AWS.M2.CARDDEMO.LOADLIB
```

Verify all load modules are present with recent timestamps.

## Step 7: CICS Configuration

### 7.1 Define CICS Resources

You have two options for defining CICS resources:

**Option 1: Use DFHCSDUP (Recommended)**

Submit the DFHCSDUP JCL with the CSD file:

```
Submit: AWS.M2.CARDDEMO.CSD(DFHCSDUP)
```

This automatically defines all required resources.

**Option 2: Manual Definition via CEDA**

Use CEDA transaction to manually define resources. See [CICS Configuration](CICS-Configuration.md) for detailed commands.

### 7.2 Define Resource Groups

Define the CARDDEMO resource group:

```
CEDA DEFINE GROUP(CARDDEMO) LIST(CARDLIST)
```

### 7.3 Define Programs

Define all COBOL programs:

```
CEDA DEFINE PROGRAM(COSGN00C) GROUP(CARDDEMO) LANGUAGE(COBOL)
CEDA DEFINE PROGRAM(COMEN01C) GROUP(CARDDEMO) LANGUAGE(COBOL)
... (repeat for all programs)
```

### 7.4 Define Transactions

Define all transactions:

```
CEDA DEFINE TRANSACTION(CC00) GROUP(CARDDEMO) PROGRAM(COSGN00C)
CEDA DEFINE TRANSACTION(CM00) GROUP(CARDDEMO) PROGRAM(COMEN01C)
... (repeat for all transactions)
```

### 7.5 Define Files

Define all VSAM files:

```
CEDA DEFINE FILE(ACCTDAT) GROUP(CARDDEMO) DSNAME(AWS.M2.CARDDEMO.ACCTDAT.VSAM.KSDS)
CEDA DEFINE FILE(CARDDAT) GROUP(CARDDEMO) DSNAME(AWS.M2.CARDDEMO.CARDDAT.VSAM.KSDS)
... (repeat for all files)
```

### 7.6 Install Resources

Install the CARDDEMO group:

```
CEDA INSTALL GROUP(CARDDEMO)
```

Verify installation:

```
CEMT INQUIRE PROGRAM(COSGN00C)
CEMT INQUIRE TRANSACTION(CC00)
CEMT INQUIRE FILE(ACCTDAT)
```

## Step 8: Verification

### 8.1 Access the Application

1. Clear your CICS terminal
2. Enter transaction: `CC00`
3. You should see the CardDemo sign-on screen

### 8.2 Test User Login

**Admin User:**
- User ID: `ADMIN001`
- Password: `PASSWORD`

**Regular User:**
- User ID: `USER0001`
- Password: `PASSWORD`

### 8.3 Verify Core Functions

Test each major function:
1. Sign-on (CC00)
2. Main Menu (CM00)
3. Account View (CAVW)
4. Card List (CCLI)
5. Transaction List (CT00)

### 8.4 Run Test Batch Job

Submit a test batch job:

```
Submit: AWS.M2.CARDDEMO.JCL(POSTTRAN)
```

Verify it completes successfully.

## Optional Module Installation

After completing the base installation, you can install optional modules:

### DB2 Transaction Type Management
See: [Optional Module - Transaction Type Management](Optional-Module-Transaction-Type.md)

### IMS-DB2-MQ Credit Card Authorizations
See: [Optional Module - Credit Card Authorizations](Optional-Module-Authorization.md)

### VSAM-MQ Account Extractions
See: [Optional Module - Account Extractions](Optional-Module-VSAM-MQ.md)

## Troubleshooting Installation

### Common Issues

**Issue: JCL Job Fails with S806**
- **Cause**: Program not found in load library
- **Solution**: Verify compilation completed successfully and load library is in STEPLIB

**Issue: CICS Transaction ABEND AFCF**
- **Cause**: File not found or not open
- **Solution**: Check file definitions and ensure OPENFIL job ran successfully

**Issue: VSAM File Not Found**
- **Cause**: Dataset not cataloged or incorrect name
- **Solution**: Verify dataset name matches CICS file definition

**Issue: Compilation Errors**
- **Cause**: Missing copybooks or incorrect library concatenation
- **Solution**: Verify SYSLIB DD statements include all copybook libraries

### Getting Help

If you encounter issues not covered here:
1. Check the [Troubleshooting](Troubleshooting.md) page
2. Review the [FAQ](FAQ.md)
3. Consult CICS and z/OS system logs
4. Raise an issue in the GitHub repository

## Next Steps

After successful installation:
1. Review the [User Guide](User-Guide.md) to understand application features
2. Read the [Admin Guide](Admin-Guide.md) for administrative functions
3. Explore [Batch Processing](Batch-Processing.md) for batch job details
4. Consider installing [Optional Modules](Home.md#optional-modules)

---

**Navigation**: [Home](Home.md) | [Quick Start](Quick-Start.md) | [CICS Configuration](CICS-Configuration.md) | [Troubleshooting](Troubleshooting.md)
