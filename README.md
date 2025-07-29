📁 Repository Structure Guide
This repository is dedicated to interview questions for Odoo, cleanly split between technical and functional topics. This guide explains the purpose of each folder, what topics are covered, and how contributors can update or extend the content.
✅ Technical Folder Structure
All developer-oriented and code-related interview questions go in the technical/ folder.
technical/
├── 01_core.md
├── 02_models.md
├── 03_fields_api.md
├── 04_views_qweb.md
├── 05_security_access.md
├── 06_rpc_api.md
├── 07_workflows_automation.md
├── 08_performance_scalability.md
├── 09_debugging_testing.md
└── 10_devtools_deployment.md

🧠 Topics Covered



File
Topics Included



01_core.md
Odoo architecture, server startup flow, module structure, init.py usage


02_models.md
ORM basics, Model vs TransientModel, inheritance, recordsets


03_fields_api.md
Field types, computed fields, constraints, @api.depends, onchange


04_views_qweb.md
XML views (form/tree/search), QWeb templates, rendering engine


05_security_access.md
User groups, access rights, record rules, security XML files


06_rpc_api.md
XML-RPC, JSON-RPC, REST API, controller routes, API authentication


07_workflows_automation.md
Server actions, automated activities, mail templates, cron jobs


08_performance_scalability.md
SQL optimization, caching, large datasets, lazy loading


09_debugging_testing.md
Logging, debugging tools, Odoo tests (TransactionCase, @tagged)


10_devtools_deployment.md
Odoo shell, scaffold, Docker, Runbot, install scripts, staging setup


✏️ How to Contribute to Technical
Use the YAML front-matter before each question:
<!-- ID: TECH-MODEL-001
difficulty: Medium
version: 13.0–17.0
tags: orm, inheritance
-->

✅ Functional Folder Structure
All business process or module-usage questions go in the functional/ folder.
functional/
├── 01_sales.md
├── 02_purchase.md
├── 03_inventory.md
├── 04_accounting.md
├── 05_project.md
├── 06_manufacturing.md
├── 07_hr_payroll.md
├── 08_crm.md
├── 09_website_ecommerce.md
└── 10_general.md

🧠 Topics Covered



File
Topics Included



01_sales.md
Sales order states, quotations, pricelists, units of measure, delivery policies


02_purchase.md
RFQs, PO approval flows, vendor bills, dropshipping


03_inventory.md
Routes, pickings, stock moves, inventory adjustments


04_accounting.md
Invoicing, reconciliation, journal entries, fiscal positions, tax config


05_project.md
Project stages, tasks, milestones, timesheet billing


06_manufacturing.md
BoMs, MOs, subcontracting, MPS, work centers


07_hr_payroll.md
Employee contracts, leaves, payslip batches, payroll structure


08_crm.md
Lead vs opportunity, stages, lost reasons, tags


09_website_ecommerce.md
Webshop setup, payment acquirers, checkout flow


10_general.md
Company settings, multi-company, user management, chatter usage


✏️ How to Contribute to Functional
Use the same YAML block format for metadata:
<!-- ID: FUNC-SALE-002
difficulty: Easy
version: 13.0–17.0
tags: sales, pricelist
-->
