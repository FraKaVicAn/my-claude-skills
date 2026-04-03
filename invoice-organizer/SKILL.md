---
name: invoice-organizer
description: Transform disorganized invoices and receipts into a tax-ready filing structure — extracts data, standardizes names, sorts by vendor/category, exports CSV
origin: ComposioHQ/awesome-claude-skills
tools: Read, Write, Bash, Glob
---

# Invoice Organizer Skill

Automate invoice organization: extract vendor, date, amount, and description from PDFs/images, standardize filenames, sort into logical folders, and export a CSV for accounting software.

## When to Activate

- Tax season — getting receipts and invoices organized
- Expense reconciliation for a project or period
- Setting up a new filing structure for ongoing vendor management
- Cleaning up a Downloads folder full of unsorted financial docs

## Supported File Types

- PDF (invoices, statements)
- Images (JPEG, PNG — scanned receipts, screenshots)
- Email attachments (forwarded to a folder)
- Bank statement exports

## Seven-Step Workflow

1. **Scan** — find all invoice-like files in the target directory
2. **Extract** — pull vendor name, invoice number, date, amount, service description
3. **Propose structure** — suggest organization scheme, confirm with user
4. **Standardize names** — rename to: `YYYY-MM-DD Vendor - Invoice#NNN - Description.ext`
5. **Sort** — move into the approved folder hierarchy
6. **Export CSV** — one row per invoice for accounting software import
7. **Report** — summary of files processed, total amounts by vendor/category

## Organization Schemes

**By vendor:**
```
invoices/
  aws/
  figma/
  stripe/
```

**By year + category:**
```
invoices/
  2025/
    software/
    services/
    hardware/
```

**By quarter:**
```
invoices/
  2025-Q1/
  2025-Q2/
```

## CSV Export Format

```csv
date,vendor,invoice_number,amount,currency,category,file_path,tax_deductible
2025-03-15,AWS,INV-2025-034,142.50,USD,cloud-infrastructure,invoices/aws/2025-03-15 AWS - INV-2025-034 - Cloud Services.pdf,yes
```

## Safety Rules

- Always preserve originals — create organized copies, not moves, unless confirmed
- Confirm the filing structure before executing any changes
- Log every rename and move to `reorganization_log.txt`

## Links

- [github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
