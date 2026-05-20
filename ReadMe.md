# Automation & QA Developer Take-Home Assessment

## Candidate

**Name:** Sunil Varma Tenali  
**Role:** Automation & QA Developer Assessment

## Submission Overview

This repository contains my completed work for the Automation & QA Developer take-home assessment.

The assessment covers:

1. Web application QA and debugging
2. n8n API integration workflow
3. Bonus uptime monitoring workflow

## Repository Structure

```text

Bonus/
  Bonus_UptimeMonitor_Sunil.json
  screenshots/

Task1/
  Task1_QA_Report_Sunil.pdf
  screenshots/

Task2/
  Task2_Workflow_Sunil.json
  README_Task2.md
  screenshots/
    task2_workflow_canvas.png
    task2_successful_execution.png


README.md
```

---

# Task 1 — Web App QA & Debug Report

## Application Tested

**App:** React Redux RealWorld Example App  
**Repository:** https://github.com/gothinkster/react-redux-realworld-example-app  
**Local URL:** http://localhost:4100  
**Browser:** Google Chrome  
**Testing Type:** Manual QA using browser testing, Chrome DevTools Console, and Network tab

## Scope Tested

The following flows were tested:

- Application startup
- Signup
- Login
- Global Feed loading
- Empty/invalid signup form submission
- API failure handling

## Summary

During testing, I found 5 issues affecting core application flows. The most serious issue was an unsafe error-handling bug in `src/middleware.js`, where the app tries to read `error.response.body` without checking whether `error.response` exists.

This caused the app to crash whenever API requests failed due to CORS, redirects, or network-level failures.

## Key Findings

1. App crashes on startup due to unsafe error handling.
2. Signup fails because the API request is blocked by CORS/API redirect.
3. Login fails because the API request is blocked by CORS/API redirect.
4. Global Feed fails to load due to API/CORS/HTTP 530 issue.
5. Signup form submits empty/invalid data without client-side validation.

## Root Cause Summary

The main root cause is that the frontend middleware assumes all failed API requests return a structured response body. However, CORS and network failures do not always include `error.response`. Because of this, the app throws another TypeError while trying to handle the original API failure.

A safer implementation would check whether `error.response` exists before accessing `error.response.body` and display a user-friendly fallback error message.

## Task 1 Files

- `task1/Task1_QA_Report_Sunil.pdf` — final QA report
- `task1/screenshots/` — screenshots captured during testing

---

# Task 2 — n8n API Integration Workflow

## Workflow Name

**Task2 - GitHub Automation Digest - Sunil**

## Goal

The goal of this workflow is to create an automated digest that watches a public data source, enriches the result with a second API call, applies conditional logic, and stores the final output in an output channel.

This workflow acts like a simple “morning brief” for popular GitHub repositories related to automation.

## APIs Used

### 1. GitHub Search Repositories API

**Endpoint Used:**

```text
https://api.github.com/search/repositories?q=topic:automation&sort=stars&order=desc&per_page=10
```

This API fetches public GitHub repositories related to the topic `automation`, sorted by star count.

### 2. GitHub Repository README API

**Endpoint Used:**

```text
https://api.github.com/repos/{owner}/{repo}/readme
```

This second API request enriches the top repository by fetching README metadata for the selected repository.

## Workflow Steps

1. **Schedule Trigger**
   - Runs the workflow every 1 hour.
   - Simulates an automated periodic digest.

2. **HTTP Request — Search GitHub Repositories**
   - Calls the GitHub Search API.
   - Fetches repositories related to automation.
   - Results are sorted by stars.

3. **Code Node — Prepare Top 5 Repositories**
   - Extracts only the top 5 repositories.
   - Keeps useful fields such as repository name, full name, owner, URL, description, stars, forks, language, and updated date.

4. **HTTP Request — Fetch Top Repository README**
   - Uses the top repository from the first API response.
   - Calls a second GitHub API endpoint to fetch README metadata.
   - This satisfies the enrichment requirement.

