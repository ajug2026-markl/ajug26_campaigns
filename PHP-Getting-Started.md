# AJUG Campaign Demo: PHP Event Calendar Getting Started

This repository contains a lightweight PHP application that reads campaign/event data from a Microsoft Dynamics 365 Business Central OData V4 endpoint and presents it as a responsive event calendar and an iframe-friendly upcoming-events widget.

It is intended as an integration/design sample for technical users. The public copy is sanitized. It contains no production credentials, company-specific hostnames, or production store names.

## What to download

Download or clone the entire PHP project and keep its folders together with their existing names and relative paths.

The important areas are:

- `config/`
- `environment/`
- `src/`
- `public/`
- `tests/`

The Power BI project and presentation PDF are not required to run the PHP sample.

Do not download only `public/index.php` or `public/widget.php`. Both pages depend on the shared Business Central client, campaign repository, normalization layer, field mappings, and lookup files under `src/`.

## Prerequisites

- A Linux or Windows web server capable of running PHP 8.2 or later.
- PHP CLI for testing.
- PHP cURL extension.
- PHP mbstring extension.
- Network access from the PHP server to your Business Central OData V4 endpoint.
- A Business Central account/API credential that can read the published campaign/event entity.
- A published Business Central OData page or API containing the fields you want to expose.

Apache is used in the example environment, but the PHP code does not depend on Apache specifically.

See `environment/ENVIRONMENT_SETUP.md` for a basic server setup example.

## Files you must configure

### 1. Business Central connection

Start with:

`config/business-central.ini.example`

Copy it to the location your environment will use and replace the placeholders with your Business Central settings.

The sample configuration includes:

- `base_url`
- `company`
- `tenant`
- `entity`
- `username`
- `password`
- `timezone`
- connection/request timeouts

Example endpoint pattern:

`https://bc.example.com/<instance>/ODataV4/Company('Company Name')/LMCampaignDetail?tenant=<tenant>`

The sample PHP pages expect the configuration file path shown in their `BC_CONFIG_FILE` constant. Change that path if you store the configuration elsewhere.

### 2. Business Central field mapping

Edit:

`src/campaigns/CampaignFields.php`

This file intentionally separates stable application concepts from the raw Business Central field names.

The sample mapping uses fields such as:

- `No`
- `Description`
- `Status_Code`
- `Starting_Date`
- `Ending_Date`
- `Last_Date_Modified`
- `AJE_Store_No`
- `AJE_CRM_Enable_On_POS`
- `AJE_Campaign_Attr_01`
- `AJE_Campaign_Attr_05`
- `AJE_Campaign_Bool_Attr_01`
- `AJE_Campaign_Bool_Attr_02`
- `AJE_Campaign_Bool_Attr_03`
- `AJE_Campaign_Bool_Attr_04`
- `AJE_Campaign_Date_Attr_01`

Your Business Central page/API does not need to use these exact names. Update the mapping so the application-level fields point to the fields published by your environment.

The PHP application should continue referring to stable names such as:

- `campaign_no`
- `description`
- `status`
- `start_date`
- `end_date`
- `store_no`
- `event_type`
- `brand_focus`
- `hidden_on_calendar`
- `hidden_on_comm_site`
- `outbounding_required`

### 3. Lookup values

Review the files under:

`src/lookups/`

The sample includes separate lookups for:

- stores/locations
- event types
- brand/vendor focus

Replace the example `Store 1`, `Store 2`, etc. values with your own location mapping.

These are intentionally separate from the main campaign code so they can later be replaced by another Business Central endpoint, SQL table, or other master-data source without rewriting the calendar pages.

## Business rules demonstrated

### Full calendar

The full calendar is served from:

`public/index.php`

The sample applies these rules:

- Current/future events are determined by effective end date, not only start date.
- An event that started earlier but ends today or later remains visible.
- Events whose effective end date is before today are removed.
- `Hidden on Calendar = Yes` removes an event from the full calendar.
- Blank status is treated as `ACTIVE`.
- `TENTATIVE` events remain visible and are visually emphasized.
- `CANCELLED` events remain visible and are struck through.
- Store/location and Event Type can be filtered in the browser.
- The page is responsive for desktop, tablet, and phone.
- Important interaction is touch-friendly and does not depend on hover.

### Embedded widget

The compact widget is served from:

`public/widget.php`

The sample applies these rules:

- Look ahead up to 50 days.
- Display a maximum of 10 visible events.
- Remove events whose effective end date is before today.
- Exclude `Hidden on Calendar = Yes`.
- Exclude `Hidden on Comm Site = Yes`.
- Sort by start date.
- Use the store lookup value as the location.

