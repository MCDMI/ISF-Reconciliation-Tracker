# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a collection of operational tools for clinical trial document management (CRA workflow). The primary artifact is a self-contained single-page HTML application for tracking TMF/ISF document reconciliation.

## Running the Application

- Open the most recent `TMF_ISF Tracker vX.X.html` file directly in a browser. No build step required.
- External dependency: xlsx-js-style (1.2.0) loaded from CDN at runtime. This is a fork of SheetJS that supports cell styling on write.
- Data persists in browser localStorage. Key may vary by version — check the current file if needed.

## Architecture (TMF_ISF Tracker)

Single HTML file (~800 lines) containing all CSS, JavaScript, and markup. No framework — vanilla JS with DOM manipulation.

### Data Layer
- All state stored in localStorage as a JSON object (`allSitesData`) keyed by site
- Each site contains: `siteDocuments`, `countryDocuments`, `studyDocuments`, `isfTracking`, `staffList`, `patients`, `patientConsents`, `picfVersions`, `ipShipments`, `relabelingEntries`, `pharmacyTempLogs`, `sampleTempLogs`, `pharmacyCalibration`, `sampleCalibration`, `localLabs`, `handovers`
- Excel import/export via xlsx-js-style (SheetJS fork with styling support)

### Excel Export (exportToExcel)
The export was rebuilt in this session to produce a formatted multi-sheet report. Current state:
- **Summary sheet**: site info, export date, per-section status table (Total/Filed/Pending)
- **12 per-tab document sheets**: Staff, Consent, IRB-IEC, Protocol-IB, IP, Labs, Monitoring, Correspondence, Legal, Setup, Manuals, Misc
- **Supplementary sheets**: Staff Details, Staff Qualifications (matrix with per-cell colour), Patient Consents, ISF-Only, IP Shipments, Handover
- **Styling**: dark blue headers with white bold text, row colour coding (green=Filed, amber=Pending, grey=N/A), auto-fit column widths
- **Import**: updated to handle both old format (single 'All Documents' sheet) and new per-tab format

#### Remaining export work (not yet implemented):
1. Consent sheet needs two sections: PICF Version Management table + Patient Consent Tracker (per-patient with amber/green colour coding)
2. Tab summary metrics block at top of each sheet (matching the summary metrics shown in the tracker UI for each tab)


### UI Structure (15 tabs)
Dashboard | Staff | Consent | IRB/IEC | Protocol/IB | IP | Labs | Monitoring | Correspondence | Legal | Setup | Manuals | Misc | ISF-Only | Handover

### Key Workflows
- TMF Excel export is uploaded (drag-and-drop or file picker), parsed, and split by site/country/study level based on configured site number
- Staff members are auto-extracted from document names via regex
- Users track ISF filing status, review dates, and comments per document
- Alert system flags: expiring medical licenses, lab accreditations, outstanding consents, incomplete staff docs
- Export produces a multi-sheet Excel workbook

## Development Notes

- No build system, no tests, no linter — edit the HTML file directly
- When modifying JavaScript, all logic lives in `<script>` tags within the single HTML file
- CSS is inline in a `<style>` block at the top
- The app must remain a single self-contained file (portable, runs offline except for CDN dependency)
- Version numbering follows `vX.Y` pattern in the filename and page title
- Keep a BACKUP copy of the previous version when making significant changes
