# Glossary

Comprehensive glossary of mainframe and CardDemo terminology.

## A

**ABEND**
Abnormal End. A program or transaction termination due to an error condition. Common CICS abends include ASRA, AFCF, AEIV.

**AIX (Alternate Index)**
A VSAM feature that provides an alternate key path to access records in a KSDS. CardDemo uses AIX on the transaction file for card number access.

**AMP (Access Method Parameters)**
JCL parameter used to specify VSAM buffer allocations and other access method options.

**Assembler**
Low-level programming language used for system utilities. CardDemo includes assembler programs MVSWAIT and COBDATFT.

## B

**Batch Processing**
Non-interactive processing of data in groups or batches, typically scheduled during off-peak hours. CardDemo batch jobs include POSTTRAN, INTCALC, CREASTMT.

**BMS (Basic Mapping Support)**
CICS facility for defining screen layouts and managing terminal I/O. CardDemo uses BMS maps for all user interfaces.

**BMP (Batch Message Processing)**
IMS batch processing mode that allows batch programs to access IMS databases.

**BUFNI/BUFND**
VSAM buffer parameters. BUFNI specifies index buffers, BUFND specifies data buffers.

## C

**CA (Control Area)**
A group of control intervals in a VSAM file. Used for space allocation and performance tuning.

**CICS (Customer Information Control System)**
IBM's transaction processing system. CardDemo runs as a CICS application.

**CI (Control Interval)**
The unit of data transfer between VSAM and a program. Similar to a block in non-VSAM files.

**COBOL (Common Business-Oriented Language)**
Primary programming language for CardDemo business logic.

**COMMAREA (Communication Area)**
Data area used to pass information between CICS programs or between pseudo-conversational transactions.

**COMP-3 (Computational-3)**
Packed decimal data format. Efficient storage for numeric data with decimal places.

**Copybook**
COBOL source code that defines data structures, included in programs via COPY statement.

**Cursor**
DB2 mechanism for processing multiple rows from a query result set. Used in COTRTLIC for paging.

## D

**DASD (Direct Access Storage Device)**
Mainframe disk storage.

**DB2**
IBM's relational database management system. Used in CardDemo optional modules.

**DBD (Database Description)**
IMS database definition. Defines the structure of an IMS database.

**DFHCOMMAREA**
Standard CICS COMMAREA name used for passing data between programs.

**DFSORT**
IBM's high-performance sort utility used in batch processing.

## E

**EBCDIC (Extended Binary Coded Decimal Interchange Code)**
Character encoding used on mainframes. Different from ASCII used on distributed systems.

**Embedded SQL**
SQL statements embedded directly in COBOL programs, used in DB2 modules.

**ESDS (Entry-Sequenced Data Set)**
VSAM file organization where records are stored in the order they are written.

## F

**FICO Score**
Credit score used in customer records (300-850 range).

## G

**GDG (Generation Data Group)**
A group of related datasets with automatic version control. CardDemo uses GDGs for backups.

**GU (Get Unique)**
IMS call to retrieve a specific segment.

## H

**HIDAM (Hierarchical Indexed Direct Access Method)**
IMS database organization combining hierarchical structure with direct access. Used in authorization module.

**HLQ (High Level Qualifier)**
First part of a mainframe dataset name, typically identifying the owner or project.

**Host Variable**
COBOL variable used in embedded SQL statements to exchange data with DB2.

## I

**IDCAMS**
IBM utility program for VSAM file operations (define, delete, repro, etc.).

**IMS (Information Management System)**
IBM's hierarchical database and transaction processing system.

**ISPF (Interactive System Productivity Facility)**
TSO-based interface for mainframe development and operations.

## J

**JCL (Job Control Language)**
Language for defining batch jobs on z/OS.

**JES (Job Entry Subsystem)**
z/OS component that manages batch job execution (JES2 or JES3).

## K

**KSDS (Key-Sequenced Data Set)**
VSAM file organization where records are stored in key sequence. Primary file type in CardDemo.

## L

**LRECL (Logical Record Length)**
Length of a logical record in a dataset.

**Luhn Algorithm**
Checksum algorithm used to validate credit card numbers.

## M

**Mapset**
Collection of BMS maps, typically one per program.

**MQ (Message Queue)**
IBM's message queuing middleware for asynchronous communication.

**MXT (Maximum Tasks)**
CICS parameter controlling maximum number of concurrent tasks.

## N

**NEWCOPY**
CICS command to load a fresh copy of a program into memory.

## O

**OCCURS**
COBOL clause defining an array or table.

**OCCURS DEPENDING ON**
COBOL clause for variable-length arrays.

## P

**PDS (Partitioned Data Set)**
Dataset containing multiple members, like a directory of files.