The widget can be embedded in SharePoint or another portal with a normal iframe.

Example:

```html
<iframe
  src="https://calendar.example.com/widget.php"
  width="100%"
  height="240"
  style="border:0; overflow:hidden"
  scrolling="no"
  title="Upcoming Events">
</iframe>
```

## Data normalization

The normalization layer is in:

`src/campaigns/CampaignNormalizer.php`

The sample demonstrates several useful Business Central cleanup rules:

- blank status -> `ACTIVE`
- `TENATIVE` -> `TENTATIVE`
- `0001-01-01` -> null
- `0000-00-00` -> null
- common Yes/No/true/false representations -> boolean
- lookup code -> friendly display value

Unknown lookup values are passed through rather than silently discarded. This makes newly introduced Business Central values easier to identify during development.

## Test the Business Central connection

Before troubleshooting the browser pages, run the CLI tests.

From the project root:

```bash
php tests/test-business-central.php
php tests/test-campaign-model.php
```

The first test verifies the Business Central/OData connection.

The second verifies that campaign records can be retrieved and normalized into the application's stable field model.

This is useful for separating:

- Business Central/API problems
- field-mapping problems
- web server/browser problems

## Open and validate

1. Configure the Business Central connection.
2. Update the field mappings.
3. Update the store, Event Type, and Brand/Vendor lookup files.
4. Run both CLI tests.
5. Point your web server at `public/`, or copy that directory into an existing web root while preserving the source paths expected by the PHP files.
6. Open `index.php` in a browser.
7. Verify current, multi-day, tentative, cancelled, and hidden events.
8. Test the location and Event Type filters.
9. Open `widget.php` directly.
10. Confirm that past one-day events are removed and that current multi-day events remain.
11. Test the widget inside an iframe if you plan to use it in SharePoint or another portal.
12. Resize the calendar for desktop, tablet, and phone layouts.

## Architecture to preserve when adapting the project

The project is intentionally split into simple layers:

```text
Business Central OData
        |
        v
BusinessCentralClient
        |
        v
CampaignRepository
        |
        v
CampaignNormalizer
        |
        +---- CampaignFields
        |
        +---- Lookups
        |
        v
Calendar / Widget / Other Consumers
```

The most important design rule is:

**Do not spread raw Business Central custom field names throughout the application.**

If `Event Type` is stored in `AJE_Campaign_Attr_01` today and later moves to a native field, changing one mapping is much easier than finding that field name across multiple pages and integrations.

## AI / coding-assistant use

The repository also includes:

`AI_KICKSTART.md`

You can upload that file together with the source tree into ChatGPT, Codex, Claude, Copilot, or another coding assistant.

A useful starter instruction is:

```text
Read AI_KICKSTART.md, TECHNICAL_GUIDE.md, and the PHP source tree first.

I want to adapt this Business Central Event Calendar sample to my own environment.

Keep the existing separation between:
BusinessCentralClient -> CampaignRepository -> CampaignNormalizer -> CampaignFields/lookups -> presentation.

First ask me for the Business Central endpoint, company, tenant, published entity name, field mappings, and lookup values you need.
```

## Before publishing or sharing your derivative

- Replace sample company/location values with values appropriate for your environment.
- Remove credentials before publishing the project.
- Search the project for production hostnames, URLs, usernames, email addresses, tenant names, and company-specific values.
- Test the downloadable copy separately from your production installation.
- Confirm that example configuration files contain placeholders rather than live credentials.

## Troubleshooting

- **The service root works but the event page returns no records:** verify the Business Central company, tenant, published entity, and whether the page is returning data after an upgrade.
- **The Business Central entity returns HTTP 404:** confirm that the OData page/API is still published and that its service/entity name has not changed.
- **The endpoint times out with curl error 28 / HTTP 000:** the server cannot currently reach the Business Central listener; check the hostname, port, listener, routing, or firewall path.
- **The API returns data but the calendar says it is unavailable:** run the CLI model test and confirm that every field in `CampaignFields.php` still exists in the published entity.
- **Yesterday's event still appears:** verify that the local presentation filter compares the effective end date with today's date.
- **The widget fails after adding a date filter:** if `$today` is used inside an anonymous PHP function, make sure the closure imports it with `use ($today)`.
- **A multi-day event disappears too early:** filter by effective end date rather than only start date.
- **A new Event Type displays as its raw code:** add it to `src/lookups/event_types.php`, or replace the local lookup with your preferred reference-data source.
