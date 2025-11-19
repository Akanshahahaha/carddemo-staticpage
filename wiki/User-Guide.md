# User Guide

Complete guide to using CardDemo as a regular user. This guide covers all user-facing functions, screens, and workflows.

## Table of Contents
- [Getting Started](#getting-started)
- [Sign-On Process](#sign-on-process)
- [Main Menu](#main-menu)
- [Account Management](#account-management)
- [Card Management](#card-management)
- [Transaction Management](#transaction-management)
- [Bill Payment](#bill-payment)
- [Reports](#reports)
- [Pending Authorizations](#pending-authorizations-optional)
- [Navigation Tips](#navigation-tips)
- [Common Tasks](#common-tasks)

## Getting Started

### Accessing CardDemo

1. Clear your CICS terminal
2. Enter transaction code: `CC00`
3. The CardDemo sign-on screen will appear

### User Types

CardDemo supports two user types:
- **Regular Users**: Access to account, card, transaction, and payment functions
- **Admin Users**: Additional access to user management and system administration

This guide focuses on regular user functions. See [Admin Guide](Admin-Guide.md) for administrative functions.

## Sign-On Process

### Sign-On Screen (CC00)

![Signon Screen](../diagrams/Signon-Screen.png)

**Fields:**
- **User ID**: Your assigned user identifier (8 characters)
- **Password**: Your password (8 characters)

**Function Keys:**
- **Enter**: Submit credentials and sign on
- **PF3**: Exit application

### Sample User Credentials

For testing purposes, use these credentials:
- User ID: `USER0001`
- Password: `PASSWORD`

### Sign-On Process

1. Enter your User ID
2. Enter your Password
3. Press **Enter**
4. If credentials are valid, you'll see the Main Menu
5. If invalid, an error message will display

### Security Notes

- Passwords are case-sensitive
- After 3 failed attempts, the user ID may be locked
- Contact your administrator to reset a locked account

## Main Menu

### Main Menu Screen (CM00)

![Main Menu](../diagrams/Main-Menu.png)

The Main Menu provides access to all CardDemo functions:

**Menu Options:**

1. **Account View** - View account information
2. **Account Update** - Update account details
3. **Card List** - List all cards for an account
4. **Card View** - View specific card details
5. **Card Update** - Update card information
6. **Transaction List** - View transaction history
7. **Transaction View** - View specific transaction details
8. **Transaction Add** - Add a new transaction
9. **Bill Payment** - Make a payment
10. **Transaction Report** - Generate transaction reports
11. **Pending Authorizations** - View pending authorizations (optional module)

**Function Keys:**
- **PF3**: Sign off and exit
- **PF4**: Return to previous screen
- **Clear**: Clear screen

### Navigation

To select a menu option:
1. Enter the option number in the **Selection** field
2. Press **Enter**

## Account Management

### Account View (CAVW)

View detailed account information including balances and credit limits.

**How to Access:**
- Main Menu → Option 1
- Or enter transaction: `CAVW`

**Input Fields:**
- **Account Number**: 11-digit account number

**Displayed Information:**
- Account ID
- Account Status (Active, Closed, etc.)
- Current Balance
- Credit Limit
- Available Credit
- Cash Credit Limit
- Open Date
- Expiration Date
- Reissue Date
- Customer Information

**Function Keys:**
- **Enter**: Display account details
- **PF3**: Return to Main Menu
- **PF4**: Return to previous screen

**Example:**
```
Account Number: 0000000001
Status: Active
Current Balance: $1,234.56
Credit Limit: $5,000.00
Available Credit: $3,765.44
```

### Account Update (CAUP)

Update account information such as credit limits and status.

**How to Access:**
- Main Menu → Option 2
- Or enter transaction: `CAUP`

**Updatable Fields:**
- Credit Limit
- Cash Credit Limit
- Account Status

**Update Process:**
1. Enter Account Number
2. Press **Enter** to retrieve account
3. Modify desired fields
4. Press **Enter** to save changes
5. Confirmation message will display

**Function Keys:**
- **Enter**: Retrieve account / Save changes
- **PF3**: Return to Main Menu without saving
- **PF4**: Return to previous screen

**Business Rules:**
- Credit limit must be greater than current balance
- Only active accounts can be updated
- Status changes may require approval

## Card Management

### Card List (CCLI)

Display all credit cards associated with an account.

**How to Access:**
- Main Menu → Option 3
- Or enter transaction: `CCLI`

**Input Fields:**
- **Account Number**: 11-digit account number

**Displayed Information:**
- Card Number (16 digits)
- Card Status (Active, Blocked, Expired)
- Expiration Date
- Cardholder Name

**Actions:**
- **S** (Select): View card details
- **U** (Update): Update card information

**Function Keys:**
- **PF7**: Page backward
- **PF8**: Page forward
- **PF3**: Return to Main Menu

**Example:**
```
Account: 0000000001

Card Number          Status    Exp Date   Name
4000123456789010     Active    12/25      JOHN DOE
4000123456789011     Active    06/26      JANE DOE
```

### Card View (CCDL)

View detailed information for a specific credit card.

**How to Access:**
- Main Menu → Option 4
- Or enter transaction: `CCDL`
- Or select 'S' from Card List

**Input Fields:**
- **Card Number**: 16-digit card number

**Displayed Information:**
- Card Number
- Account Number
- Card Status
- Expiration Date
- Cardholder Name
- Embossed Name
- Card Type
- Issue Date
- Active Status

**Function Keys:**
- **PF3**: Return to Main Menu
- **PF4**: Return to Card List

### Card Update (CCUP)

Update credit card information.

**How to Access:**
- Main Menu → Option 5
- Or enter transaction: `CCUP`
- Or select 'U' from Card List

**Updatable Fields:**
- Card Status (Active, Blocked, Closed)
- Expiration Date
- Cardholder Name

**Update Process:**
1. Enter Card Number
2. Press **Enter** to retrieve card
3. Modify desired fields
4. Press **Enter** to save changes
5. Confirmation message will display

**Function Keys:**
- **Enter**: Retrieve card / Save changes
- **PF3**: Return to Main Menu without saving

**Business Rules:**
- Cannot reactivate an expired card
- Blocked cards require approval to unblock
- Name changes must match customer records

## Transaction Management

### Transaction List (CT00)

View transaction history for a credit card.

**How to Access:**
- Main Menu → Option 6
- Or enter transaction: `CT00`

**Input Fields:**
- **Card Number**: 16-digit card number
- **Start Date**: Optional filter (YYYY-MM-DD)
- **End Date**: Optional filter (YYYY-MM-DD)

**Displayed Information:**
- Transaction ID
- Transaction Date
- Transaction Type
- Description
- Amount
- Merchant Name
- Category

**Actions:**
- **S** (Select): View transaction details

**Function Keys:**
- **PF7**: Page backward
- **PF8**: Page forward
- **PF3**: Return to Main Menu

**Sorting:**
- Transactions are displayed in reverse chronological order (newest first)

**Example:**
```
Card: 4000123456789010

Trans ID    Date        Type  Description      Amount
T0001234    2025-11-15  PUR   GROCERY STORE    $45.67
T0001233    2025-11-14  PUR   GAS STATION      $52.30
T0001232    2025-11-13  PAY   PAYMENT          -$100.00
```

### Transaction View (CT01)

View detailed information for a specific transaction.

**How to Access:**
- Main Menu → Option 7
- Or enter transaction: `CT01`
- Or select 'S' from Transaction List

**Input Fields:**
- **Transaction ID**: Unique transaction identifier

**Displayed Information:**
- Transaction ID
- Card Number
- Transaction Date and Time
- Transaction Type
- Transaction Category
- Description
- Amount
- Merchant Information:
  - Merchant Name
  - Merchant City
  - Merchant State
  - Merchant ZIP
- Authorization Code
- Processing Information

**Function Keys:**
- **PF3**: Return to Main Menu
- **PF4**: Return to Transaction List

### Transaction Add (CT02)

Manually add a new transaction (for testing or adjustment purposes).

**How to Access:**
- Main Menu → Option 8
- Or enter transaction: `CT02`

**Required Fields:**
- **Card Number**: 16-digit card number
- **Transaction Type**: Type code (e.g., PUR for Purchase)
- **Transaction Category**: Category code
- **Amount**: Transaction amount
- **Description**: Transaction description
- **Merchant Name**: Merchant identifier

**Optional Fields:**
- Merchant City
- Merchant State
- Merchant ZIP

**Add Process:**
1. Enter all required fields
2. Press **Enter** to validate
3. Confirm transaction details
4. Press **Enter** to submit
5. Transaction ID will be generated and displayed

**Function Keys:**
- **Enter**: Validate / Submit transaction
- **PF3**: Cancel and return to Main Menu
- **PF5**: Clear form

**Business Rules:**
- Card must be active
- Amount must be positive for purchases
- Amount must be negative for payments/credits
- Transaction type must be valid
- Available credit must be sufficient

**Transaction Types:**
- **PUR**: Purchase
- **PAY**: Payment
- **ADJ**: Adjustment
- **FEE**: Fee
- **INT**: Interest

## Bill Payment

### Bill Payment Screen (CB00)

Make a payment toward your credit card balance.

**How to Access:**
- Main Menu → Option 9
- Or enter transaction: `CB00`

**Input Fields:**
- **Account Number**: 11-digit account number
- **Card Number**: 16-digit card number
- **Payment Amount**: Amount to pay
- **Payment Date**: Date of payment (defaults to today)
- **Payment Method**: Check, Transfer, etc.

**Displayed Information:**
- Current Balance
- Minimum Payment Due
- Payment Due Date
- Available Credit

**Payment Process:**
1. Enter Account Number
2. Enter Card Number
3. Press **Enter** to retrieve balance
4. Enter Payment Amount
5. Select Payment Method
6. Press **Enter** to process payment
7. Confirmation message and receipt number will display

**Function Keys:**
- **Enter**: Retrieve balance / Process payment
- **PF3**: Cancel and return to Main Menu
- **PF5**: Clear form

**Business Rules:**
- Payment amount must be positive
- Payment cannot exceed current balance
- Minimum payment must be met by due date
- Payment posts immediately to available credit

**Payment Methods:**
- **CHK**: Check
- **ACH**: Electronic Transfer
- **WIRE**: Wire Transfer
- **CASH**: Cash Payment

## Reports

### Transaction Report (CR00)

Generate and view transaction reports.

**How to Access:**
- Main Menu → Option 10
- Or enter transaction: `CR00`

**Report Options:**
1. **Account Summary**: Summary of all accounts
2. **Transaction Detail**: Detailed transaction listing
3. **Category Summary**: Transactions grouped by category
4. **Merchant Summary**: Transactions grouped by merchant

**Input Fields:**
- **Report Type**: Select from options above
- **Account Number**: Optional filter
- **Card Number**: Optional filter
- **Start Date**: Report period start
- **End Date**: Report period end

**Report Process:**
1. Select Report Type
2. Enter filter criteria
3. Press **Enter** to generate report
4. Report displays on screen
5. Use PF7/PF8 to scroll through pages

**Function Keys:**
- **PF7**: Page backward
- **PF8**: Page forward
- **PF3**: Return to Main Menu
- **PF5**: Print report (if configured)

**Report Output:**
- Screen display (default)
- Print queue (PF5)
- Batch report generation available

## Pending Authorizations (Optional)

This feature is available only if the Credit Card Authorizations optional module is installed.

### Authorization Summary (CPVS)

View pending authorization requests for your cards.

**How to Access:**
- Main Menu → Option 11
- Or enter transaction: `CPVS`

**Input Fields:**
- **Account Number**: 11-digit account number

**Displayed Information:**
- Authorization ID
- Card Number
- Authorization Date/Time
- Merchant Name
- Amount
- Status (Pending, Approved, Declined)

**Actions:**
- **S** (Select): View authorization details

**Function Keys:**
- **PF7**: Page backward
- **PF8**: Page forward
- **PF3**: Return to Main Menu

### Authorization Details (CPVD)

View detailed information for a specific authorization.

**How to Access:**
- Select 'S' from Authorization Summary
- Or enter transaction: `CPVD`

**Displayed Information:**
- Complete authorization details
- Merchant information
- Transaction amount
- Authorization response
- Fraud indicators

**Actions:**
- **PF5**: Mark as fraudulent (if suspicious)

**Function Keys:**
- **PF3**: Return to Main Menu
- **PF4**: Return to Authorization Summary
- **PF5**: Mark as fraud

See [Optional Module - Authorization](Optional-Module-Authorization.md) for more details.

## Navigation Tips

### Function Key Reference

| Key | Function |
|:----|:---------|
| **Enter** | Submit / Confirm |
| **PF3** | Exit / Return to Main Menu |
| **PF4** | Return to previous screen |
| **PF5** | Clear / Special function |
| **PF7** | Page backward |
| **PF8** | Page forward |
| **PF12** | Cancel |
| **Clear** | Clear screen |

### Screen Navigation

- Use **PF3** to return to Main Menu from any screen
- Use **PF4** to return to the previous screen
- Use **PF7/PF8** to scroll through multi-page lists
- Use **Clear** to clear the screen and start over

### Data Entry Tips

- Tab between fields
- Use uppercase for consistency
- Date format: YYYY-MM-DD
- Amount format: 9999.99 (no $ or commas)
- Card numbers: 16 digits, no spaces
- Account numbers: 11 digits, no spaces

## Common Tasks

### Check Account Balance

1. Enter `CAVW` or Main Menu → Option 1
2. Enter Account Number
3. Press **Enter**
4. View Current Balance and Available Credit

### View Recent Transactions

1. Enter `CT00` or Main Menu → Option 6
2. Enter Card Number
3. Press **Enter**
4. Browse transaction list with PF7/PF8

### Make a Payment

1. Enter `CB00` or Main Menu → Option 9
2. Enter Account Number and Card Number
3. Press **Enter** to see balance
4. Enter Payment Amount
5. Press **Enter** to process

### Report a Lost Card

1. Enter `CCUP` or Main Menu → Option 5
2. Enter Card Number
3. Press **Enter**
4. Change Status to "BLOCKED"
5. Press **Enter** to save
6. Contact administrator for replacement

### Update Personal Information

1. Contact your administrator
2. Provide updated information
3. Administrator will update via Admin functions

---

**Navigation**: [Home](Home.md) | [Admin Guide](Admin-Guide.md) | [Application Screens](Application-Screens.md) | [FAQ](FAQ.md)