**PF Key (Program Function Key)**
Function keys (PF1-PF24) used for navigation and commands in CICS applications.

**PSB (Program Specification Block)**
IMS definition of database access requirements for a program.

**Pseudo-conversational**
CICS programming technique where a transaction terminates between user interactions, freeing resources.

## Q

**Queue**
MQ message queue for storing and forwarding messages.

## R

**RACF (Resource Access Control Facility)**
IBM's security system for z/OS.

**RECFM (Record Format)**
Format of records in a dataset (F=Fixed, V=Variable, B=Blocked).

**REDEFINES**
COBOL clause allowing multiple definitions of the same storage area.

**RMF (Resource Measurement Facility)**
z/OS performance monitoring tool.

**RRDS (Relative Record Data Set)**
VSAM file organization where records are accessed by relative record number.

## S

**SDSF (System Display and Search Facility)**
Tool for viewing and managing batch jobs and system output.

**Segment**
Unit of data in an IMS hierarchical database.

**SHAREOPTIONS**
VSAM parameter controlling file sharing between jobs and regions.

**SMF (System Management Facilities)**
z/OS component that records system activity for accounting and performance analysis.

**SQL (Structured Query Language)**
Language for accessing relational databases like DB2.

**SQLCA (SQL Communication Area)**
Data structure containing DB2 return codes and diagnostic information.

**SQLCODE**
Return code from DB2 SQL operation.

**SSA (Segment Search Argument)**
IMS call parameter specifying which segment to access.

**STEPLIB**
JCL DD statement specifying libraries to search for programs.

**Symbolic Map**
COBOL copybook generated from BMS map definition.

**SYNCPOINT**
CICS command to commit database changes.

## T

**TSO (Time Sharing Option)**
Interactive interface to z/OS for development and operations.

## U

**UNSTRING**
COBOL verb for parsing delimited strings.

## V

**VSAM (Virtual Storage Access Method)**
IBM's file access method for mainframe data storage. Primary storage in CardDemo.

## W

**Working Storage**
COBOL section for program variables and data structures.

## Z

**z/OS**
IBM's mainframe operating system.

---

## CardDemo-Specific Terms

**ACCTDAT**
Account master VSAM file containing account information.

**CARDDAT**
Card master VSAM file containing credit card information.

**CARDXREF**
Cross-reference VSAM file linking accounts, cards, and customers.

**CUSTDAT**
Customer master VSAM file containing customer information.

**TRANSACT**
Transaction VSAM file containing transaction history.

**USRSEC**
User security VSAM file containing user credentials and permissions.

**TRANTYPE**
Transaction type reference file.

**TRANCATG**
Transaction category reference file.

## Transaction Codes

**CC00**
Sign-on transaction.

**CM00**
Main menu transaction.

**CAVW**
Account view transaction.

**CAUP**
Account update transaction.

**CCLI**
Card list transaction.

**CCDL**
Card detail/view transaction.

**CCUP**
Card update transaction.

**CT00**
Transaction list transaction.

**CT01**
Transaction view transaction.

**CT02**
Transaction add transaction.

**CR00**
Reports transaction.

**CB00**
Bill payment transaction.

**CA00**
Admin menu transaction.

**CU00-CU03**
User management transactions (list, add, update, delete).

**CTLI**
Transaction type list (DB2 module).

**CTTU**
Transaction type add/edit (DB2 module).

**CPVS**
Pending authorization summary (IMS-DB2-MQ module).

**CPVD**
Pending authorization details (IMS-DB2-MQ module).

**CP00**
Authorization request processor (IMS-DB2-MQ module).

**CDRD**
Date inquiry via MQ (VSAM-MQ module).

**CDRA**
Account inquiry via MQ (VSAM-MQ module).

## Program Naming Conventions

**CO prefix**
CICS Online program (e.g., COSGN00C, COACTVWC).

**CB prefix**
COBOL Batch program (e.g., CBTRN02C, CBACT04C).

**C suffix**
COBOL program.

**Y suffix**
Copybook (e.g., CSUSR01Y, CVACT01Y).

## Job Names

**CLOSEFIL**
Close VSAM files for batch processing.

**OPENFIL**
Open VSAM files after batch processing.

**POSTTRAN**
Post daily transactions to accounts.

**INTCALC**
Calculate monthly interest charges.

**CREASTMT**
Generate monthly account statements.

**TRANBKP**
Backup transaction file.

**TRANEXTR**
Extract transaction types from DB2.

**CBPAUP0J**
Purge expired authorizations.

---

**Navigation**: [Home](Home.md) | [FAQ](FAQ.md) | [Technical Components](Technical-Components.md) | [Data Models](Data-Models.md)
