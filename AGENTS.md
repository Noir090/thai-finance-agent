# Personal Finance Agent

Use this folder as a local personal-finance workspace. `database.csv` is the source of truth. `dashboard.html` reads that file and presents a read-only view.

## First use and privacy

- If `database.csv` or `tax_preparation.csv` is missing, create it by copying the matching `*.example.csv` template. Never overwrite an existing local file.
- These CSVs, receipts, tax documents, account exports, and usage logs are private local data. Keep them out of commits and uploads. The repository `.gitignore` excludes the standard filenames and attachment folders; inspect the staged file list before every push.
- Do not put real transactions or real tax records in examples, screenshots, bug reports, or demo data. Use synthetic examples only.

## Transaction workflow

- When the user attaches a receipt or types a transaction, identify the date, merchant or recipient, amount paid, transaction type, and a suitable category. Append the transaction to `database.csv` unless the user asks for a correction or deletion.
- Treat a purchase receipt as an expense. Treat money received as income. Record investment purchases and sales as separate `investment` rows with positive amounts and `action` set to `Buy` or `Sell`.
- Use `transfer` only when the receipt or user clearly identifies movement between the user's own accounts. A payment to a merchant or another person remains an expense when the user says it is spending.
- Record the amount the user actually paid. For a discount, government subsidy, or co-payment, record the net amount paid and preserve the original amount and discount in `description` when shown.
- Store dates as Gregorian ISO `YYYY-MM-DD` in the `Asia/Bangkok` timezone. Convert Thai Buddhist Era years to Gregorian by subtracting 543. Store `year_month` as `YYYY-MM`.
- Store `amount_thb` as a number with two decimal places. Do not split totals into line items unless the user asks or the receipt makes a useful split clear.
- Use consistent categories. Suggested expense categories include `Food/Beverage`, `Groceries`, `Transportation`, `Housing`, `Utilities`, `Shopping`, `Health`, `Entertainment`, `Subscriptions`, `Travel`, `Transfer money`, and `Other`. Suggested income categories include `Salary`, `Freelance`, `Interest`, `Refund`, and `Other`. For investments, use `Fund`, `Stocks`, `Bonds`, `Crypto`, or `Savings` when appropriate. Keep existing category spellings once they appear in the database.
- Before appending, check likely duplicates using date, amount, merchant or recipient, receipt number, and source. Do not add a likely duplicate; explain the match.
- If a slip needs review, show the original receipt image inline with the extracted details, state what is uncertain, and ask the user a specific follow-up (for example, what the purchase was or which category it belongs in). Use the attached image or local source path when available. If only the category is uncertain and the transaction is otherwise clear, use the closest category provisionally and set `data_quality_flag` to `needs_review: <brief reason>`, then ask the user to confirm or correct it. Do not record a transaction until the user clarifies a critical field such as date, amount, or transaction type.
- Assign the next unused ID in the `TXN000001` sequence. Keep the exact CSV schema below, preserve existing rows, and keep rows sorted by date. Use the attachment filename in `source_reference` for receipt images and `manual_entry` as the source for typed entries.
- For a slip processed in Codex, use the current thread's local session log when available: match the user turn containing the image, read its `turn_context.model`, and sum `usage.input_tokens` and `usage.output_tokens` from `token_usage_record` entries with the same `turn_id`. Read only this metadata; do not copy unrelated prompts or responses. These are totals for the whole Codex turn (including conversation context and cached input), not an image-only token count. Fill the columns only when the turn contains exactly one slip and one model; if the log is missing, the turn has multiple slips, or multiple models cannot be separated, leave the fields blank. For typed entries, leave the fields blank. Never estimate or invent usage. Existing historical rows remain blank.
- Do not store full account or card numbers, credentials, or QR contents. Store only details needed to understand and reconcile the transaction.
- Keep receipts and financial data local to this folder. Do not upload, share, or send them to external services.
- When a receipt or income record may matter for Thai tax preparation, add a row to `tax_preparation.csv` and link its `transaction_id` to `database.csv` when possible. Use the Buddhist Era year in `tax_year`.
- Mark candidate deductions `needs_review` until the supporting documents and rules for that tax year are checked. Do not assume an expense or investment is deductible. Check current Thai rules before calculating or advising on tax.
- For tax screening, follow `THAI_TAX_GUIDE.md`, then verify current-year rules against official sources. Distinguish assessable income, income-related expenses, personal allowances/deductions, and tax credits. State when ordinary personal spending is not deductible by default; do not present an unchecked candidate as an entitlement.
- Keep the salary deposit in `database.csv` as the cash actually received. For tax preparation, use gross assessable salary from the employer's certificate (50 bis) when available and reconcile it with the deposit; do not overwrite the cash transaction with gross salary. Do not subtract a salary-withheld contribution from gross salary and also deduct that same contribution again. Record a qualifying contribution separately only once, at the amount supported for that tax year.
- Keep historical worksheet snapshots separate from current-year tax conclusions. Do not treat a copied prior-year estimate as a fresh calculation.
- Keep source worksheet rollups in `income_total` or `deduction_total` rows; do not add them to matching detail rows when summarizing. Preserve source references for copied values.

## Database schema

Keep these columns in this exact order:

`transaction_id,date,transaction_type,description,amount_thb,category,action,year_month,source,source_reference,data_quality_flag,model,token_input,token_output`

`tax_preparation.csv` columns are:

`tax_year,transaction_id,record_type,tax_category,amount_thb,source_reference,document_status,review_status,notes`

Use `record_type` values such as `income`, `income_total`, `deduction`, `deduction_total`, `tax_paid`, `taxable_income`, `tax_estimate`, and `tax_saving`. Keep candidate records separate from source snapshots and do not add source totals to detail rows.

Read `THAI_TAX_GUIDE.md` for the reusable screening and evidence checklist. Treat its categories as prompts for review, not a replacement for checking the selected tax year's rules.

## Dashboard

- The dashboard is read-only. Update `database.csv` and `tax_preparation.csv` to add or correct records, then reload the page.
- It provides year/month transaction filters, monthly trends, expense categories, recent transactions, and a Tax Prep section with a Buddhist Era tax-year filter and document/review status.
- On macOS, open `open-dashboard.command`. It serves this folder on `127.0.0.1:8765` and opens the dashboard. Keep its Terminal window open while using the dashboard and press Control-C there to stop the server.
- If the dashboard is opened directly with `file://`, choose both `database.csv` and `tax_preparation.csv` in the file picker or use `open-dashboard.command` for automatic loading.

The repository contains `database.example.csv` and `tax_preparation.example.csv` templates. Copy them to the local names above before opening the dashboard; the local files are ignored by Git.
