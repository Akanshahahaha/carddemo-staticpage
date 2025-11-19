# Contributing to CardDemo

Thank you for your interest in contributing to CardDemo! This guide will help you get started with contributing to the project.

## Table of Contents
- [Code of Conduct](#code-of-conduct)
- [How to Contribute](#how-to-contribute)
- [Development Setup](#development-setup)
- [Coding Standards](#coding-standards)
- [Testing Guidelines](#testing-guidelines)
- [Submitting Changes](#submitting-changes)
- [Documentation](#documentation)
- [Community](#community)

## Code of Conduct

By participating in this project, you agree to maintain a respectful and inclusive environment for all contributors.

### Our Standards

- Be respectful and inclusive
- Welcome newcomers and help them learn
- Focus on constructive feedback
- Respect differing viewpoints and experiences
- Accept responsibility and apologize for mistakes

## How to Contribute

### Types of Contributions

**Code Contributions:**
- Bug fixes
- New features
- Performance improvements
- Code refactoring

**Documentation:**
- Wiki improvements
- Code comments
- README updates
- Tutorial creation

**Testing:**
- Test case creation
- Bug reporting
- Test automation

**Community:**
- Answering questions
- Reviewing pull requests
- Sharing use cases

### Getting Started

1. **Fork the Repository**
   ```bash
   # Fork on GitHub, then clone your fork
   git clone https://github.com/YOUR-USERNAME/aws-mainframe-modernization-carddemo.git
   cd aws-mainframe-modernization-carddemo
   ```

2. **Create a Branch**
   ```bash
   git checkout -b feature/your-feature-name
   # or
   git checkout -b fix/your-bug-fix
   ```

3. **Make Your Changes**
   - Follow coding standards
   - Add tests if applicable
   - Update documentation

4. **Test Your Changes**
   - Compile all programs
   - Run existing tests
   - Test manually in CICS

5. **Commit Your Changes**
   ```bash
   git add .
   git commit -m "Description of your changes"
   ```

6. **Push to Your Fork**
   ```bash
   git push origin feature/your-feature-name
   ```

7. **Create Pull Request**
   - Go to the original repository
   - Click "New Pull Request"
   - Select your branch
   - Fill in the PR template

## Development Setup

### Prerequisites

- Access to z/OS mainframe environment
- CICS Transaction Server
- COBOL compiler
- TSO/ISPF access
- Git client

### Local Development

1. **Clone Repository**
   ```bash
   git clone https://github.com/aws-samples/aws-mainframe-modernization-carddemo.git
   ```

2. **Review Structure**
   ```
   app/
   ├── cbl/          - COBOL programs
   ├── cpy/          - Copybooks
   ├── bms/          - BMS maps
   ├── jcl/          - JCL jobs
   ├── asm/          - Assembler programs
   └── data/         - Sample data
   ```

3. **Set Up Environment**
   - Create datasets with your HLQ
   - Upload source code
   - Compile programs
   - Define CICS resources

### Development Workflow

1. **Make Changes Locally**
   - Edit files in your local repository
   - Test syntax if possible

2. **Upload to Mainframe**
   - Transfer files to mainframe datasets
   - Use text mode for source code
   - Use binary mode for data

3. **Compile and Test**
   - Compile modified programs
   - Test in CICS
   - Run batch jobs if applicable

4. **Iterate**
   - Fix issues
   - Recompile
   - Retest

## Coding Standards

### COBOL Standards

**Program Structure:**
```cobol
IDENTIFICATION DIVISION.
PROGRAM-ID. PROGRAMNAME.
AUTHOR. YOUR-NAME.
DATE-WRITTEN. YYYY-MM-DD.

ENVIRONMENT DIVISION.

DATA DIVISION.
WORKING-STORAGE SECTION.
LINKAGE SECTION.

PROCEDURE DIVISION.
    PERFORM MAIN-PROCESS
    GOBACK.

MAIN-PROCESS.
    [Main logic here]
    .
```

**Naming Conventions:**
- Programs: CO prefix for online, CB for batch
- Paragraphs: Descriptive names with hyphens
- Variables: WS- prefix for working storage
- Constants: Use 88-level conditions

**Code Style:**
- Indent 4 spaces per level
- Use structured programming (no GO TO except GOBACK)
- One statement per line
- Meaningful variable names
- Comments for complex logic

**Error Handling:**
```cobol
EXEC CICS READ
    FILE('ACCTDAT')
    INTO(ACCOUNT-RECORD)
    RIDFLD(WS-ACCOUNT-ID)
    RESP(WS-RESP)
END-EXEC.

EVALUATE WS-RESP
    WHEN DFHRESP(NORMAL)
        PERFORM PROCESS-ACCOUNT
    WHEN DFHRESP(NOTFND)
        PERFORM ACCOUNT-NOT-FOUND
    WHEN OTHER
        PERFORM HANDLE-ERROR
END-EVALUATE.
```

### JCL Standards

**Job Card:**
```jcl
//JOBNAME  JOB (ACCT),'DESCRIPTION',
//         CLASS=A,
//         MSGCLASS=X,
//         MSGLEVEL=(1,1),
//         NOTIFY=&SYSUID
```

**Step Structure:**
```jcl
//STEP01   EXEC PGM=PROGRAMNAME
//STEPLIB  DD DSN=LOAD.LIBRARY,DISP=SHR
//SYSOUT   DD SYSOUT=*
//INPUT    DD DSN=INPUT.FILE,DISP=SHR
//OUTPUT   DD DSN=OUTPUT.FILE,DISP=(NEW,CATLG,DELETE),
//            SPACE=(CYL,(10,5)),
//            DCB=(RECFM=FB,LRECL=80,BLKSIZE=0)
```

### BMS Standards

**Map Definition:**
```
MAPNAME  DFHMSD TYPE=&SYSPARM,                                         X
               MODE=INOUT,                                             X
               LANG=COBOL,                                             X
               CTRL=(FREEKB,FRSET),                                    X
               STORAGE=AUTO,                                           X
               TIOAPFX=YES

MAP01    DFHMDI SIZE=(24,80),                                          X
               LINE=1,                                                 X
               COLUMN=1

         DFHMDF POS=(1,1),                                             X
               LENGTH=20,                                              X
               ATTRB=(PROT,BRT),                                       X
               INITIAL='SCREEN TITLE'
```

### Documentation Standards

**Program Header:**
```cobol
      *****************************************************************
      * PROGRAM NAME: PROGRAMNAME
      * DESCRIPTION:  Brief description of program purpose
      * AUTHOR:       Your Name
      * DATE:         YYYY-MM-DD
      *
      * INPUTS:       List of input files/parameters
      * OUTPUTS:      List of output files
      *
      * CHANGE HISTORY:
      * DATE       AUTHOR      DESCRIPTION
      * ---------- ----------- ---------------------------------------
      * YYYY-MM-DD Your Name   Initial version
      *****************************************************************
```

**Paragraph Comments:**
```cobol
      *****************************************************************
      * PARAGRAPH: PROCESS-ACCOUNT
      * PURPOSE:   Process account record and update balance
      * CALLED BY: MAIN-PROCESS
      *****************************************************************
       PROCESS-ACCOUNT.
```

## Testing Guidelines

### Unit Testing

**Test Each Program:**
- Test with valid data
- Test with invalid data
- Test boundary conditions
- Test error conditions

**Test Checklist:**
- [ ] Program compiles without errors
- [ ] Program links successfully
- [ ] All file operations work
- [ ] Error handling works correctly
- [ ] Screen displays correctly (for online programs)
- [ ] Data validation works
- [ ] Business logic is correct

### Integration Testing

**Test Interactions:**
- Test transaction flows
- Test file sharing
- Test batch job sequences
- Test optional module integration

### Regression Testing

**Before Submitting:**
- Run existing test cases
- Verify no existing functionality broken
- Test related components
- Check performance impact

### Test Documentation

Document your tests:
```
Test Case: TC001 - Account View
Description: Verify account view displays correct information
Prerequisites: Account 00000000001 exists
Steps:
  1. Sign on as USER0001
  2. Enter transaction CAVW
  3. Enter account number 00000000001
  4. Press Enter
Expected Result: Account details display correctly
Actual Result: [Pass/Fail]
```

## Submitting Changes

### Pull Request Process

1. **Update Documentation**
   - Update README if needed
   - Update wiki pages
   - Add code comments

2. **Create Pull Request**
   - Use descriptive title
   - Fill in PR template
   - Reference related issues

3. **PR Template**
   ```markdown
   ## Description
   Brief description of changes
   
   ## Type of Change
   - [ ] Bug fix
   - [ ] New feature
   - [ ] Documentation update
   - [ ] Performance improvement
   
   ## Testing
   - [ ] Unit tests pass
   - [ ] Integration tests pass
   - [ ] Manual testing completed
   
   ## Checklist
   - [ ] Code follows style guidelines
   - [ ] Documentation updated
   - [ ] No new warnings
   - [ ] Tests added/updated
   ```

4. **Code Review**
   - Address reviewer comments
   - Make requested changes
   - Update PR as needed

5. **Merge**
   - Maintainer will merge when approved
   - Delete your branch after merge

### Commit Message Guidelines

**Format:**
```
<type>: <subject>

<body>

<footer>
```

**Types:**
- feat: New feature
- fix: Bug fix
- docs: Documentation
- style: Formatting
- refactor: Code restructuring
- test: Adding tests
- chore: Maintenance

**Example:**
```
feat: Add transaction search by date range

- Added date range input fields to CT00 screen
- Updated COTRN00C program to filter by date
- Added date validation logic
- Updated user guide documentation

Closes #123
```

### Branch Naming

**Convention:**
- feature/description - New features
- fix/description - Bug fixes
- docs/description - Documentation
- refactor/description - Code refactoring

**Examples:**
- feature/add-transaction-search
- fix/account-balance-calculation
- docs/update-installation-guide
- refactor/optimize-vsam-access

## Documentation

### Wiki Updates

When adding features:
1. Update relevant wiki pages
2. Add new pages if needed
3. Update navigation links
4. Add examples and screenshots

### Code Comments

**When to Comment:**
- Complex algorithms
- Business rules
- Non-obvious logic
- Workarounds
- TODO items

**When Not to Comment:**
- Obvious code
- Self-explanatory logic
- Redundant information

### README Updates

Update README.md for:
- New features
- Changed requirements
- New dependencies
- Installation changes

## Community

### Getting Help

**Resources:**
- GitHub Issues
- Wiki documentation
- Code comments
- Community discussions

**Asking Questions:**
- Search existing issues first
- Provide context and details
- Include error messages
- Share relevant code snippets

### Reporting Bugs

**Bug Report Template:**
```markdown
## Bug Description
Clear description of the bug

## Steps to Reproduce
1. Step one
2. Step two
3. Step three

## Expected Behavior
What should happen

## Actual Behavior
What actually happens

## Environment
- z/OS version:
- CICS version:
- CardDemo version:

## Additional Context
Screenshots, logs, etc.
```

### Feature Requests

**Feature Request Template:**
```markdown
## Feature Description
Clear description of the feature

## Use Case
Why is this feature needed?

## Proposed Solution
How should it work?

## Alternatives Considered
Other approaches considered

## Additional Context
Mockups, examples, etc.
```

### Code Review

**As a Reviewer:**
- Be constructive and respectful
- Focus on code quality
- Suggest improvements
- Approve when satisfied

**As an Author:**
- Respond to all comments
- Make requested changes
- Explain your decisions
- Thank reviewers

## Recognition

Contributors will be recognized in:
- CONTRIBUTORS.md file
- Release notes
- Project documentation

Thank you for contributing to CardDemo!

---

**Navigation**: [Home](Home.md) | [Development Guide](Development-Guide.md) | [Testing Guide](Testing-Guide.md)
