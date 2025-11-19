# Frequently Asked Questions (FAQ)

Common questions and answers about CardDemo.

## Table of Contents
- [General Questions](#general-questions)
- [Installation Questions](#installation-questions)
- [Usage Questions](#usage-questions)
- [Technical Questions](#technical-questions)
- [Optional Modules](#optional-modules)
- [Troubleshooting](#troubleshooting)

## General Questions

### What is CardDemo?

CardDemo is a comprehensive mainframe application that simulates a credit card management system. It's designed to showcase AWS and partner technologies for mainframe migration and modernization scenarios, providing a realistic environment for testing various modernization approaches.

### Who should use CardDemo?

CardDemo is ideal for:
- Organizations planning mainframe modernization
- Teams evaluating migration tools and strategies
- Developers learning mainframe technologies
- Training and education purposes
- Testing modernization approaches

### What technologies does CardDemo use?

**Core Technologies:**
- COBOL (business logic)
- CICS (transaction processing)
- VSAM (data storage)
- JCL (batch processing)
- BMS (screen definitions)
- Assembler (utilities)

**Optional Technologies:**
- DB2 (relational database)
- IMS DB (hierarchical database)
- MQ (message queuing)

### Is CardDemo production-ready?

CardDemo is designed as a demonstration and testing application, not for production use. It provides realistic functionality for modernization testing but should not be used to process real customer data or financial transactions.

### What license is CardDemo released under?

CardDemo is released under the Apache 2.0 license as a community resource.

### Can I modify CardDemo for my needs?

Yes! CardDemo is open source and can be modified, extended, and customized for your specific use cases. Contributions back to the community are welcome.

## Installation Questions

### How long does installation take?

Base installation typically takes 2-4 hours, depending on your familiarity with mainframe systems and your environment. Optional modules add 1-2 hours each.

### What are the minimum system requirements?

**Required:**
- z/OS operating system
- CICS Transaction Server
- VSAM support
- JCL execution capability
- TSO/ISPF access

**Optional (for modules):**
- DB2 subsystem
- IMS DB subsystem
- IBM MQ

### Do I need special permissions to install CardDemo?

Yes, you need:
- Authority to create datasets
- Authority to define CICS resources
- Authority to submit JCL jobs
- File transfer access
- Optionally: DB2, IMS, MQ administration rights

### Can I install CardDemo on AWS Mainframe Modernization?

Yes! CardDemo is specifically designed to work with AWS Mainframe Modernization services and can be deployed on AWS infrastructure.

### What if I don't have DB2 or IMS?

The base CardDemo application works without DB2 or IMS. These are only required for optional modules. You can install and use the base application with just CICS and VSAM.

### Can I use a different High Level Qualifier (HLQ)?

Yes, you can use any HLQ that follows your organization's naming conventions. Just update all JCL and configuration to use your chosen HLQ.

## Usage Questions

### What are the default login credentials?

**Admin User:**
- User ID: ADMIN001
- Password: PASSWORD

**Regular User:**
- User ID: USER0001
- Password: PASSWORD

**Important:** Change these default passwords immediately after installation.

### How do I reset a user password?

1. Sign on as admin
2. Access Admin Menu (CA00)
3. Select Update User (CU02)
4. Enter user ID
5. Press PF5 to reset password
6. Enter new password

### Can I add my own test data?

Yes! You can:
- Add users via the Admin Menu
- Add transactions via CT02
- Modify sample data files before loading
- Create custom data generation scripts

### How do I generate reports?

Use transaction CR00 (Reports) to generate various reports:
- Account summary
- Transaction detail
- Category summary
- Merchant summary

### What transaction types are supported?

**Standard Types:**
- PUR: Purchase
- PAY: Payment
- ADJ: Adjustment
- FEE: Fee
- INT: Interest
- REF: Refund
- RET: Return

Additional types can be added via the Transaction Type Management module (DB2).

### How often should batch jobs run?

**Recommended Schedule:**
- Daily: Transaction posting, backups
- Monthly: Interest calculation, statement generation
- As needed: Data maintenance, reports

## Technical Questions

### What COBOL version is required?

CardDemo uses COBOL 85 or later. Most modern mainframe COBOL compilers are compatible.

### Can I use CardDemo with CICS TS 5.x?

Yes, CardDemo is compatible with CICS Transaction Server 5.x and later versions.

### What VSAM file organization does CardDemo use?

CardDemo primarily uses KSDS (Key-Sequenced Data Sets) with some AIX (Alternate Index) for secondary key access. Optional features may use ESDS and RRDS.

### How is data security handled?

- User credentials stored in USRSEC VSAM file
- Passwords encrypted
- Role-based access control (User vs Admin)
- RACF integration for system-level security
- Transaction audit trail

### Can CardDemo handle concurrent users?

Yes, CardDemo is designed for multi-user CICS environments with proper file sharing (SHAREOPTIONS 2,3) and transaction management.

### What is the maximum transaction volume?

CardDemo can handle typical test volumes (thousands of transactions per day). Performance depends on your mainframe resources and configuration.

### How do I backup CardDemo data?

Use the provided batch jobs:
- TRANBKP: Backup transaction file
- GDG: Generation Data Groups for versioning
- Regular VSAM backups via IDCAMS REPRO

### Can I integrate CardDemo with external systems?

Yes! The VSAM-MQ optional module demonstrates integration patterns. You can extend CardDemo to integrate with:
- Web services
- REST APIs
- Other mainframe applications
- Cloud services

## Optional Modules

### Do I need all optional modules?

No, optional modules are independent. Install only the modules you need for your use case.

### Can I install optional modules later?

Yes, optional modules can be installed at any time after the base application is operational.

### What does the IMS-DB2-MQ module demonstrate?

This module shows:
- Real-time authorization processing
- IMS hierarchical database usage
- DB2 relational database integration
- MQ message queuing
- Multi-database transactions

### What does the DB2 Transaction Type module demonstrate?

This module shows:
- DB2 embedded SQL in COBOL
- Cursor processing (forward/backward)
- CRUD operations
- Referential integrity
- Data synchronization between DB2 and VSAM

### What does the VSAM-MQ module demonstrate?

This module shows:
- MQ request/response patterns
- VSAM data extraction
- Message correlation
- Asynchronous processing

### Can I create my own optional modules?

Yes! CardDemo's modular architecture makes it easy to add new features. Follow the existing module patterns and contribute back to the community.

## Troubleshooting

### Why can't I sign on?

**Common causes:**
- Wrong user ID or password
- Account locked (3 failed attempts)
- Account inactive
- USRSEC file not open

**Solution:** Check credentials, verify account status, ensure USRSEC file is open.

### Why is my transaction failing with AFCF?

AFCF indicates a file control error. Usually means:
- File not defined in CICS
- File not open
- File not found

**Solution:** Check file status with `CEMT INQ FILE(filename)` and open if needed.

### Why are batch jobs failing?

**Common causes:**
- Files not closed (need CLOSEFIL)
- Wrong dataset names
- Insufficient space
- Missing programs

**Solution:** Review job output, check return codes, verify file status.

### Why is performance slow?

**Common causes:**
- VSAM file fragmentation
- Insufficient buffers
- High system load
- Inefficient programs

**Solution:** Reorganize files, tune buffers, optimize programs, check system resources.

### Where can I find error codes?

See the [Troubleshooting Guide](Troubleshooting.md#error-code-reference) for comprehensive error code reference including:
- CICS ABEND codes
- JCL return codes
- VSAM return codes
- DB2 SQL codes

### How do I enable debug mode?

For CICS programs:
- Use CEDF (CICS Execution Diagnostic Facility)
- Add display statements in COBOL
- Review transaction dumps

For batch programs:
- Add DISPLAY statements
- Check SYSOUT output
- Use COBOL debugging options

### What if I need more help?

1. Check the [Troubleshooting Guide](Troubleshooting.md)
2. Review relevant wiki pages
3. Check CICS and z/OS logs
4. Raise an issue in the GitHub repository
5. Consult the mainframe community

## Performance and Scalability

### How many users can CardDemo support?

CardDemo can support dozens of concurrent users in a typical test environment. Actual capacity depends on:
- CICS region size
- VSAM buffer allocation
- System resources
- Transaction mix

### Can CardDemo scale to production volumes?

CardDemo demonstrates production-ready patterns but is sized for testing. For production volumes, you would need to:
- Increase VSAM file sizes
- Tune CICS parameters
- Optimize batch windows
- Add monitoring and alerting

### How do I tune CardDemo for better performance?

**Key tuning areas:**
- VSAM buffer allocation (BUFNI/BUFND)
- CICS MXT and MAXTASK
- File reorganization schedule
- Batch job scheduling
- Program optimization

## Customization and Extension

### Can I add new transactions?

Yes! Follow these steps:
1. Create BMS map for screen
2. Write COBOL program
3. Compile and link
4. Define CICS transaction
5. Add to menu if desired

### Can I add new fields to records?

Yes, but requires:
1. Update copybook
2. Redefine VSAM file
3. Reload data
4. Recompile programs
5. Test thoroughly

### Can I change the screen layouts?

Yes, modify the BMS maps and recompile. Remember to:
- Update symbolic copybooks
- Recompile programs using the maps
- Test all affected transactions

### Can I integrate with my existing systems?

Yes! CardDemo provides integration patterns via:
- MQ messaging
- Batch file exchange
- DB2 data sharing
- Custom interfaces

## Migration and Modernization

### What modernization patterns does CardDemo demonstrate?

- Application discovery and analysis
- Rehosting (lift and shift)
- Replatforming (minimal changes)
- Refactoring (code modernization)
- Service extraction
- Data migration

### Can I use CardDemo to test migration tools?

Yes! CardDemo is specifically designed for testing:
- Code analysis tools
- Migration assessment tools
- Automated conversion tools
- Performance testing tools
- Integration testing

### What AWS services work with CardDemo?

- AWS Mainframe Modernization
- Amazon RDS (for DB2 migration)
- Amazon MQ (for MQ migration)
- AWS Database Migration Service
- AWS Application Migration Service

### How do I measure migration success?

Use CardDemo to test:
- Functional equivalence
- Performance comparison
- Data integrity
- Integration points
- User experience

---

**Navigation**: [Home](Home.md) | [Troubleshooting](Troubleshooting.md) | [Glossary](Glossary.md) | [Installation Guide](Installation-Guide.md)
