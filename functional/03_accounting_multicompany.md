# Odoo Accounting & Multi-Company Functional Questions

Functional interview questions covering invoicing, accounts affected by sales/purchases, payment reconciliation, fiscal positions, multi-currency, and multi-company setups.

---

### 1. Which Accounts are affected when creating & posting an Invoice from a Sales Order?

Assuming standard automated inventory valuation on delivery and invoicing:

- **On Customer Invoice Posting:**
  - **Debit:** **Accounts Receivable** (from the Customer/Partner) — increases amount owed by customer.
  - **Credit:** **Income / Revenue Account** (from Product or Product Category) — recognizes sales revenue.
  - **Credit:** **Tax Payable Account** (from Taxes on invoice lines) — records sales tax/VAT liability.
- **On Goods Delivery (Automated Valuation):**
  - **Debit:** **Stock Interim (Delivered)** or **COGS (Cost of Goods Sold)** — recognizes cost of sold inventory.
  - **Credit:** **Stock Valuation Account** (Asset) — reduces inventory value on hand.

*(Note: In Anglo-Saxon accounting, COGS is recognized when the customer invoice is posted rather than at delivery, clearing the Stock Interim Delivered account.)*

---

### 2. Explain Payment Reconciliation and Outstanding Accounts

When a customer pays an invoice, Odoo tracks the lifecycle cleanly across two steps:
1. **Register Payment on Invoice (`In Payment`):** Moves the balance from **Accounts Receivable** into an **Outstanding Receipts** interim account (configured on the Bank/Cash journal).
2. **Bank Statement Reconciliation (`Paid`):** When the bank feed/statement line arrives and is matched against the payment, Odoo clears **Outstanding Receipts** into the **Main Bank Account**.

Partial payments leave the invoice open with a remaining residual balance; minor rounding or bank fee differences can be settled via a **Write-Off** reconciliation model.

---

### 3. Differentiate Credit Notes and Vendor Refunds

- **Credit Note (Customer):** Reverses part or all of a posted Customer Invoice (e.g., returned goods, pricing error) without deleting posted legal entries—preserving the audit trail.
- **Vendor Credit Note / Refund:** Reverses part or all of a posted Vendor Bill when returning items to a supplier or receiving a supplier rebate.

---

### 4. What are Fiscal Positions, and why are they used?

A **Fiscal Position** automatically maps default **Taxes** and **Income/Expense Accounts** to different ones based on the partner's country, state, or tax status (e.g., *Domestic -> Export 0% Tax*, *B2B Intra-EU Reverse Charge*, or *Tax-Exempt Government Entity*).

This lets you keep one standard tax setup on your products while Odoo automatically swaps taxes and accounts when creating a Sales Order, Purchase Order, or Invoice for an international or exempt partner.

---

### 5. How does Odoo handle Multi-Currency and Exchange Differences?

- Enable foreign currencies in Accounting Settings and configure automatic currency rate updates (ECB, Open Exchange Rates, etc.).
- When an invoice is posted in a foreign currency (e.g., USD in a EUR company), Odoo stores both the foreign amount and the company currency equivalent at the **invoice date's rate**.
- When paid later at a different exchange rate, reconciling the payment automatically posts a **Realized Foreign Exchange Gain or Loss** journal entry.
- At period close, **Unrealized Currency Gains/Losses** reports let you revalue open receivables, payables, and foreign bank balances.

---

### 6. Explain Multi-Company Setup and How Data Is Managed

Odoo allows multiple legal entities (companies) to operate within a single database:
- **Company Assignment (`company_id`):** Transactions (Sales Orders, Invoices, Stock Moves, Journals) belong to a specific company. Master data (Partners, Products) can either be **shared across all companies** (`company_id` empty) or **restricted to one company**.
- **Company Switcher:** Users with access to multiple companies can switch their active company or view records across multiple selected companies simultaneously.
- **Company-Dependent Fields:** Properties like a product's *Income Account*, *Expense Account*, or *Cost* can differ per company even when the product itself is shared.
- **Inter-Company Transactions:** With Inter-Company Rules enabled, confirming a Sales Order in Company A to Company B can automatically generate the matching Purchase Order (or Invoice/Bill) in Company B.
