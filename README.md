# Laboratory Custom App

`labratory_custom_app` is a Frappe application that extends the Healthcare
laboratory workflow. It adds a laboratory dashboard and customizes Lab Test
template handling, including age- and gender-specific reference ranges.

The project has a Vue 3 frontend in `frontend/` and a Python/Frappe backend in
`labratory_custom_app/`. The frontend dashboard calls a whitelisted Frappe API;
the Lab Test customization is installed through Frappe hooks.

## Features

### Laboratory dashboard

- Shows summary cards for Lab Tests, Sample Collections, Patients, Sales
  Invoices, and paid invoice amounts.
- Filters date-based totals and lists by a selected start and end date. The
  dashboard initially selects today.
- Displays status counts for Lab Tests, Sample Collections, and Sales Invoices.
  Lab Test status totals include Draft and Completed.
- Lists up to 10 Lab Tests and 10 Sales Invoices in the selected date range.
- Provides shortcuts to patient, sample collection, Lab Test, report, and Sales
  Invoice records in the Frappe Desk.
- Reports the total number of Patients across all dates; the other date-based
  summary figures follow the selected range.

### Lab Test workflow customization

- Overrides the Healthcare `Lab Test` controller through Frappe's
  `override_doctype_class` hook.
- Supports selecting multiple Lab Test Templates on a Lab Test and creates
  result rows for the selected template types (Compound, Single, Descriptive,
  and Imaging).
- Loads templates associated with invoiced laboratory items and links the
  invoice items to the resulting Lab Test.
- Removes generated rows for templates that are deselected and tags generated
  rows with their test group.
- Selects a patient's reference range using their age and sex when a template's
  normal range contains age/sex-specific entries.

### Frappe integration

- Declares Custom Field fixtures for Patient, Lab Test, Normal Test Result,
  Normal Test Template, and Lab Test Template.
- Registers the Laboratory app on the Frappe Apps screen and includes the
  compiled frontend JavaScript in Desk.
- Ensures the app's Module Def records exist after installation and migration.

## Requirements

- A working [Frappe Bench](https://github.com/frappe/bench) and site.
- The following Frappe apps must be installed on the bench and target site
  before installing `labratory_custom_app`:

  | App          | Required for                                                                                          |
  | ------------ | ----------------------------------------------------------------------------------------------------- |
  | `frappe`     | Frappe Framework, which provides the app runtime and core features.                                   |
  | `healthcare` | The Lab Test controller and the Patient, Sample Collection, Lab Test, and Lab Test Template DocTypes. |
  | `erpnext`    | The Sales Invoice DocType used by the dashboard and invoice-to-Lab-Test workflow.                     |

- Python and Frappe versions supported by the bench you use. This app's
  `pyproject.toml` currently declares Python `>=3.14`; confirm that your Frappe
  stack supports that Python version before installing.
- Node.js and npm for frontend development and builds.

Use versions of Frappe, Healthcare, and ERPNext that are compatible with one
another and with your bench. This app does not pin their versions in its
dependency metadata.

## Installation

From your bench directory, fetch the app:

```bash
cd /path/to/your/frappe-bench
bench get-app https://github.com/erp-quantumberg/erpnext-healthcare-labratory_custom_app.git --branch main
```

Install it on the target site:

```bash
bench --site your-site.example install-app labratory_custom_app
```

If the app is already present in `apps/`, skip `bench get-app`. Replace
`your-site.example` with the site name configured in your bench.

## Setup and development

### Run the Frappe site

In the bench directory, start the bench services:

```bash
bench start
```

### Run the frontend development server

In a second terminal:

```bash
cd /path/to/your/frappe-bench/apps/labratory_custom_app/frontend
npm install
npm run dev
```

Open the site through the Vite development server, usually at
`http://your-site.example:8080`. The Vite configuration proxies requests to the
Frappe web server using the `webserver_port` in
`sites/common_site_config.json`.

### Build frontend assets

The production frontend is served from the app's public assets directory. Build
it after changing frontend code:

```bash
cd /path/to/your/frappe-bench/apps/labratory_custom_app/frontend
npm install
npm run build
```

The Vite build writes files to
`labratory_custom_app/public/frontend/`. From the bench directory, rebuild the
Frappe app assets:

```bash
bench build --app labratory_custom_app
```

The app's hooks include the compiled JavaScript in Desk. The frontend package
also provides `npm run preview` for previewing a built frontend locally.

### Run tests

From the bench directory, run the app's Frappe tests on a test site:

```bash
bench --site your-test-site.example run-tests --app labratory_custom_app
```

## Configuration notes

- The dashboard API is
  `labratory_custom_app.api.lab_dashboard.get_dashboard_data`, available at
  `/api/method/labratory_custom_app.api.lab_dashboard.get_dashboard_data`.
- The API accepts `from_date` and `to_date` in `YYYY-MM-DD` format. If either
  value is omitted, it defaults to today.
- The custom Lab Test controller is registered in `hooks.py`. Keep the
  Healthcare app installed and its Lab Test controller importable.
- The frontend development server reads
  `sites/common_site_config.json` to determine the Frappe web server port. Run
  Vite from the `frontend/` directory within a bench that has this file.
- For age/sex-specific normal ranges, enter one rule per line in the format
  `from_age|to_age|gender|reference_range`, for example:

  ```text
  0|12|all|Range for ages 0 through 12
  13|120|female|Female range for ages 13 through 120
  13|120|male|Male range for ages 13 through 120
  ```

  Age bounds are inclusive. Gender may be `all` or the patient's sex value;
  the value is compared without regard to letter case. Use the same format
  consistently in the source template's normal range.

- Do not enable development-only security settings on a production site.

## Troubleshooting

### The app or Lab Test customization cannot be imported

Confirm that the app is installed on the site and that a compatible Healthcare
app is installed in the same bench. After installing or updating apps, run
`bench --site your-site.example migrate` from the bench directory.

### The dashboard shows an error or no data

Check that the site's Healthcare and ERPNext DocTypes used by the dashboard are
available, and inspect the browser's network request to the dashboard API and
the Frappe server log for the underlying error. Try a date range containing
records. Patient totals are not date-filtered; other summary totals and lists
are.

### Vite cannot start or cannot reach the Frappe site

Run `npm install` from `apps/labratory_custom_app/frontend`, ensure
`sites/common_site_config.json` exists in the bench, and verify that
`webserver_port` matches the running Frappe web server. Also check that the
requested site hostname resolves to your local bench.

### Desk loads an old or missing frontend

Rebuild the frontend with `npm run build`, then run
`bench build --app labratory_custom_app` from the bench directory. Confirm that
the generated files exist under `labratory_custom_app/public/frontend/` and
reload the site after clearing the browser cache.

### A custom reference range is not selected

Check that the Patient has an age and sex, that the template's range lines
contain exactly four `|`-separated values, and that the age bounds are
integers. The patient's age must fall within the inclusive bounds, and the
gender entry must match the patient's sex or be `all`.

## Contributing

Contributions and bug reports are welcome. Before opening a pull request:

1. Make changes in the relevant backend or `frontend/` source files.
2. Run the Frappe tests relevant to your change.
3. For frontend changes, run `npm run build` from `frontend/`.
4. Install and run the repository's pre-commit hooks:

   ```bash
   cd /path/to/your/frappe-bench/apps/labratory_custom_app
   pre-commit install
   pre-commit run --all-files
   ```

Keep pull requests focused and include steps to reproduce or verify behavior
when fixing a bug or adding a feature.

## License

This project is distributed under the MIT License. See [license.txt](license.txt)
for the license terms.
