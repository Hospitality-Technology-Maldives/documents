# FLOAT CREDIT

**Created by:** Mohamed Shaif Hassan  
**Last Updated:** 9th August 2024  
**Company:** Hospitality Technology Maldives

---

### Purpose

This document serves as a user guide for configuring the Float Credit formula to set up package codes in Opera, specifically for handling resort credit.

### Intended Audience

This document is intended for users responsible for configuring package codes in Opera.

---

### Getting Started

To properly set up the resort credit package, the following configurations must be addressed:

- **Package Forecast Group:** A forecast group must be created, which will serve as the main identifier to distinguish a package code as a resort credit or a regular package. This forecast group must be linked to all resort credit package codes.
- **Application Setting:** A new application setting, located under _General > Settings > RESORT CREDIT PKG GROUP_ (Package forecast group to identify resort credit packages), has been created. By default, this setting is set to `ADV_PKG`. You can either create a forecast group called `ADV_PKG` or use a different name by adjusting the application setting value accordingly.
- **Package Code:** All packages configured as resort floating credit must meet the following requirements:
  - Must be configured as a simple package.
  - The forecast group should be selected according to the `RESORT CREDIT PKG GROUP` application setting value.
  - The posting rhythm should be set to "Post every night."
  - The calculation rule should be set to "Fixed amount."
  - The float formula must be set to "Formula."

Once these requirements are met, the package can be attached to a reservation separately as a sell-separate package, or it can be linked to a rate code or package group.

---

### Package Formula

```sql
HTOPERA.FLOAT_PKG.GEN_CREDIT(
    `Allowance pool code`, -- Allowance pool
    RESV_NAME_ID,
    `List of alternative transaction codes`, -- Comma-separated list of alternative transaction codes
    `Calculation rule`, -- F = Fixed, P = Per Person, A = Per Adult, C = Per Children, R = Per Room
    `Mode`,
    `Price1`,
    `Price2`, -- Optional amount
    `Price3`  -- Optional amount
    );

-- Example

HTOPERA.FLOAT_PKG.GEN_CREDIT('SPA', RESV_NAME_ID, '32002,32001,30223', 'A', 'NA', 55);
```

- **Allowance Pool Code:** This is the code for the allowance pool, which can be used across multiple packages to combine the total allowance created by the package.
- **Alternative Transaction Codes:** A comma-separated list of transaction codes that are eligible for applying the resort credit.
- **Calculation Rule:** Set the appropriate calculation rule based on your requirements.
- **Mode:** The calculation mode should be set to 'NA' and cannot be changed.
- **Price:** The resort credit amount. The total credit will be calculated based on the calculation rule.
- **Price2:** If using a children’s price bucket, this is the credit amount for the second bucket.
- **Price3:** If using a children’s price bucket, this is the credit amount for the first bucket.

---

### How It Works

**Check-in Process**  
When a reservation with a resort credit package code is checked in, a package floating allowance is created and recorded. At this point, no financial posting occurs. If desired, the check-in can be canceled, and the floating allowance will be removed for the reservation.

**Posting**  
When a transaction, designated as an alternative, is posted to the room, the system will apply the resort credit to this transaction. A negative posting, not exceeding the total allowance, will automatically be posted to the same window within a few minutes. The adjustment transaction code will match the posted transaction code. If no adjustment transaction code exists, the same transaction code will be used.

**Night Audit Posting**  
If a resort credit is applied during the cashiering window, the night audit will post an equivalent amount as the package posting. This amount will not exceed the total allowance created for that package. If the allowance is not fully utilized, the balance will be posted during the final night audit.

---

### Scenarios

> [!NOTE]
>
> - `FC` and `NA` are matching control types.
> - `C` and `D` are matching control types for posting and adjustment posting.
> - The Credit and Debit columns should balance to fully utilize the resort credit.

**Example 1:**  
A guest is entitled to a $55 resort credit, arriving on the 16th and departing on the 18th (2 nights).

| Date     | Trx Code | Debit | Credit | Type | Reference                         |
| -------- | -------- | ----- | ------ | ---- | --------------------------------- |
| 16/08/23 | 32000    | -     | 55     | FC   | Allowance Created at Check-In     |
| 16/08/23 | 32002    | 20    | -      | C    | POS Posting                       |
| 16/08/23 | 32002    | -     | 20     | D    | Adjusted Credit                   |
| 16/08/23 | 32000    | 20    | -      | NA   | Night Audit Allocation            |
| 17/08/23 | 32000    | 35    | -      | NA   | Night Audit Allocation (Last Day) |
| 18/08/23 | 32002    | 50    | -      | C    | POS Posting                       |
| 18/08/23 | 32002    | -     | 35     | D    | Adjusted Credit                   |
| 18/08/23 | 32002    | -15   | -      | C    | Overage                           |

**Explanation:**

- On the first day, during check-in, an allowance of $55 is created for the guest. This is a pseudo amount, and nothing is posted to the room at this point.
- The guest makes a $20 posting to the room.
- A -$20 adjustment is posted to the room automatically to apply the resort credit.
- During the night audit, a +$20 posting is made, reflecting the amount utilized that day for resort credit.
- The following day, no POS posting is made to the room.
- During the night audit, the remaining +$35 is posted to the room as this is the last night audit. If the resort credit is not fully consumed, the balance will be posted during the last night audit.
- On the departure day, the guest posts a $50 transaction to the room.
- A +$35 adjustment is posted to the room automatically to apply the maximum available resort credit.
- A balance of $15 remains on the room, shown as a pseudo amount -$15 as overage. This is not a physical posting.

> [!NOTE]
>
> - `FC` and `NA` records will balance each other out.
> - `C` and `D` records will balance each other out.

---
