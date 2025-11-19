# Admin Guide

Complete guide for CardDemo administrators. This guide covers all administrative functions, user management, system configuration, and maintenance tasks.

## Table of Contents
- [Admin Access](#admin-access)
- [Admin Menu](#admin-menu)
- [User Management](#user-management)
- [Transaction Type Management](#transaction-type-management-optional)
- [System Maintenance](#system-maintenance)
- [Security Administration](#security-administration)
- [Monitoring and Troubleshooting](#monitoring-and-troubleshooting)
- [Best Practices](#best-practices)

## Admin Access

### Admin Credentials

Admin users have elevated privileges to manage users and system configuration.

**Default Admin Account:**
- User ID: `ADMIN001`
- Password: `PASSWORD`

**Important**: Change default passwords immediately after installation.

### Admin Sign-On

1. Enter transaction: `CC00`
2. Enter Admin User ID
3. Enter Admin Password
4. Press **Enter**
5. You'll see the Main Menu with admin options available

### Admin Privileges

Admin users can:
- Create, update, and delete user accounts
- Manage transaction types (with optional DB2 module)
- View all accounts and transactions
- Access system configuration
- Generate administrative reports
- Perform system maintenance tasks

## Admin Menu

### Accessing Admin Menu (CA00)

From the Main Menu:
1. Select Option 12 (Admin Menu)
2. Or enter transaction: `CA00`

![Admin Menu](../diagrams/Admin-Menu.png)

### Admin Menu Options

**User Management:**
1. **List Users** (CU00) - View all system users
2. **Add User** (CU01) - Create new user account
3. **Update User** (CU02) - Modify user information
4. **Delete User** (CU03) - Remove user account

**Transaction Type Management** (Optional DB2 Module):
5. **Transaction Type List** (CTLI) - List/update/delete transaction types
6. **Transaction Type Add/Edit** (CTTU) - Add or edit transaction types

**Function Keys:**
- **PF3**: Return to Main Menu
- **PF4**: Return to previous screen

## User Management

### List Users (CU00)

View all users in the system with their details.

**How to Access:**
- Admin Menu → Option 1
- Or enter transaction: `CU00`

**Displayed Information:**
- User ID
- User Name
- User Type (User/Admin)
- Status (Active/Inactive)
- Last Login Date

**Actions:**
- **S** (Select): View user details
- **U** (Update): Update user information
- **D** (Delete): Delete user account

**Function Keys:**
- **PF7**: Page backward
- **PF8**: Page forward
- **PF3**: Return to Admin Menu

**Example:**
```
User ID    Name              Type    Status    Last Login
ADMIN001   System Admin      Admin   Active    2025-11-18
USER0001   John Doe          User    Active    2025-11-17
USER0002   Jane Smith        User    Active    2025-11-16
USER0003   Bob Johnson       User    Inactive  2025-11-10
```

### Add User (CU01)

Create a new user account.

**How to Access:**
- Admin Menu → Option 2
- Or enter transaction: `CU01`

**Required Fields:**
- **User ID**: Unique 8-character identifier (uppercase)
- **User Name**: Full name (up to 50 characters)
- **Password**: Initial password (8 characters minimum)
- **User Type**: USER or ADMIN
- **Status**: ACTIVE or INACTIVE

**Optional Fields:**
- Email Address
- Phone Number
- Department
- Notes

**Add Process:**
1. Enter all required fields
2. Press **Enter** to validate
3. Review information
4. Press **Enter** to create user
5. Confirmation message with User ID will display

**Function Keys:**
- **Enter**: Validate / Create user
- **PF3**: Cancel and return to Admin Menu
- **PF5**: Clear form

**Business Rules:**
- User ID must be unique
- User ID cannot be changed after creation
- Password must meet complexity requirements
- Admin users should be limited to authorized personnel
- New users are created with ACTIVE status by default

**Password Requirements:**
- Minimum 8 characters
- Must contain at least one letter
- Must contain at least one number
- Case-sensitive

**Example:**
```
User ID: USER0010
User Name: Alice Williams
Password: ********
User Type: USER
Status: ACTIVE
Email: alice.williams@example.com
```

### Update User (CU02)

Modify existing user information.

**How to Access:**
- Admin Menu → Option 3
- Or enter transaction: `CU02`
- Or select 'U' from User List

**Input Fields:**
- **User ID**: Enter user ID to update

**Updatable Fields:**
- User Name
- Password (if resetting)
- User Type
- Status
- Email Address
- Phone Number
- Department
- Notes

**Update Process:**
1. Enter User ID
2. Press **Enter** to retrieve user
3. Modify desired fields
4. Press **Enter** to save changes
5. Confirmation message will display

**Function Keys:**
- **Enter**: Retrieve user / Save changes
- **PF3**: Return to Admin Menu without saving
- **PF5**: Reset password

**Common Update Scenarios:**

**Reset Password:**
1. Retrieve user
2. Press **PF5** (Reset Password)
3. Enter new password
4. Confirm new password
5. Press **Enter** to save

**Unlock Account:**
1. Retrieve user
2. Change Status to ACTIVE
3. Press **Enter** to save

**Change User Type:**
1. Retrieve user
2. Change User Type (USER/ADMIN)
3. Press **Enter** to save
4. User must sign off and sign on again for changes to take effect

**Deactivate User:**
1. Retrieve user
2. Change Status to INACTIVE
3. Press **Enter** to save
4. User will not be able to sign on

### Delete User (CU03)

Remove a user account from the system.

**How to Access:**
- Admin Menu → Option 4
- Or enter transaction: `CU03`
- Or select 'D' from User List

**Input Fields:**
- **User ID**: Enter user ID to delete

**Delete Process:**
1. Enter User ID
2. Press **Enter** to retrieve user
3. Review user information
4. Press **Enter** to confirm deletion
5. Confirmation prompt will display
6. Enter 'Y' to confirm or 'N' to cancel
7. Deletion confirmation message will display

**Function Keys:**
- **Enter**: Retrieve user / Confirm deletion
- **PF3**: Cancel and return to Admin Menu

**Business Rules:**
- Cannot delete currently signed-on users
- Cannot delete the last admin user
- Deletion is permanent and cannot be undone
- Consider deactivating instead of deleting for audit purposes

**Warning**: Deleting a user does not delete their transaction history or associated data.

## Transaction Type Management (Optional)

These functions are available only if the DB2 Transaction Type Management optional module is installed.

### Transaction Type List (CTLI)

List, update, and delete transaction types.

**How to Access:**
- Admin Menu → Option 5
- Or enter transaction: `CTLI`

**Displayed Information:**
- Transaction Type Code (2 characters)
- Transaction Description
- Category Code
- Status

**Actions:**
- **U** (Update): Update transaction type description
- **D** (Delete): Delete transaction type

**Function Keys:**
- **PF7**: Page backward
- **PF8**: Page forward
- **PF3**: Return to Admin Menu

**Update Process:**
1. Enter 'U' next to transaction type
2. Modify description
3. Press **Enter** to save

**Delete Process:**
1. Enter 'D' next to transaction type
2. Press **Enter**
3. Confirm deletion
4. Transaction type will be deleted if not in use

**Business Rules:**
- Cannot delete transaction types with existing transactions
- Transaction type codes cannot be changed
- Changes are immediately reflected in DB2

See [Optional Module - Transaction Type Management](Optional-Module-Transaction-Type.md) for details.

### Transaction Type Add/Edit (CTTU)

Add new transaction types or edit existing ones.

**How to Access:**
- Admin Menu → Option 6
- Or enter transaction: `CTTU`

**Fields:**
- **Transaction Type Code**: 2-character code (for new types)
- **Transaction Description**: Description (up to 50 characters)
- **Category Code**: Associated category
- **Status**: ACTIVE or INACTIVE

**Add Process:**
1. Enter new Transaction Type Code
2. Enter Description
3. Select Category Code
4. Press **Enter** to save

**Edit Process:**
1. Enter existing Transaction Type Code
2. Press **Enter** to retrieve
3. Modify Description or Category
4. Press **Enter** to save

**Function Keys:**
- **Enter**: Retrieve / Save
- **PF3**: Cancel and return to Admin Menu
- **PF5**: Clear form

## System Maintenance

### File Management

**Close Files for Batch Processing:**
```
Submit: CLOSEFIL job
```

This closes all VSAM files to allow batch jobs to run.

**Open Files After Batch:**
```
Submit: OPENFIL job
```

This reopens VSAM files for CICS access.

**Important**: Always close files before running batch jobs and reopen them afterward.

### Batch Job Monitoring

Monitor batch jobs through:
1. JES2/JES3 job queue
2. SDSF or equivalent job monitoring tool
3. Job output and return codes

**Critical Batch Jobs:**
- **POSTTRAN**: Transaction posting (daily)
- **INTCALC**: Interest calculation (monthly)
- **CREASTMT**: Statement generation (monthly)
- **CBPAUP0J**: Authorization purge (daily, if using optional module)

### CICS Resource Management

**Check File Status:**
```
CEMT INQUIRE FILE(ACCTDAT)
```

**Check Transaction Status:**
```
CEMT INQUIRE TRANSACTION(CC00)
```

**Check Program Status:**
```
CEMT INQUIRE PROGRAM(COSGN00C)
```

**Refresh Program:**
```
CEMT SET PROGRAM(COSGN00C) NEWCOPY
```

**Enable/Disable Transaction:**
```
CEMT SET TRANSACTION(CC00) ENABLED
CEMT SET TRANSACTION(CC00) DISABLED
```

### Database Maintenance

**VSAM File Backup:**
```
Submit: TRANBKP job (Transaction backup)
```

**VSAM File Reorganization:**
Periodically reorganize VSAM files for optimal performance:
1. Close files (CLOSEFIL)
2. Run IDCAMS REPRO to backup
3. Delete and redefine VSAM cluster
4. Restore data from backup
5. Rebuild alternate indexes
6. Open files (OPENFIL)

**DB2 Maintenance** (if using optional modules):
- Run RUNSTATS on tables
- Reorganize tables as needed
- Backup DB2 tables regularly

## Security Administration

### Password Management

**Password Policy:**
- Minimum 8 characters
- Must be changed every 90 days (configurable)
- Cannot reuse last 5 passwords
- Locked after 3 failed attempts

**Reset User Password:**
1. Access CU02 (Update User)
2. Enter User ID
3. Press PF5 (Reset Password)
4. Enter new password
5. Confirm and save

**Unlock User Account:**
1. Access CU02 (Update User)
2. Enter User ID
3. Change Status to ACTIVE
4. Save changes

### Access Control

**User Type Permissions:**

**Regular Users:**
- View own accounts and cards
- View own transactions
- Make payments
- Generate reports for own accounts

**Admin Users:**
- All regular user permissions
- Create/update/delete users
- View all accounts and transactions
- Manage transaction types
- System configuration

### Audit Trail

Transaction history provides audit trail:
- All transactions are logged
- User ID captured for each transaction
- Timestamps recorded
- Changes to user accounts logged

**View Audit Information:**
1. Access transaction history (CT00)
2. Filter by user or date range
3. Review transaction details

## Monitoring and Troubleshooting

### System Health Checks

**Daily Checks:**
- Verify all CICS files are open
- Check batch job completion
- Review error logs
- Monitor user activity

**Weekly Checks:**
- Review VSAM file space utilization
- Check for locked user accounts
- Review transaction volumes
- Verify backup completion

**Monthly Checks:**
- Review user access rights
- Audit admin activities
- Check for inactive users
- Review system performance

### Common Issues

**Issue: Users Cannot Sign On**
- Check USRSEC file is open
- Verify CICS region is active
- Check user account status
- Verify password is correct

**Issue: Transaction Fails**
- Check VSAM file status
- Verify file is not closed for batch
- Check for file space issues
- Review CICS logs for errors

**Issue: Batch Job Fails**
- Verify files are closed (CLOSEFIL)
- Check for file locks
- Review JCL for errors
- Check dataset availability

**Issue: Performance Degradation**
- Check VSAM file fragmentation
- Review CICS region storage
- Monitor transaction volumes
- Check for long-running transactions

### Error Log Review

**CICS Error Logs:**
```
CEMT INQUIRE TASK
CEMT INQUIRE TRANSACTION(CC00)
```

**Review CICS Messages:**
Check CICS message log for:
- ABEND codes
- File errors
- Program errors
- Security violations

**JCL Job Output:**
Review SYSOUT for:
- Return codes
- Error messages
- Warning messages
- Completion statistics

## Best Practices

### User Management

1. **Use Strong Passwords**: Enforce password complexity requirements
2. **Limit Admin Access**: Grant admin privileges only when necessary
3. **Regular Audits**: Review user accounts quarterly
4. **Deactivate vs Delete**: Deactivate users instead of deleting for audit trail
5. **Document Changes**: Maintain log of user account changes

### System Maintenance

1. **Regular Backups**: Backup VSAM files daily
2. **Batch Window**: Schedule batch jobs during off-peak hours
3. **File Management**: Always close files before batch, reopen after
4. **Monitor Space**: Check VSAM file space utilization regularly
5. **Test Changes**: Test in development before production

### Security

1. **Change Default Passwords**: Immediately after installation
2. **Limit Admin Users**: Minimize number of admin accounts
3. **Review Access**: Audit user access regularly
4. **Monitor Activity**: Review transaction logs for suspicious activity
5. **Secure Credentials**: Store passwords securely

### Performance

1. **Reorganize Files**: Reorganize VSAM files periodically
2. **Monitor Transactions**: Track transaction response times
3. **Optimize Batch**: Schedule batch jobs efficiently
4. **Review Logs**: Check for performance warnings
5. **Capacity Planning**: Monitor growth and plan for capacity

### Documentation

1. **Maintain Runbooks**: Document operational procedures
2. **Record Changes**: Log all configuration changes
3. **Update Contacts**: Keep emergency contact list current
4. **Document Issues**: Record problems and resolutions
5. **Training**: Ensure backup admins are trained

---

**Navigation**: [Home](Home.md) | [User Guide](User-Guide.md) | [CICS Configuration](CICS-Configuration.md) | [Troubleshooting](Troubleshooting.md)
