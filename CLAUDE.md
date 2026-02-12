# CLAUDE.md - Project Instructions

## Project Overview

This repository contains **Solution Design Verification Reports** for **Technica International**, a Lebanese make-to-order, project-driven manufacturing company implementing **Dynamics 365 Finance & Operations (F&O)** and **D365 Sales (CRM)**.

The reports review Functional Requirements Documents (FRDs) and User Training Manuals against D365 best practices, identifying gaps, risks, and enhancement opportunities.

## Repository Structure

```
/
├── CLAUDE.md                  # This file — project instructions
├── README.md
├── FRDs/                      # FRD Solution Design Verification Reports (18 files)
│   ├── FRD01_Sales_Review.md
│   ├── FRD02_SalesSupport_Review.md
│   ├── ...
│   └── FRD13_Integration_Review.md
├── TrainingManuals/            # Training Manual Verification Reports (7 files)
│   ├── TM_AccountsPayable_Review.md
│   ├── TM_AccountsReceivable_Review.md
│   ├── ...
│   └── TM_SalesMarketing_Review.md
├── *.pdf                       # Source FRD documents
└── *.docx                      # Source Training Manual documents
```

## Review File Format Conventions

All review files (FRD and Training Manual) MUST follow this format:

### 1. Header

```markdown
# [Document Title] - Solution Design Verification Report

**Document:** [Source document name and version]
**Prepared by:** [Author] | Contributor: [Contributor]
**Review Date:** [YYYY-MM-DD]
**Platform:** [D365 modules involved]
```

### 2. Enhancement Summary Table (MANDATORY — immediately after header)

Every review file must begin with an Enhancement Summary table listing ALL areas requiring attention:

```markdown
## ENHANCEMENT SUMMARY

> The table below lists all areas requiring attention, their severity, and where to find them in this document.

| # | Severity | Process / Area | Enhancement | Section |
|---|----------|---------------|-------------|---------|
| 1 | **CRITICAL** | [area] | [description] | [link to section] |
| 2 | **HIGH** | [area] | [description] | [link to section] |
| 3 | **MEDIUM** | [area] | [description] | [link to section] |
| 4 | **LOW** | [area] | [description] | [link to section] |

**Totals:** X CRITICAL | Y HIGH | Z MEDIUM | W LOW
```

- Severity levels: **CRITICAL**, **HIGH**, **MEDIUM**, **LOW**
- Order rows by severity (CRITICAL first, LOW last)
- Section column should contain markdown links to the relevant heading

### 3. Attention Markers in Document Body

Use these visual markers throughout the document to make issues easy to locate:

**Attention areas:**
```markdown
**>> ATTENTION AREA: [short description of the issue]**
> [Blockquote with detailed explanation of the concern]
```

**Enhancement recommendations:**
```markdown
**>> ENHANCEMENT:** [recommendation text]
**>> ENHANCEMENT (CRITICAL):** [critical recommendation text]
```

**Rating table flags (used in assessment tables):**
```markdown
| Aspect | **>> NEEDS REVIEW <<** | Comment |
| Aspect | **>> RISKY <<** | Comment |
| Aspect | **>> NEEDS ARCHITECTURE <<** | Comment |
| Aspect | **>> NEEDS IMPROVEMENT <<** | Comment |
```

### 4. Document Structure

After the Enhancement Summary, follow this structure:
1. **Executive Summary** — high-level assessment
2. **Process-by-Process Review** — each process/section with assessment tables and recommendations
3. **Overall Design Verdict** — summary rating table and final verdict

## File Naming Conventions

- FRD reviews: `FRDs/FRD[XX]_[ModuleName]_Review.md`
- Training Manual reviews: `TrainingManuals/TM_[ModuleName]_Review.md`

## D365 Modules Covered

| Module | FRD | Training Manual |
|--------|-----|-----------------|
| Sales (CRM) | FRD01 | TM_SalesMarketing |
| Sales Support | FRD02 | — |
| R&D | FRD03 | — |
| Project Accounting | FRD04 | — |
| Production | FRD05 | TM_ProductionControl |
| Logistics | FRD06 | TM_Logistics |
| Procurement | FRD07 | — |
| Accounts Payable | FRD07_AP | TM_AccountsPayable |
| Inventory Management | FRD009 | TM_InventoryManagement |
| Asset Management | FRD10 | — |
| Customer Service | FRD011 | — |
| Spare Parts | FRD_SpareParts | — |
| Expense Management | FRD12_Expense | — |
| GL / Cash & Bank | FRD12_GL_CashBank | TM_GL_CashBank |
| Accounts Receivable | FRD12_AR | TM_AccountsReceivable |
| Budgeting | FRD12_Budgeting | — |
| Fixed Assets | FRD12_FixedAssets | — |
| Integration | FRD13 | — |

## Key Technical Context

- **Client:** Technica International (Lebanon) — make-to-order manufacturing
- **Platform:** D365 F&O + D365 Sales (CRM)
- **Integration:** Dual-Write between CRM and F&O
- **Localization:** Poland localization used for Lebanese VAT (dual-rate)
- **Key patterns:** Power Automate for workflows, Business Events for integrations, DMF for data migration
- **Technica ND:** Fictitious legal entity used for intercompany transactions — flagged as risk area

## Review Guidelines

When creating or updating review files:
1. Always cross-reference the source document against D365 standard capabilities
2. Flag anything that duplicates OOB functionality as a potential over-customization
3. Compare Training Manual content against corresponding FRD to identify coverage gaps
4. Use the severity scale consistently:
   - **CRITICAL** — Blocks go-live, causes data loss, or represents a fundamental architecture gap
   - **HIGH** — Significant risk or missing functionality that will cause major issues
   - **MEDIUM** — Suboptimal design that should be improved but won't block progress
   - **LOW** — Minor improvements, cosmetic issues, or nice-to-haves
