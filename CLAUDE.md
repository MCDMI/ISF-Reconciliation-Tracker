# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a collection of operational tools for clinical trial document management (CRA workflow). The primary artifact is a self-contained single-page HTML application for tracking TMF/ISF document reconciliation.

## Running the Application

- Open the most recent `TMF_ISF Tracker vX.X.html` file directly in a browser. No build step required.
- External dependency: SheetJS (xlsx 0.18.5) loaded from CDN at runtime.
- Data persists in browser localStorage. Key may vary by version — check the current file if needed.

## Architecture (TMF_ISF Tracker)

Single HTML file (~800 lines) containing all CSS, JavaScript, and markup. No framework — vanilla JS with DOM manipulation.

### Data Layer
- All state stored in localStorage as a JSON object (`allSitesData`) keyed by site
- Each site contains: `siteDocuments`, `countryDocuments`, `studyDocuments`, `isfTracking`, `staffList`, `patients`, `patientConsents`, `picfVersions`, `ipShipments`, `relabelingEntries`, `pharmacyTempLogs`, `sampleTempLogs`, `pharmacyCalibration`, `sampleCalibration`, `localLabs`, `handovers`
- Excel import/export via SheetJS (XLSX library)

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
