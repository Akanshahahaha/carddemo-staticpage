# CardDemo Wiki - Mainframe Credit Card Management Application

![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)
![License](https://img.shields.io/badge/license-Apache%202.0-green.svg)

Welcome to the comprehensive wiki for the CardDemo mainframe application. This wiki provides in-depth documentation for understanding, installing, configuring, and extending the CardDemo credit card management system.

## What is CardDemo?

CardDemo is a comprehensive mainframe application that simulates a credit card management system. Designed specifically to showcase AWS and partner technologies for mainframe migration and modernization scenarios, it provides a realistic environment for testing various modernization approaches including discovery, migration, performance testing, service enablement, and more.

The application intentionally incorporates various coding styles and patterns to exercise analysis, transformation, and migration tooling across different mainframe programming paradigms.

## Quick Links

### Getting Started
- [Installation Guide](Installation-Guide.md) - Complete setup instructions
- [Quick Start](Quick-Start.md) - Get up and running quickly
- [Architecture Overview](Architecture.md) - System design and components

### User Documentation
- [User Guide](User-Guide.md) - Regular user functions and workflows
- [Admin Guide](Admin-Guide.md) - Administrative functions
- [Application Screens](Application-Screens.md) - Screen-by-screen reference

### Technical Documentation
- [Technical Components](Technical-Components.md) - Programs, files, and resources
- [Data Models](Data-Models.md) - Database structures and file layouts
- [Batch Processing](Batch-Processing.md) - Batch job execution guide
- [CICS Configuration](CICS-Configuration.md) - CICS resource setup

### Optional Modules
- [Credit Card Authorizations (IMS-DB2-MQ)](Optional-Module-Authorization.md) - Real-time authorization processing
- [Transaction Type Management (DB2)](Optional-Module-Transaction-Type.md) - DB2-based transaction type maintenance
- [Account Extractions (VSAM-MQ)](Optional-Module-VSAM-MQ.md) - MQ integration for data extraction

### Development & Extension
- [Development Guide](Development-Guide.md) - Building and compiling programs
- [Extending CardDemo](Extending-CardDemo.md) - Adding new features
- [Testing Guide](Testing-Guide.md) - Testing strategies and approaches

### Reference
- [Troubleshooting](Troubleshooting.md) - Common issues and solutions
- [FAQ](FAQ.md) - Frequently asked questions
- [Glossary](Glossary.md) - Mainframe and CardDemo terminology
- [API Reference](API-Reference.md) - Program interfaces and copybooks

## Key Features

### Core Functionality
- **Customer Management**: Comprehensive customer data management with VSAM storage
- **Account Management**: Credit card account creation, viewing, and updates
- **Card Management**: Credit card issuance and maintenance
- **Transaction Processing**: Online and batch transaction handling
- **Bill Payment**: Payment processing and account reconciliation
- **Reporting**: Transaction reports and statements

### Technology Stack
- **COBOL**: Primary programming language for business logic
- **CICS**: Transaction processing and online user interface
- **VSAM (KSDS with AIX)**: Primary data storage with indexed access
- **JCL**: Batch job control and execution
- **RACF**: Security and access control
- **ASSEMBLER**: System-level utilities (MVSWAIT, COBDATFT)

### Optional Technologies
- **DB2**: Relational database for transaction types and fraud tracking
- **IMS DB**: Hierarchical database for authorization data
- **MQ**: Message queuing for asynchronous processing
- **Advanced Data Formats**: COMP, COMP-3, Zoned Decimal, Signed, Unsigned
- **Additional Dataset Types**: VSAM (ESDS/RRDS), GDG, PDS

## Application Statistics

- **29 COBOL Programs** in the base application
- **33 JCL Jobs** for batch processing
- **14 Online Transactions** for user interaction
- **Multiple Optional Modules** for extended functionality
- **Comprehensive Test Data** included for all scenarios

## Use Cases

CardDemo is designed to support the following mainframe modernization scenarios:

1. **Application Discovery and Analysis**: Understand mainframe application structure and dependencies
2. **Migration Assessment**: Evaluate complexity and effort for migration projects
3. **Modernization Strategy Development**: Test different approaches to modernization
4. **Performance Testing**: Benchmark performance of migration tools and platforms
5. **System Augmentation**: Add modern capabilities to existing mainframe applications
6. **Service Enablement**: Extract and expose mainframe services
7. **Test Creation and Automation**: Develop automated testing strategies

## Documentation Structure

This wiki is organized into several major sections:

### Installation and Setup
Complete guides for installing CardDemo in your mainframe environment, including prerequisites, dataset creation, program compilation, and CICS configuration.

### User Documentation
End-user focused documentation covering all user-facing functions, screens, and workflows for both regular users and administrators.

### Technical Documentation
In-depth technical information about the application architecture, components, data structures, and processing flows.

### Optional Modules
Detailed documentation for each optional module, including installation, configuration, and usage instructions.

### Development and Extension
Guides for developers who want to modify, extend, or integrate with CardDemo.

### Reference Materials
Quick reference guides, troubleshooting information, and supplementary materials.

## Getting Help

- **Issues**: Report bugs or request features in the GitHub repository
- **Questions**: Consult the [FAQ](FAQ.md) or [Troubleshooting](Troubleshooting.md) pages
- **Community**: Engage with other users and contributors

## Contributing

CardDemo is a community resource. Contributions are welcome in the form of:
- Code enhancements and bug fixes
- Documentation improvements
- Additional test scenarios
- New optional modules

See the [Contributing Guide](Contributing.md) for details on how to contribute.

## License

CardDemo is released under the Apache 2.0 license as a community resource for mainframe modernization.

---

**Navigation**: [Installation Guide](Installation-Guide.md) | [Architecture](Architecture.md) | [User Guide](User-Guide.md) | [Technical Components](Technical-Components.md)
