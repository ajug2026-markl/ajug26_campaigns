# AJUG Campaign Demo: Getting Started

This repository contains a Power BI Project (PBIP) report definition. It is intended as a report-design sample for experienced Power BI users.

The public copy is sanitized. It contains no credentials and does not include the original semantic model or source data. The report is a thin report, so its visuals will not return data until you connect it to a compatible semantic model in your own Power BI/Fabric environment.

## What to download

Download or clone the entire repository and keep these items together with their existing names and relative paths:

- `AJUG Campaign Demo.pbip`
- `AJUG Campaign Demo.Report/` and everything beneath it

The `php-event-cal/` materials and `Presentation-Slides.pdf` are not required to open the PBIP report.

Do not download only the `.pbip` file. It is a small pointer file; the report definition lives in the adjacent `.Report` directory.

## Prerequisites

- A current version of Power BI Desktop that supports Power BI Project (`.pbip`) files and the enhanced PBIR report format.
- Access to a Power BI semantic model with a compatible schema, or permission to create and publish one.
- Build permission on the target semantic model if it is hosted in the Power BI service.
- Optional: access to your own Microsoft Dynamics 365 Business Central environment if you want the report's drill-through hyperlinks to work.

## Files you must configure

### 1. Semantic-model connection

Edit `AJUG Campaign Demo.Report/definition.pbir` before opening the project. Its sanitized connection string contains these placeholders:

- `REPLACE_WITH_YOUR_WORKSPACE`
- `REPLACE_WITH_YOUR_SEMANTIC_MODEL`
- `00000000-0000-0000-0000-000000000000`

Replace them with your Power BI workspace name, semantic-model name, and semantic-model ID. Preserve the JSON quoting and the rest of the connection-string structure.

A practical way to obtain a correctly formatted connection is to create a small PBIP thin report in Power BI Desktop, connect it live to your target semantic model, save it, and copy the `connectionString` value from that report's `definition.pbir` file.

If you prefer to bind the report through Power BI Desktop, open the project, acknowledge the expected connection error from the placeholders, and use the available semantic-model connection/rebind workflow for your installed Desktop version. Save the PBIP afterward and verify that `definition.pbir` now points only to your environment.

### 2. Business Central hyperlink base

In `AJUG Campaign Demo.Report/definition/reportExtensions.json`, find the local measure named `__lm__BC URL Base`. Replace:

`https://businesscentral.example.com/?company=YOUR_COMPANY`

with the root URL and URL-encoded company value for your Business Central environment. Leave the example value unchanged if you do not need Business Central links; the URL is inert and uses the reserved `example.com` domain.

The dependent measures expect Business Central page 20 and fields such as G/L Account No., Global Dimension 2 Code, Shortcut Dimension 3 Code, and Posting Date. Adjust their filters if your Business Central configuration differs.

## Required semantic-model shape

The report definition references model objects including the following tables:

- `DimCampaign`
- `DimDate`
- `DimGLAccount`
- `FactGLEntry`

It also references additional fields and measures embedded throughout the visual definitions. Power BI binds visuals by table, column, and measure name, so a differently named model will show broken visuals until you map or rename those references.

For the smoothest reuse, connect to a model that reproduces the original table/field/measure names. Otherwise, open each page in Power BI Desktop and repair fields and measures in the visual configuration panes. Local report measures are stored in `AJUG Campaign Demo.Report/definition/reportExtensions.json`.

## Open and validate

1. Close any older copy of this report in Power BI Desktop.
2. Configure the semantic-model connection as described above.
3. Double-click `AJUG Campaign Demo.pbip`, or open it from Power BI Desktop.
4. Sign in to the Power BI service when prompted and select credentials for your own environment only.
5. Check every visible and hidden report page for missing fields, error visuals, filters, bookmarks, drill-through behavior, and Business Central links.
6. Save the project, close Power BI Desktop, reopen the `.pbip`, and repeat a quick validation.

The included pages are Campaign / Event List, Sales Event Recap, Customer Spend by Event Segment, Non-Sales Event Recap, Business Initiative Review (Dimension 2), AJUG Conf Notes, and TEST - DimCampaign Dates.

## Before publishing or sharing your derivative

- Search the project text for your company name, tenant/workspace names, hostnames, URLs, email addresses, semantic-model IDs, and local/UNC file paths.
- Do not commit `.pbi/localSettings.json`, `.pbi/cache.abf`, credentials, exported data, gateway details, or access tokens.
- Review static text, tooltips, bookmarks, local measures, custom themes, and hidden pages; sensitive values are not limited to data-source files.
- Confirm that screenshots, PDFs, sample exports, and other files outside the PBIP project are safe before distributing them.
- Test the downloadable copy in a clean folder or with a separate Power BI account before release.

## Troubleshooting

- **The project opens but all visuals fail:** the report is not connected to a compatible semantic model, or model object names do not match.
- **Power BI asks for the original dataset:** a placeholder remains in `definition.pbir`, or the report was not rebound and saved.
- **Business Central links go to `example.com`:** configure `__lm__BC URL Base` as described above.
- **The `.pbip` file appears empty or very small:** this is expected; make sure the adjacent `.Report` directory was downloaded too.
