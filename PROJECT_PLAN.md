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

### 1. Reconciled Documents Sort Order - FIXED

When a document is ticked as reconciled it moves to the bottom of the list in some tabs but not others. This should be consistent across all tabs - decide on one behaviour and apply it everywhere.

**Fix:** Extracted `sortDocsByStatus()` helper and applied it to all custom tab renderers (IRB, Correspondence, Legal, Manuals, Misc). All tabs now sort consistently: pending > new > filed > NA.

### 2. Consent Tab - Patient Management - FIXED

Need the ability to:
- Delete a patient that has been created
- Edit patient details after creation
- Edit or delete ICF entries that have been added to a patient

**Fix:** Added `deletePatient()`, `editPatientId()`, and `deleteConsent()` functions. Patient cards now have Edit ID and Delete buttons in the header, and each consent row has a delete (×) button.

### 3. Colour Coding - New vs Filed - ALREADY FIXED (pre-v4.4)

Both 'New' and 'Filed' statuses are currently showing as green which makes them indistinguishable. 'New' should be changed to blue to match the original design intent.

**Status:** Already resolved — CSS has `.status-new` as blue (#e3f2fd) and `.status-filed` as green (#d4edda).

### 4. Duplicate Document Names - FIXED

When two documents have identical names, changes made to one (such as reconciling) are incorrectly applied to both. The system should treat documents with the same name as entirely separate items with independent state.

**Fix:** Added `assignDupIndices()` function that detects documents sharing the same key and assigns a `dupIndex` to disambiguate them. `generateKey()` now appends `|dup{n}` for 2nd+ duplicates. Called on data load, file upload, and manual doc creation.

### 5. SIV Report Classification - ALREADY FIXED (pre-v4.4)

The SIV Report is currently marked as TMF-only but this is incorrect - it can be reconciled at site. This classification needs to be corrected.

**Status:** Already resolved — "Site Initiation Process Report" is NOT in `tmfOnlyClassifications`.

### 6. Correspondence Tab - Reconciliation - ALREADY FIXED (pre-v4.4)

Two issues:
- The reconciliation control in the Correspondence tab is still a dropdown instead of a checkbox like all other tabs.
- When 'Done' is selected via the dropdown it does not automatically update the reconciliation date to the current date.

**Status:** Already resolved — Correspondence tab uses the standard ISF checkbox with `toggleIsfCheckbox()`.

### 7. Move Document Function - FIXED

Two issues:
- The Move Document option is only present in some tabs, not all. It should be available in every tab.
- The dropdown menu for selecting the destination tab does not include all possible tabs - the full list of tabs should be available as destinations.

**Fix:** Added move button column to IRB, Correspondence, Legal, Manuals, and Misc tab renderers. Added ISF-Only and Handover to `moveTabOptions`. Integrated `getDocsForTab()` filtering into custom renderers so moved docs appear/disappear correctly.

---

## Backlog - Enhancements Identified

### 8. Excel Export - Staff Grid

The exported Excel should include the full staff summary grid as it appears in the HTML (name, role, and status of each essential document per staff member). Currently the export does not replicate this view.

### 9. Staff Tab - Filter by Missing Document

Add filter controls to the staff summary grid so the CRA can filter by document type (e.g. show only staff missing CV, or missing FDF). This brings staff needing attention to the top for ease of collection.

### 10. Staff Tab - Group Documents by Category

The staff documents section currently renders as one long list. Split into category headings: Privacy Statements, CVs, Medical Licenses, Financial Disclosure Forms, GCP Certificates, Qualification Supporting Information. Each group under its own sub-heading.

### 11. Staff Tab - Clickable Column Headers to Sort by Missing

The staff qualifications grid column headers (Privacy, CV, Med Lic, GCP, FDF, IATA) should be clickable. Clicking a column header sorts the grid so that staff missing that document appear at the top. This is more intuitive than the dropdown filter (which still works as an alternative).

---

## Phase 7 - Staff Training Bulk Entry & Tracking

**Problem:** Training entry is currently one row at a time per staff member via the inline dropdown + date picker (`addTrainingInline`). Recording that all staff completed SIV training, or that all Pharmacists completed a specific training on the same date, requires repeating the same entry for every person. Very manual.

**Data model context:** Each staff member has a `training` array of `{name, dateCompleted}` objects on `staffList[]`. Known training names are collected in `usedTrainingNames` (site-level, saved with site data). Roles come from the fixed role list ('PI','Sub-I','Study Coordinator','Research Nurse','Pharmacist','Lab Staff','Data Entry','Regulatory/Ethics','Staff','Other'). Any new feature must reuse this structure — no schema changes needed except where noted.

**Build order:** 7.1 → 7.2 → 7.4 → 7.3 → 7.5 → 7.6. Each item is independently shippable; 7.1 alone solves the core pain point.

### 7.1 Bulk Add Training Modal (build first — highest value, lowest risk)

- "+ Bulk Add Training" button in the Staff Summary header next to "+ Add Staff"
- Modal contains: training name dropdown (from `usedTrainingNames` plus "+ New training..." option, same pattern as inline add), single date picker, and a checkbox list of staff
- Quick-select controls: "Select All Active" (active = no `delegationStopDate`), and per-role select buttons (e.g. tick all Pharmacists in one click)
- On save: add the training to every selected staff member; skip silently if that staff member already has the same training name + date (dedupe); add name to `usedTrainingNames` if new
- Covers both key scenarios: "everyone did SIV on date X" and "all Pharmacists did training Y on date X"

### 7.2 Training Matrix View

- Toggle on Staff tab: list view ↔ matrix view
- Matrix: rows = staff (active first, delegation-log order), columns = training names from `usedTrainingNames`, cells = completion date or empty
- Click empty cell → quick date entry (offer a "default date" field at top of matrix so repeated clicks reuse the same date)
- Click column header → "mark all staff without this training as complete on [date]"
- Empty cells double as a training gap analysis for monitoring visits
- Consider adding this matrix to the Excel export (same green/red/grey per-cell coding as the existing Staff Qualifications Matrix sheet)

### 7.3 Copy Training from Another Staff Member

- When adding new staff mid-study, a "Copy training from..." dropdown listing existing staff; clones their training array (option: keep original dates vs enter a single new date)

### 7.4 Paste-from-Excel Training Import

- Textarea import: paste rows of Name / Training / Date (tab- or comma-separated, e.g. from a sponsor LMS report)
- Fuzzy-match names against `staffList` (reuse the last-name matching approach from `extractStaffFromDocuments`)
- Show a preview/confirm table of matched and unmatched rows before committing; unmatched rows can be skipped or assigned manually

### 7.5 Training Templates per Role (optional — design decision needed)

- Define required trainings per role (e.g. Pharmacist = IP Handling, IRT, Temp Excursion)
- New staff auto-populate those trainings as pending (no date); requires allowing `dateCompleted` to be empty and rendering pending rows distinctly
- Pending items surface as gaps; complete them via 7.1 or 7.2
- Decide: templates configurable per site vs hardcoded defaults

### 7.6 Training Log Photo/Scan Upload with OCR (build last — experimental)

- Add Tesseract.js via CDN (same pattern as the xlsx-js-style include; note: confirm CDN access works through the Bayer proxy)
- User uploads a photo/scan of a training log → OCR extracts text → parse training name, date, and attendee names → fuzzy-match against `staffList` → mandatory review/confirm table before anything is committed
- Reality check: handwritten sign-in sheets OCR poorly; typed sheets and LMS printouts work well
- Fallback mode if name recognition is unreliable: OCR extracts only training name + date from the header, then opens the 7.1 bulk modal pre-filled so the CRA just ticks attendees

---

## Technical Notes

- Single HTML file, no build system, vanilla JavaScript
- Data stored in browser localStorage
- Export/import via xlsx-js-style (SheetJS fork with styling support)
- CLAUDE.md exists in project folder - Claude Code reads it automatically each session
- Proxy config: credentials stored in `%USERPROFILE%\.claude-code-router\config.json` - update if Bayer password changes
- Node.js installed portable at `C:\Users\BTZMD\OneDrive - Bayer\Files\MGA\node-v24.14.1-win-x64`
- Router config at `%USERPROFILE%\.claude-code-router\`
