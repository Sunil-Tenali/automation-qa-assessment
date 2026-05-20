# Task 2 - n8n GitHub Automation Digest

## Overview
This workflow runs every hour and creates a digest of popular GitHub repositories related to automation.

## APIs Used
I used the GitHub REST API because it is public, reliable, and returns structured JSON data.

The first API request searches repositories with the topic "automation" and sorts them by stars.

The second API request fetches README metadata for the top repository to enrich the result.

## Transformation
The workflow keeps the top 5 repositories and extracts only useful fields:
name, full name, URL, description, stars, forks, language, and updated date.

## Conditional Logic
An IF node checks whether the top repository has more than 1000 stars.
If yes, the workflow marks it as a high-interest digest.
If no, it sends a normal digest.

## Output
The final digest is sent to Discord / Google Sheet.

## Error Handling
HTTP Request nodes use Continue On Fail.
If an API call fails, the workflow routes the error to a fallback notification instead of crashing silently.

## Credentials
Secrets such as webhook URLs or API tokens are not hard-coded.
They are stored in n8n credentials or environment variables.