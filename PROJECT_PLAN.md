# TMF/ISF Tracker - Project Build Plan

## Project Context

A self-contained single-page HTML application for Clinical Research Associates (CRAs) to track essential TMF/ISF documents for clinical trial sites. Documents are uploaded via Excel export from Veeva Vault.

## Session Startup Routine

1. Open Command Prompt (Windows key > type cmd > Enter)
2. `cd "C:\Users\BTZMD\OneDrive - Bayer\Files\MGA"`
3. `ccr code`
4. At the start of each session paste this first:

> "Before we start any work today, please: 1) create a backup copy of the current tracker file by duplicating it and adding '_BACKUP' and today's date to the filename, and 2) increment the version number in the active file's filename and in the page title inside the HTML. Then confirm both are done before we proceed."

---

## Phase 1 - COMPLETE

- Global search bar per tab, filters visible rows as you type, clears on tab switch
- Colour coding fix: amber/yellow = unreconciled, green = new, red = genuinely urgent only
- Remove COV tab
- Remove closeout percentage from dashboard

## Phase 2 - COMPLETE

- Split dashboard alerts into four categories: Staff, Consent, IP/Pharmacy, Labs
- Each category shows green "All good" when clean, or list of alerts when not
- CRA personal reminders attached to site, with due date
- Reminders appear in dashboard alerts in their own section
- CRA can mark reminder complete - disappears from active view, saved to history
- History viewable on demand
- Reminders and history exportable and importable

## Phase 3 - IN PROGRESS

- [x] Add Document button inside every tab
- [x] Form with free text fields: Name (mandatory), Classification, Version, Date plus tab-specific fields *(Works but no tab-specific fields)*
- [x] Manually added documents show Pending and Manual badges *(Mostly works; "Pending" fails on fresh sites)*
- [x] Manually added documents survive export/import
- [x] Move to... option on every document row - unobtrusive, click to expand, lists other tabs *(Update render functions to use movedDocs and show move button)*
- [x] Moved documents stay moved after new Veeva uploads
- [ ] Manual merge - CRA can select two rows and merge them, reconciliation status preserved, Manual tag removed if one row was from Veeva
- [ ] Automatic match on Veeva upload - fuzzy name matching against manually added documents, popup at upload time showing two documents side by side, CRA confirms or dismisses, confirmed matches merge with reconciliation status preserved

## Phase 4 (TBC?) - Consent Enhancement (complex, optional feature)

Design notes: Entire feature must be optional - sites without subgroups ignore it completely.

- **Consent Type Enhancement**: When a CRA creates a new consent type or version, add an 'Other' category option. Also add an optional subgroup field so the CRA can specify which subgroup this ICF applies to if relevant.
- **Patient Subgroup**: When creating a new patient, add an optional subgroup dropdown. The subgroup options available should be drawn from whatever subgroups have been defined against consent types.
- **Consent Relevance by Subgroup**: The system should use the patient's subgroup to determine which ICF versions are relevant for that patient. For example a patient in Subgroup 1 should only need to have signed ICFs that are designated for Subgroup 1 or for all patients. ICFs designated for Subgroup 2 should not appear as required for a Subgroup 1 patient.

## Phase 5 - Staff Tab Improvements

- Clicking Role badge edits role inline
- Clicking Systems field opens system access update
- Clicking Status shows start/end date fields
- PI appears at top of list by default
- Drag and drop to reorder staff to match delegation log order
- **Manual override for undetected documents** - CRA needs a way to manually mark a specific document as present in Veeva/TMF when the system hasn't auto-detected it. This is different from the existing ISF filing checkbox. Example: Privacy Statement or FDF exists in Veeva but the system flags it as missing due to a naming mismatch. CRA should be able to click something on that document row to say "this document IS in Veeva, the system just didn't find it" and the missing alert should clear.
- **CV requirements** - needs design decision: CVs are only required in Veeva for PI and Sub-I. Other roles (especially Pharmacist) don't need CV in Veeva but CRA may want to record that CV exists at site. Need to decide how to handle: configurable per role vs simple dual column (CV in Veeva / CV at site). Currently system flags missing CVs for all roles which causes false alerts for Pharmacists.

## Phase 6 - Formatted Excel Export - COMPLETE

Design notes: The current export is functional and fully formatted - colours, headers, separate tabs - so the downloaded file is usable as a standalone document mirroring the HTML tracker.

Implemented features:
- Summary sheet with site info, export date, per-section status table
- 12 per-tab document sheets with tab-specific summary metrics
- Staff Details and Staff Qualifications Matrix (per-cell green/red/grey coding)
- Patient Consents sheet with PICF Version Management + Patient Consent Tracker
- ISF-Only, IP Shipments, Handover sheets
- Dark blue headers with white bold text, row colour coding (green=Filed, amber=Pending, grey=N/A)
- Auto-fit column widths
- Backward-compatible import (handles both old and new format)

---

## Backlog - Issues Identified During Live Use

### 1. Reconciled Documents Sort Order

When a document is ticked as reconciled it moves to the bottom of the list in some tabs but not others. This should be consistent across all tabs - decide on one behaviour and apply it everywhere.

### 2. Consent Tab - Patient Management

Need the ability to:
- Delete a patient that has been created
- Edit patient details after creation
- Edit or delete ICF entries that have been added to a patient

### 3. Colour Coding - New vs Filed

Both 'New' and 'Filed' statuses are currently showing as green which makes them indistinguishable. 'New' should be changed to blue to match the original design intent.

### 4. Duplicate Document Names

When two documents have identical names, changes made to one (such as reconciling) are incorrectly applied to both. The system should treat documents with the same name as entirely separate items with independent state.

### 5. SIV Report Classification

The SIV Report is currently marked as TMF-only but this is incorrect - it can be reconciled at site. This classification needs to be corrected.

### 6. Correspondence Tab - Reconciliation

Two issues:
- The reconciliation control in the Correspondence tab is still a dropdown instead of a checkbox like all other tabs.
- When 'Done' is selected via the dropdown it does not automatically update the reconciliation date to the current date.

Both need to be fixed to match the behaviour of other tabs.

### 7. Move Document Function

Two issues:
- The Move Document option is only present in some tabs, not all. It should be available in every tab.
- The dropdown menu for selecting the destination tab does not include all possible tabs - the full list of tabs should be available as destinations.

---

## Technical Notes

- Single HTML file, no build system, vanilla JavaScript
- Data stored in browser localStorage
- Export/import via xlsx-js-style (SheetJS fork with styling support)
- CLAUDE.md exists in project folder - Claude Code reads it automatically each session
- Proxy config: credentials stored in `%USERPROFILE%\.claude-code-router\config.json` - update if Bayer password changes
- Node.js installed portable at `C:\Users\BTZMD\OneDrive - Bayer\Files\MGA\node-v24.14.1-win-x64`
- Router config at `%USERPROFILE%\.claude-code-router\`
