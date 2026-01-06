# Automated Job Scraper for Executive/Admin Assistant Positions

## Automation Scheduling

This workflow uses n8n's built-in **Schedule Trigger** node:

### Current Configuration
- **Frequency:** Daily
- **Time:** 9:00 AM EST
- **Timezone:** America/New_York

### How to Modify Schedule
1. Open the workflow in n8n
2. Click on "Daily Job Scraper Schedule" node
3. Adjust the trigger settings:
   - Change to Weekly/Hourly/Custom intervals
   - Modify time of day
   - Update timezone if needed
4. Save the workflow
<img width="1475" height="873" alt="image" src="https://github.com/user-attachments/assets/94009811-4896-46a6-9c57-34b721ebee80" />


### Alternative: Manual Execution
- Click "Execute Workflow" button in n8n to run on-demand
- Useful for testing or immediate job searches

---

# User Guide

## Overview
This n8n workflow automates job searching by scraping **RemoteOK**, validating positions with AI, generating personalized outreach messages, and saving results to Google Sheets.

## Prerequisites
- n8n instance (cloud or self-hosted)
- OpenAI API account with credits
- Google account with Sheets access
- RemoteOK API access (free, no key required)

## Installation Instructions

### 1. Set Up n8n
**Option A: n8n Cloud (Recommended for beginners)**
- Sign up at [n8n.io](https://n8n.io)
- Create a new workflow

**Option B: Self-Hosted**
```bash
npm install n8n -g
n8n start
```

### 2. Import the Workflow
1. Download `job-scraper-workflow.json`
2. In n8n, click **"Import from File"**
3. Select the downloaded JSON file
4. The workflow will appear on your canvas

### 3. Configure Credentials
**OpenAI API:**
1. Get API key from [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
2. In n8n: **Settings** → **Credentials** → **Add Credential** → **OpenAI**
3. Paste your API key
4. Connect to both AI nodes:
   - "Validate Job Criteria"
   - "Generate Outreach Message"

**Google Sheets:**
1. In n8n: **Settings** → **Credentials** → **Add Credential** → **Google Sheets OAuth2**
2. Follow OAuth flow to authorize n8n
3. Connect to "Save to Google Sheets" node

### 4. Set Up Your Google Sheet
1. Create a new Google Sheet
2. Add column headers:
   - Job Posting URL
   - Job Title
   - Company Name
   - Location
   - Job Description
   - Outreach Message
   - Date Added
3. Copy the Sheet ID from URL (between `/d/` and `/edit`)
4. In n8n, open **"Save to Google Sheets"** node
5. Replace `YOUR_GOOGLE_SHEET_ID_HERE` with your actual ID

### 5. Test the Workflow
1. Click **"Execute Workflow"** button
2. Monitor each node's execution
3. Check your Google Sheet for results
4. Verify outreach messages are clean and personalized

---

## Running Instructions

### Automated (Scheduled)
- Workflow runs automatically daily at 9 AM EST
- No action needed once configured
- Check Google Sheet each morning for new jobs

### Manual Execution
- Click **"Execute Workflow"** in n8n
- Useful for immediate job searches
- Can run multiple times per day

---

## Customization Guide

### Update Search Parameters
**Change Job Board or Search Terms:**
Open **"Fetch Job Board Data"** node and modify URL to target different RemoteOK tags:
- `https://remoteok.com/api?tag=admin`
- `https://remoteok.com/api?tag=non-tech`
- `https://remoteok.com/api` (all jobs)

**Adjust Job Limit:**
Open **"Workflow Configuration"** node and change `maxJobs` value (default: 50).

### Update LLM Prompts
**Validation Criteria:**
Open **"Validate Job Criteria"** node. Edit the System Message to adjust filtering logic (e.g., make relevance more/less strict).

**Outreach Message Style:**
Open **"Generate Outreach Message"** node. Modify the System Message to change:
- Tone (professional, casual, enthusiastic)
- Length (currently 150-200 words)
- Content focus (skills, experience, enthusiasm)

**Example Prompt Variations:**
> **Professional & Concise:**
> "Generate a brief, professional outreach message (100-150 words) highlighting relevant EA/AA experience and expressing interest."

> **Enthusiastic & Detailed:**
> "Create an engaging outreach message (200-250 words) that shows genuine excitement about the role and company, with specific examples of how your skills match their needs."

### Modify Schedule
Open **"Daily Job Scraper Schedule"** node and change frequency:
- Daily at specific time
- Weekly on specific day
- Hourly intervals
- Custom cron expression
- *Adjust timezone if needed*

---

## Troubleshooting

### Common Issues
**Issue: No jobs appearing in Google Sheet**
- Check OpenAI API credits/quota
- Verify Google Sheets credentials are connected
- Ensure Sheet ID is correct
- Check if any jobs passed validation filters

**Issue: Workflow times out**
- Reduce `maxJobs` in configuration (try 20-30)
- Check internet connection
- Verify RemoteOK API is accessible

**Issue: HTML tags in job descriptions**
- Verify **"Clean HTML from Description"** node is active
- Check node configuration for proper regex patterns

**Issue: Duplicate jobs in sheet**
- Duplicate prevention is automatic based on Job URL
- If duplicates appear, check **"Check for Duplicates"** node logic

**Issue: Outreach messages contain tracking codes**
- Verify **"Generate Outreach Message"** node has post-processing
- Check that STIMULATE code removal is configured

**Issue: AI validation too strict/lenient**
- Adjust **"Validate Job Criteria"** prompt
- Make relevance criteria more inclusive or exclusive
- Test with manual execution to see validation reasoning

### RemoteOK API Changes
If RemoteOK changes their API structure:
1. Test API response: `https://remoteok.com/api`
2. Update field mappings in:
   - "Extract Job Description" node
   - "Structure Job Data" node
3. Verify data flows correctly through validation

### Rate Limiting
If you hit API rate limits:
- Add delay between requests (Wait node)
- Reduce `maxJobs` parameter
- Spread executions throughout the day

---

## How to Use the Google Sheet

### Daily Workflow for Sales Rep
**1. Review New Jobs (added since last run)**
- Check Date Added column for today's date
- Review job titles and companies

**2. Read AI-Generated Outreach**
- Each job has a personalized message ready
- Customize if needed (add personal touches)

**3. Apply to Jobs**
- Click Job Posting URL
- Copy outreach message
- Submit application on job board

**4. Track Applications**
- Add columns: "Applied" (checkbox), "Response Date", "Status"
- Mark jobs as you apply
- Track follow-ups

### Sheet Organization Tips
- Filter by Date to see newest jobs first
- Highlight promising roles with color coding
- Add notes column for interview prep or follow-up reminders
- Archive old jobs to a separate sheet after 30 days

---

## Architecture Overview
```mermaid
graph TD
    A[Schedule Trigger (Daily 9 AM)] --> B[Fetch Jobs from RemoteOK API]
    B --> C[Filter Out Metadata]
    C --> D[Extract Job Details]
    D --> E[AI Validation]
    E --> F[Filter Valid Jobs]
    F --> G[Clean HTML from Descriptions]
    G --> H[Generate Personalized Outreach Messages (AI)]
    H --> I[Check for Duplicates]
    I --> J[Save to Google Sheets]
```

**AI Models Used:**
- **GPT-4o-mini:** Job validation (fast, cost-effective)
- **GPT-4o:** Outreach generation (high quality)

**Data Sources:**
- **RemoteOK API** (free, no authentication required)

**Output:**
- Google Sheets with 7 columns
- Automatic duplicate prevention
- Clean, HTML-free descriptions

### Cost Estimates
**OpenAI API Usage (per run):**
- Validation: ~$0.01-0.05 (50 jobs × GPT-4o-mini)
- Outreach: ~$0.10-0.30 (10-20 valid jobs × GPT-4o)
- **Total per day:** ~$0.15-0.35
- **Monthly estimate:** ~$5-10 (daily runs)

**n8n Costs:**
- **Cloud:** Free tier available (5,000 executions/month)
- **Self-hosted:** Free (hosting costs vary)

---

## Support & Maintenance

**Regular Maintenance:**
- Monitor Google Sheet for quality
- Review AI validation reasoning periodically
- Update prompts if job market changes
- Check OpenAI API usage/costs monthly

**Updates:**
- Export workflow regularly as backup
- Document any customizations made
- Test after n8n platform updates

**License & Credits**
- **Built with:** n8n, OpenAI GPT-4o & GPT-4o-mini, RemoteOK API, Google Sheets API

**Questions or Issues?**
- Contact: e.yegonbett@gmail.com

---

## Google Sheets Integration

**Provide:**
1. Link to your populated Google Sheet (with view access)
2. Screenshot showing all columns filled with sample data
3. Column documentation:

### Google Sheets Column Reference

| Column | Description | Example |
| :--- | :--- | :--- |
| **Job Posting URL** | Direct link to apply | `https://remoteok.com/remote-jobs/123456` |
| **Job Title** | Position name | Executive Assistant |
| **Company Name** | Hiring company | Destination Knot |
| **Location** | Job location | Remote (Worldwide) |
| **Job Description** | Clean text description | Assist clients with booking hotels... |
| **Outreach Message** | AI-generated personalized message | I was excited to see your Executive Assistant opening... |
| **Date Added** | When job was scraped | 2025-01-05 09:00:00 |
