# Google Sheet & Docs Template Structure

## Google Sheet columns

| Column        | Type   | Notes                                              |
|---------------|--------|-----------------------------------------------------|
| Invoice ID    | string | Unique invoice identifier                           |
| Invoice Date  | string | Date the invoice is issued                          |
| Company       | string | Client/company name                                 |
| Email         | string | Client email — used by the Gmail node               |
| Slack ID      | string | Client's Slack member ID — used by the Slack node    |
| Address       | string | Billing address                                     |
| Item          | string | Line item description                               |
| Qty           | number | Quantity                                             |
| Unit Price    | number | Price per unit                                       |
| Tax %         | number | Tax percentage applied to the subtotal               |
| Due Date      | string | Payment due date                                     |
| Status        | string | `Pending` → picked up by the workflow; set to `Invoice Sent` after processing |

`row_number` is auto-managed by n8n's Google Sheets node — no need to add it manually.

## Google Docs template placeholders

Create a Google Doc invoice template and insert these exact placeholders (curly braces included)
wherever you want the data to appear:

```
{{invoice_id}}
{{invoice_date}}
{{company}}
{{address}}
{{item_line}}
{{due_date}}
{{subtotal}}
{{tax}}
{{total}}
```

The workflow copies this template per invoice, then calls the Google Docs `batchUpdate` API to
replace each placeholder with the corresponding value before exporting to PDF.

`item_line` is a pre-formatted string built in the "Code - Calculate Totals" node, e.g.:
```
Consulting Services  |  Qty: 3  |  Unit Price: $150.00  |  Amount: $450.00
```
If you have multiple line items per invoice, you'll want to extend that Code node to loop over
items and join multiple `item_line` entries.
