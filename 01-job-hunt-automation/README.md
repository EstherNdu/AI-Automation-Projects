# International Job Hunt Automation

An n8n workflow that automatically finds international Data Analyst job listings twice a day and sends 
email alerts so I never have to manually scan job boards again.

## The Problem

Job hunting often means opening multiple tabs e.g LinkedIn, Indeed and various job boards scrolling for
long stretches, and finding roles that are expired, not remote, or not a good fit. This automation removes that 
manual effort entirely.

## How It Works

Schedule Trigger → HTTP Request → Code → If → Google Sheets + Gmail

## APIs & Tools Used

- [Adzuna Jobs API](https://developer.adzuna.com) (free tier)
- Google Sheets API
- Gmail API

## My Setup Process

I started by signing up at [developer.adzuna.com](https://developer.adzuna.com) for free API credentials, 
since this is what I used to pull job listings.

From there, I enabled the **Google Sheets API**, **Google Drive API**, and **Gmail API** in [Google Cloud Console]
(https://console.cloud.google.com), then created OAuth 2.0 credentials and added myself as a test user so I could 
authenticate without going through Google's full verification process.

With the credentials in place, I built the workflow from scratch in n8n, node by node:

1. **Schedule Trigger** - set to run automatically twice a day, at 8 AM and 4 PM
2. **HTTP Request** - connected to the Adzuna API to fetch fresh job listings
3. **Code node** - added this to flatten the nested API response into individual job items, since the raw data came back 
bundled inside a "results" array
4. **If node** - filtered the flattened jobs to only keep roles with "Analyst" in the title
5. **Google Sheets** - logged every matching job (title, company, location, date found, and apply link) to a tracker sheet
6. **Gmail** - sent a formatted HTML email alert with clickable apply links for every new match

I connected my Google and Adzuna credentials directly in each relevant node, set up the Google Sheet with columns for Job Title,
Company, Location, Job URL, and Date Found, then published the workflow so it would run automatically on schedule.


## Lessons Learned

* API query parameter names matter. Adzuna requires lowercase keys (`app_id` not `App_ID`)
* Not all countries are supported by every job API - had to switch from `ng` to `gb` for Adzuna
* Nested JSON (`results[]`, `jobs[]`) needs a Code node to flatten before filtering works correctly
* OAuth tokens expire - reconnecting the credential resolves "invalid grant" errors
* Google Sheets nodes need the **Google Drive API** enabled too, or you get a 403 error when loading the spreadsheet list

## Status

Live and running daily (currently on localhost - migration to n8n Cloud planned for 24/7 uptime without requiring my laptop to stay on)

<img width="835" height="353" alt="image" src="https://github.com/user-attachments/assets/95831418-6cb2-4707-bb4c-3a403e817c92" />