5. **IF Node — Check Star Threshold**
   - Checks whether the top repository has more than 1000 stars.
   - If true, the repository is marked as a high-interest item.
   - If false, it is treated as a normal digest item.

6. **Code Node — Build Digest Message**
   - Converts the cleaned repository data into a readable digest message.
   - Includes the top 5 repositories and the reason for the conditional branch.

7. **Google Sheets Output**
   - Appends the final digest to a Google Sheet.
   - Stores timestamp, status, top repository, star count, and digest message.

## Transformation Logic

The GitHub API returns a large JSON response. The workflow transforms it by keeping only the top 5 repositories and extracting the most useful fields for a team digest.

This keeps the output clean, readable, and focused.

## Conditional Logic

The workflow uses an IF node with this condition:

```text
Top repository stars > 1000
```

If the condition is true, the digest is marked as `high_interest`.  
If the condition is false, the digest is handled as a normal digest.

## Error Handling

The HTTP Request nodes are configured so the workflow does not fail silently.

If an API request fails, the workflow creates a fallback error message and logs it to Google Sheets. This helps identify whether the failure occurred during the GitHub search API call or during the README enrichment API call.

## Credentials

Secrets and external service access are not hard-coded in the workflow.

Google Sheets access is handled through n8n’s Google Sheets OAuth2 credential system.

## Task 2 Files

- `task2/Task2_Workflow_Sunil.json` — exported n8n workflow
- `task2/README_Task2.md` — short explanation of the workflow
- `task2/screenshots/task2_workflow_canvas.png` — workflow canvas screenshot
- `task2/screenshots/task2_successful_execution.png` — successful execution screenshot

---

# Bonus Task — Uptime Monitor Workflow

## Workflow Name

**Bonus - Uptime Monitor - Sunil**

## Goal

The bonus workflow monitors the availability of a deployed web app. It checks the app every 5 minutes and logs whether the app is healthy or unavailable.

## Application Monitored

**App URL:** https://month-of-muscle.netlify.app/

## Workflow Steps

1. **Schedule Trigger**
   - Runs every 5 minutes.
   - Creates a simple uptime monitoring system.

2. **HTTP Request — Ping Web App**
   - Sends a GET request to the application URL.
   - Captures the HTTP status code from the response.

3. **IF Node — Check App Health**
   - Checks whether the returned status code is equal to 200.
   - If the status code is 200, the app is considered healthy.
   - If the status code is not 200, the app is treated as unavailable or unhealthy.

4. **Code Node — Build Alert Message**
   - Creates an alert message when the app does not return HTTP 200.
   - The message includes the URL, status code, timestamp, and suggested action.

5. **Code Node — Build Success Log**
   - Creates a healthy status log when the app returns HTTP 200.

6. **Google Sheets Output**
   - Logs success and failure results into Google Sheets.
   - Provides a visible history of uptime checks.

## Health Check Logic

The main condition used in the workflow is:

```text
Status code != 200
```

If this condition is true, the workflow logs an alert.  
If false, it logs the app as healthy.

## Error Handling

The HTTP Request node is configured to continue on failure so request failures can still be logged instead of stopping the workflow silently.

This ensures downtime, invalid responses, or network-level failures are visible in the output channel.

## Bonus Files

- `bonus/Bonus_UptimeMonitor_Sunil.json` — exported n8n uptime monitor workflow
- `bonus/screenshots/bonus_workflow_canvas.png` — bonus workflow canvas screenshot

---

# Loom Video

A Loom video walkthrough is included with the submission.

**Loom Video Link:** [Link for Video](https://drive.google.com/file/d/1cdivaDtfsf0nsXZTgGlyANc8kJO4VpKW/view?usp=sharing)

In the video, I explain:

- The QA approach used in Task 1
- The 5 issues found in the web app
- The root cause of the main error-handling issue
- The Task 2 n8n workflow design
- The API calls, transformation, IF condition, output, and error handling
- The bonus uptime monitor workflow

---
