# Optional Module: Account Extractions (VSAM-MQ)

Complete documentation for the Account Extractions optional module, which demonstrates MQ integration for data extraction and inquiry.

## Table of Contents
- [Module Overview](#module-overview)
- [Features](#features)
- [Architecture](#architecture)
- [Installation](#installation)
- [Components](#components)
- [Message Formats](#message-formats)
- [Usage Guide](#usage-guide)
- [Technical Details](#technical-details)

## Module Overview

The Account Extractions module is an optional extension that demonstrates integration between VSAM and IBM MQ. This module enables the extraction and transmission of account data through MQ channels, showcasing asynchronous processing patterns commonly used in mainframe environments.

### Business Value

- **Asynchronous Communication**: Non-blocking data exchange via MQ
- **System Integration**: Connect mainframe applications with distributed systems
- **Data Extraction**: Extract account and system data on demand
- **Request/Response Pattern**: Demonstrate MQ request/response architecture
- **Loose Coupling**: Independent system communication

### Technologies Used

- **COBOL**: Business logic implementation
- **CICS**: Transaction processing
- **VSAM**: Data source for account information
- **MQ**: Message queuing for asynchronous communication

## Features

### System Date Inquiry (CDRD)

Query the system date through an MQ request/response pattern:
- Send date request via MQ
- Receive system date in response
- Demonstrates simple MQ communication

### Account Details Inquiry (CDRA)

Retrieve account information through MQ channels:
- Send account inquiry request with account number
- Retrieve complete account details from VSAM
- Receive account data in response message
- Demonstrates data extraction via MQ

### Integration Patterns

- **Request/Response**: Synchronous-style communication using asynchronous MQ
- **Message Correlation**: Correlate requests with responses
- **Data Transformation**: Convert VSAM data to message format
- **Error Handling**: Proper error handling for MQ operations

## Architecture

### Component Architecture

```
┌──────────────────┐
│ External System  │
│ / Client App     │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ MQ Request Queue │
│ CARDDEMO.REQUEST │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ CICS Transaction │
│ CDRD / CDRA      │
└────────┬─────────┘
         │
         ├─→ VSAM Files (Read Account Data)
         │
         └─→ MQ Response Queue
             │
             ▼
         ┌──────────────────┐
         │ MQ Response Queue│
         │ CARDDEMO.RESPONSE│
         └──────────────────┘
```

### Message Flow

**Date Inquiry Flow:**
```
1. Client sends date request → Request Queue
2. CDRD transaction triggered
3. Retrieves system date
4. Sends response → Response Queue
5. Client receives date response
```

**Account Inquiry Flow:**
```
1. Client sends account request → Request Queue
2. CDRA transaction triggered
3. Reads account from VSAM
4. Formats account data
5. Sends response → Response Queue
6. Client receives account data
```

## Installation

### Prerequisites

1. **Base CardDemo**: Installed and operational
2. **IBM MQ**: Configured and accessible from CICS
3. **CICS with MQ**: CICS-MQ bridge configured
4. **MQ Queues**: Request and response queues defined

### Installation Steps

#### Step 1: Configure MQ Resources

Define MQ queues for CardDemo:

```
DEFINE QLOCAL('CARDDEMO.REQUEST.QUEUE') REPLACE +
  DESCR('CardDemo Request Queue') +
  DEFPSIST(YES) +
  MAXDEPTH(5000) +
  USAGE(NORMAL)

DEFINE QLOCAL('CARDDEMO.RESPONSE.QUEUE') REPLACE +
  DESCR('CardDemo Response Queue') +
  DEFPSIST(YES) +
  MAXDEPTH(5000) +
  USAGE(NORMAL)
```

#### Step 2: Upload Source Code

Upload module source code to mainframe datasets:

```
app/app-vsam-mq/cbl/     → CARDDEMO.CBL
app/app-vsam-mq/csd/     → CICS resource definitions
```

Programs to upload:
- **CODATE01**: System date inquiry
- **COACCT01**: Account details inquiry

#### Step 3: Compile Programs

Compile COBOL programs with MQ support:

```jcl
//COMPILE  EXEC PGM=IGYCRCTL,PARM='LIB,OBJECT,MAP'
//STEPLIB  DD DSN=COBOL.SIGYCOMP,DISP=SHR
//SYSLIB   DD DSN=CARDDEMO.CPY,DISP=SHR
//         DD DSN=MQ.SCSQCOBC,DISP=SHR
//SYSIN    DD DSN=CARDDEMO.CBL(CODATE01),DISP=SHR
//SYSLIN   DD DSN=&&LOADSET,DISP=(MOD,PASS)
//SYSPRINT DD SYSOUT=*
```

Link with MQ libraries:

```jcl
//LINK     EXEC PGM=IEWL,PARM='LIST,XREF,LET'
//SYSLIB   DD DSN=MQ.SCSQLOAD,DISP=SHR
//         DD DSN=CICS.SDFHLOAD,DISP=SHR
//SYSLMOD  DD DSN=CARDDEMO.LOADLIB,DISP=SHR
//SYSLIN   DD DSN=&&LOADSET,DISP=(OLD,DELETE)
//SYSPRINT DD SYSOUT=*
```

#### Step 4: Define CICS Resources

**Programs:**
```
CEDA DEFINE PROGRAM(CODATE01) GROUP(CARDDEMO) LANGUAGE(COBOL)
CEDA DEFINE PROGRAM(COACCT01) GROUP(CARDDEMO) LANGUAGE(COBOL)
```

**Transactions:**
```
CEDA DEFINE TRANSACTION(CDRD) GROUP(CARDDEMO) PROGRAM(CODATE01)
CEDA DEFINE TRANSACTION(CDRA) GROUP(CARDDEMO) PROGRAM(COACCT01)
```

**MQ Connection:**
```
CEDA DEFINE MQCONN(MQ01) GROUP(CARDDEMO) +
  MQNAME(MQ.QUEUE.MANAGER) +
  INITQNAME(SYSTEM.CICS.INITQUEUE)
```

**MQ Queues:**
```
CEDA DEFINE MQQUEUE(CARDREQ) GROUP(CARDDEMO) +
  QNAME(CARDDEMO.REQUEST.QUEUE) +
  DESCRIPTION(CardDemo Request Queue)

CEDA DEFINE MQQUEUE(CARDRES) GROUP(CARDDEMO) +
  QNAME(CARDDEMO.RESPONSE.QUEUE) +
  DESCRIPTION(CardDemo Response Queue)
```

**Install Resources:**
```
CEDA INSTALL GROUP(CARDDEMO)
```

#### Step 5: Test Installation

**Test Date Inquiry:**
1. Send test message to request queue
2. Execute CDRD transaction
3. Verify response in response queue

**Test Account Inquiry:**
1. Send account request with account number
2. Execute CDRA transaction
3. Verify account data in response

## Components

### Programs

#### CODATE01 - System Date Inquiry
- **Transaction**: CDRD
- **Function**: Retrieve and return system date via MQ
- **Input**: Date request message from MQ
- **Output**: Date response message to MQ
- **Processing**:
  1. Read request from MQ request queue
  2. Get system date
  3. Format date response
  4. Write response to MQ response queue

**COBOL Structure:**
```cobol
IDENTIFICATION DIVISION.
PROGRAM-ID. CODATE01.

DATA DIVISION.
WORKING-STORAGE SECTION.
01  WS-REQUEST-MSG.
    05  WS-REQUEST-TYPE        PIC X(04).
    05  WS-REQUEST-ID          PIC X(08).

01  WS-RESPONSE-MSG.
    05  WS-RESPONSE-TYPE       PIC X(04).
    05  WS-RESPONSE-ID         PIC X(08).
    05  WS-SYSTEM-DATE         PIC X(10).

PROCEDURE DIVISION.
    PERFORM READ-MQ-REQUEST
    PERFORM GET-SYSTEM-DATE
    PERFORM SEND-MQ-RESPONSE
    GOBACK.
```

#### COACCT01 - Account Details Inquiry
- **Transaction**: CDRA
- **Function**: Retrieve account details from VSAM and return via MQ
- **Input**: Account request message with account number
- **Output**: Account data response message
- **Files Accessed**: ACCTDAT (VSAM)
- **Processing**:
  1. Read request from MQ request queue
  2. Extract account number from request
  3. Read account record from VSAM
  4. Format account data for response
  5. Write response to MQ response queue

**COBOL Structure:**
```cobol
IDENTIFICATION DIVISION.
PROGRAM-ID. COACCT01.

DATA DIVISION.
WORKING-STORAGE SECTION.
01  WS-ACCT-REQUEST-MSG.
    05  WS-REQUEST-TYPE        PIC X(04).
    05  WS-REQUEST-ID          PIC X(08).
    05  WS-ACCOUNT-NUMBER      PIC X(11).

01  WS-ACCT-RESPONSE-MSG.
    05  WS-RESPONSE-TYPE       PIC X(04).
    05  WS-RESPONSE-ID         PIC X(08).
    05  WS-ACCOUNT-DATA        PIC X(300).

PROCEDURE DIVISION.
    PERFORM READ-MQ-REQUEST
    PERFORM READ-VSAM-ACCOUNT
    PERFORM FORMAT-RESPONSE
    PERFORM SEND-MQ-RESPONSE
    GOBACK.
```

## Message Formats

### Date Request Message

**Format:**
```
01  DATE-REQUEST-MSG.
    05  REQUEST-TYPE           PIC X(04) VALUE 'DATE'.
    05  REQUEST-ID             PIC X(08).
```

**Example:**
```
DATE12345678
```

**Fields:**
- **REQUEST-TYPE**: 'DATE' (4 bytes)
- **REQUEST-ID**: Unique request identifier (8 bytes)

### Date Response Message

**Format:**
```
01  DATE-RESPONSE-MSG.
    05  RESPONSE-TYPE          PIC X(04) VALUE 'DATE'.
    05  RESPONSE-ID            PIC X(08).
    05  SYSTEM-DATE            PIC X(10).
```

**Example:**
```
DATE123456782025-11-18
```

**Fields:**
- **RESPONSE-TYPE**: 'DATE' (4 bytes)
- **RESPONSE-ID**: Matches request ID (8 bytes)
- **SYSTEM-DATE**: YYYY-MM-DD format (10 bytes)

### Account Request Message

**Format:**
```
01  ACCT-REQUEST-MSG.
    05  REQUEST-TYPE           PIC X(04) VALUE 'ACCT'.
    05  REQUEST-ID             PIC X(08).
    05  ACCOUNT-NUMBER         PIC X(11).
```

**Example:**
```
ACCT1234567800000000001
```

**Fields:**
- **REQUEST-TYPE**: 'ACCT' (4 bytes)
- **REQUEST-ID**: Unique request identifier (8 bytes)
- **ACCOUNT-NUMBER**: Account ID (11 bytes)

### Account Response Message

**Format:**
```
01  ACCT-RESPONSE-MSG.
    05  RESPONSE-TYPE          PIC X(04) VALUE 'ACCT'.
    05  RESPONSE-ID            PIC X(08).
    05  RESPONSE-CODE          PIC X(02).
    05  ACCOUNT-DATA           PIC X(300).
```

**Example:**
```
ACCT1234567800[account data 300 bytes]
```

**Fields:**
- **RESPONSE-TYPE**: 'ACCT' (4 bytes)
- **RESPONSE-ID**: Matches request ID (8 bytes)
- **RESPONSE-CODE**: '00' = Success, '01' = Not Found, '99' = Error (2 bytes)
- **ACCOUNT-DATA**: Account record data (300 bytes)

**Response Codes:**
- **00**: Success - Account found
- **01**: Account not found
- **99**: System error

## Usage Guide

### System Date Inquiry

**Using CICS Transaction:**

1. Execute transaction: `CDRD`
2. Transaction reads from request queue
3. Returns date to response queue

**Using MQ Client:**

```python
import pymqi

# Connect to queue manager
qmgr = pymqi.connect('MQ.QUEUE.MANAGER')

# Put request message
request_queue = pymqi.Queue(qmgr, 'CARDDEMO.REQUEST.QUEUE')
request_msg = 'DATE12345678'
request_queue.put(request_msg)

# Get response message
response_queue = pymqi.Queue(qmgr, 'CARDDEMO.RESPONSE.QUEUE')
response_msg = response_queue.get()
print(f"System Date: {response_msg[12:22]}")

# Disconnect
qmgr.disconnect()
```

### Account Details Inquiry

**Using CICS Transaction:**

1. Execute transaction: `CDRA`
2. Transaction reads account request from queue
3. Retrieves account from VSAM
4. Returns account data to response queue

**Using MQ Client:**

```python
import pymqi

# Connect to queue manager
qmgr = pymqi.connect('MQ.QUEUE.MANAGER')

# Put request message
request_queue = pymqi.Queue(qmgr, 'CARDDEMO.REQUEST.QUEUE')
account_number = '00000000001'
request_msg = f'ACCT12345678{account_number}'
request_queue.put(request_msg)

# Get response message
response_queue = pymqi.Queue(qmgr, 'CARDDEMO.RESPONSE.QUEUE')
response_msg = response_queue.get()

# Parse response
response_code = response_msg[12:14]
if response_code == '00':
    account_data = response_msg[14:314]
    print(f"Account Data: {account_data}")
else:
    print(f"Error: {response_code}")

# Disconnect
qmgr.disconnect()
```

## Technical Details

### MQ Operations in COBOL

**Read from Queue:**
```cobol
EXEC CICS READQ TS
    QUEUE('CARDDEMO.REQUEST.QUEUE')
    INTO(WS-REQUEST-MSG)
    LENGTH(WS-MSG-LEN)
    RESP(WS-RESP)
END-EXEC.

IF WS-RESP NOT = DFHRESP(NORMAL)
    PERFORM ERROR-HANDLING
END-IF.
```

**Write to Queue:**
```cobol
EXEC CICS WRITEQ TS
    QUEUE('CARDDEMO.RESPONSE.QUEUE')
    FROM(WS-RESPONSE-MSG)
    LENGTH(WS-MSG-LEN)
    RESP(WS-RESP)
END-EXEC.

IF WS-RESP NOT = DFHRESP(NORMAL)
    PERFORM ERROR-HANDLING
END-IF.
```

### Message Correlation

**Request ID Generation:**
```cobol
MOVE FUNCTION CURRENT-DATE TO WS-TIMESTAMP.
STRING WS-TIMESTAMP(1:8) DELIMITED BY SIZE
       WS-TIMESTAMP(9:6) DELIMITED BY SIZE
       INTO WS-REQUEST-ID.
```

**Response Correlation:**
```cobol
MOVE WS-REQUEST-ID TO WS-RESPONSE-ID.
```

### VSAM Access

**Read Account:**
```cobol
EXEC CICS READ
    FILE('ACCTDAT')
    INTO(ACCOUNT-RECORD)
    RIDFLD(WS-ACCOUNT-NUMBER)
    RESP(WS-RESP)
END-EXEC.

EVALUATE WS-RESP
    WHEN DFHRESP(NORMAL)
        MOVE '00' TO WS-RESPONSE-CODE
        MOVE ACCOUNT-RECORD TO WS-ACCOUNT-DATA
    WHEN DFHRESP(NOTFND)
        MOVE '01' TO WS-RESPONSE-CODE
    WHEN OTHER
        MOVE '99' TO WS-RESPONSE-CODE
END-EVALUATE.
```

### Error Handling

**MQ Error Handling:**
```cobol
IF WS-RESP NOT = DFHRESP(NORMAL)
    EVALUATE WS-RESP
        WHEN DFHRESP(QIDERR)
            MOVE 'Queue not found' TO WS-ERROR-MSG
        WHEN DFHRESP(ITEMERR)
            MOVE 'No messages available' TO WS-ERROR-MSG
        WHEN DFHRESP(LENGERR)
            MOVE 'Message length error' TO WS-ERROR-MSG
        WHEN OTHER
            MOVE 'Unknown MQ error' TO WS-ERROR-MSG
    END-EVALUATE
    PERFORM LOG-ERROR
END-IF.
```

**VSAM Error Handling:**
```cobol
IF WS-RESP NOT = DFHRESP(NORMAL)
    EVALUATE WS-RESP
        WHEN DFHRESP(NOTFND)
            MOVE 'Account not found' TO WS-ERROR-MSG
        WHEN DFHRESP(NOTOPEN)
            MOVE 'File not open' TO WS-ERROR-MSG
        WHEN OTHER
            MOVE 'VSAM error' TO WS-ERROR-MSG
    END-EVALUATE
    PERFORM LOG-ERROR
END-IF.
```

### Performance Considerations

**Queue Depth Monitoring:**
- Monitor request queue depth
- Implement queue depth alerts
- Scale processing based on load

**Message Persistence:**
- Use persistent messages for critical data
- Non-persistent for high-volume, low-priority

**Connection Pooling:**
- Reuse MQ connections
- Configure appropriate connection limits

## Integration Patterns

### Request/Response Pattern

**Synchronous-Style Communication:**
1. Client sends request
2. Client waits for response
3. Server processes request
4. Server sends response
5. Client receives response

**Implementation:**
- Use correlation ID to match requests/responses
- Set appropriate timeout values
- Handle timeout scenarios

### Publish/Subscribe Pattern

**Extension Possibility:**
- Define topics for account updates
- Publish account changes
- Subscribers receive notifications

### Message Transformation

**VSAM to Message:**
```cobol
MOVE ACCT-ID TO MSG-ACCT-ID.
MOVE ACCT-CURR-BAL TO MSG-BALANCE.
MOVE ACCT-CREDIT-LIMIT TO MSG-CREDIT-LIMIT.
```

**Message to VSAM:**
```cobol
MOVE MSG-ACCT-ID TO ACCT-ID.
MOVE MSG-BALANCE TO ACCT-CURR-BAL.
MOVE MSG-CREDIT-LIMIT TO ACCT-CREDIT-LIMIT.
```

## Troubleshooting

### Common Issues

**Issue: Messages not appearing in queue**
- Check queue definition
- Verify queue is not full (MAXDEPTH)
- Check MQ channel status
- Verify application has PUT authority

**Issue: Cannot read from queue**
- Verify queue exists
- Check GET authority
- Ensure messages are available
- Check queue is not disabled

**Issue: CICS-MQ connection fails**
- Verify MQ queue manager is running
- Check CICS-MQ bridge configuration
- Verify MQCONN resource is installed
- Check MQ channel is active

**Issue: Account not found**
- Verify account number format (11 digits)
- Check ACCTDAT file is open
- Verify account exists in VSAM
- Check file access permissions

### Debugging Tips

**Enable MQ Tracing:**
```
ALTER QMGR TRACE(ON)
```

**Check Queue Status:**
```
DISPLAY QUEUE(CARDDEMO.REQUEST.QUEUE) ALL
```

**Monitor Queue Depth:**
```
DISPLAY QUEUE(CARDDEMO.REQUEST.QUEUE) CURDEPTH
```

**Check CICS Resources:**
```
CEMT INQUIRE MQCONN
CEMT INQUIRE MQQUEUE(CARDREQ)
```

---

**Navigation**: [Home](Home.md) | [Optional Modules](Home.md#optional-modules) | [Installation Guide](Installation-Guide.md) | [Technical Components](Technical-Components.md)
