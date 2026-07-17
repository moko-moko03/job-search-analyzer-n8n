# job-search-analyzer-n8n

The workflow searches for job opportunities using filters generated from the user's profile and analyzes how well each job matches the user's background using AI.

## What's included

This repository contains four n8n workflows in the [workflows/](workflows/) folder:

| File | Role |
| --- | --- |
| `Job search & analyze.json` | The main workflow. It searches for jobs, uses OpenAI to score how well each job matches your profile, and saves the results to a spreadsheet. |
| `Sub-Workflow-Logging.json` | A shared sub-workflow that writes run logs to a spreadsheet. It's called by the other three workflows. |
| `Sub-workflow-OpenAI-Error.json` | A sub-workflow that retries OpenAI calls on failure and logs errors. |
| `Error logging.json` | Runs automatically whenever another workflow fails, and forwards the error details to `Sub-Workflow-Logging`. |

## Data source

Job listings are fetched from the [Himalayas Remote Jobs API](https://himalayas.app/api), which provides remote job postings. This API is free to use and doesn't require authentication or an API key.

## Prerequisites

- A running n8n instance (n8n Cloud or self-hosted)
- A Google account (used for Google Sheets)
- An OpenAI API key (create one at [platform.openai.com](https://platform.openai.com/))

## Setup guide

### 1. Download and import the workflows

1. Download all four `.json` files from the [workflows/](workflows/) folder to your computer.
2. In n8n, go to the **Workflows** list and click **Import from File** (or, inside a workflow, use the "..." menu in the top right → **Import from File**). Import all four files, one at a time.
3. Check that all four workflows now appear in your n8n workspace.

> **Note:** After importing, the "Execute Workflow" nodes that call a sub-workflow (for example `Call 'Sub-Workflow-Logging'`) may still point at the workflow IDs from the original export, so they can show as "not found." This is fixed in step 5 below — don't worry about it yet.

### 2. Set your spreadsheet URLs

These workflows use three of your own Google Sheets: one for job data, one for logs, and one for your profile. Enter their URLs into the "Edit Fields" node at the very start of each workflow:

> **Tip:** Ready-to-use Google Sheets templates are available in the [spreadsheet/](spreadsheet/) folder — download and copy them into your own Google Drive if you'd rather not build the sheets from scratch. Do not rename any of the columns in these templates, since the workflows reference them by name and will stop working correctly if the column names are changed.

1. Open **`Job search & analyze`** and double-click the **`Edit Fields4`** node — the node directly connected to the trigger (`When clicking 'Execute workflow'`). It has three fields:
   - `sheetLog` — URL of your log spreadsheet
   - `sheetJob` — URL of your job-data spreadsheet
   - `sheetProfile` — URL of your profile spreadsheet
   
   Replace the placeholder value (`YOUR_SPREADSHEET_URI`) in each field with the URL of your own spreadsheet. They can be three separate spreadsheets, or three tabs in one spreadsheet — either works, as long as each URL points to the right one.
2. Open **`Sub-Workflow-Logging`** and double-click the **`Edit Fields`** node right after its trigger (`When Executed by Another Workflow`). It has one field:
   - `sheetLog` — set this to the **same URL** you used for the log spreadsheet in step 1, so both workflows write to the same place.

> **Note:** Every node reads from the first tab of each spreadsheet (`gid=0`, usually named `Sheet1`). If you're creating new spreadsheets, it helps to add a header row on that first tab in advance:
>
> - Job-data sheet: `id, title, companyName, employmentType, currency, minSalary, maxSalary, salaryPeriod, seniority, locationRestrictions, categories, description, pubDate, expiryDate, applicationLink, source, addDate, skills, jobFit, jobFitReason, preferencesMatch, preferencesMatchReason`
> - Log sheet: `Run ID, Start, Status, End, Jobs Added, Workflow Name, Node Name, Error Message`
> - Profile sheet: see step 6 below.

### 3. Set up credentials (Google Sheets & OpenAI)

In n8n, go to **Credentials** in the left sidebar and create the following:

- **Google Sheets (OAuth2)** — sign in with your Google account and grant access to Google Sheets.
- **OpenAI API** — enter the API key you created earlier.

> If you already have credentials of these types from another project, you can reuse them instead of creating new ones.

### 4. Connect the credentials to the workflows (Setup tab)

1. Open **`Job search & analyze`** and click **Open side panel** in the top right.
2. Go to the **Setup** tab. It lists the credentials this workflow needs (Google Sheets account, OpenAI account).
3. For each item, select the credential you created in step 3.
4. Do the same for **`Sub-Workflow-Logging`**: open it, click **Open side panel** → **Setup**, and connect its Google Sheets credential.

### 5. Reconnect the sub-workflow calls

Right after importing, the **Execute Workflow** nodes inside `Job search & analyze`, `Sub-workflow-OpenAI-Error`, and `Error logging` (the nodes named things like `Call 'Sub-Workflow-Logging'`) may show an error because they still reference the original workflow IDs. Open each of these nodes and reselect the correct target — `Sub-Workflow-Logging` or `Sub-workflow-OpenAI-Error` — from the dropdown list.

### 6. Set up your User Profile spreadsheet

In the spreadsheet you set as `sheetProfile` in step 2, add a header row on the first tab with your profile details, and one data row below it with your own information:

| Column | Example |
| --- | --- |
| `userSkills` | Your skills (e.g. JavaScript, SQL, project management) |
| `userExperience` | A summary of your work experience |
| `workPreference` | Your preferred work style (e.g. remote, full-time) |
| `location` | Your preferred work location |
| `targetPosition` | The job title/position you're targeting |
| `salaryExpectation` | Your expected salary range |
| `preferredIndustry` | Your preferred industry |
| `preferredRole` | Your preferred role |

If you already have a `User Profile` spreadsheet, just make sure the column names match the ones above.

### 7. Publish the sub-workflows

`Sub-Workflow-Logging` and `Sub-workflow-OpenAI-Error` are called by other workflows via "Execute Workflow" nodes. Open each one and turn on the **Publish** toggle in the top right — if it's left off, calls to these sub-workflows will fail.

> `Error logging` also needs to be published, since it relies on an `Error Trigger` that must be active to catch errors from other workflows. `Job search & analyze` is triggered manually, so it doesn't need to be published.

### 8. Set the error notification workflow

1. Open **`Job search & analyze`**, and from the "..." menu choose **Settings**.
2. Under **Error Workflow (to notify when this one errors)**, select **`Error logging`**.
3. Now, if `Job search & analyze` fails during a run, `Error logging` will automatically start and record the error in your log spreadsheet via `Sub-Workflow-Logging`.

## Running the workflow

Once everything above is set up, open `Job search & analyze` and click **Execute workflow**. Matching jobs, along with an AI-generated fit score, will be added to your job-data spreadsheet.
