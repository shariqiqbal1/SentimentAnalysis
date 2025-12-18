# Voice of Customer Sentiment Agent (n8n + Gemini)

This repo contains an n8n workflow that turns Reddit and Twitter / X chatter into feature-level Voice of Customer reports.

You give it a simple list in Google Sheet with `Product` and `Feature/Topic`.  
It pulls public conversations (Reddit and X), runs them through Tavily and Gemini, and writes back a detailed summary with:

1. The emotional vibe  
2. The real recurring problems  
3. Short verbatim quotes (voice of customer)  
4. Clear recommendation for the PM

Example rows:

- Lyft · "Women+ Connect" safety feature sentiment  
- DoorDash · DashPass vs Uber One value for money  
- Waymo · Robotaxi pickup accuracy vs Human Drivers  
- Instacart · Grocery Substitution Logic complaints  
- Bolt · Driver Quality standards vs Cheap Pricing  
- Uber · Uber One Cancellation Process difficulty  

---

## Screenshots

Place your screenshots in a `screenshots` folder and update paths if needed.

### n8n workflow

<img width="806" height="415" alt="image" src="https://github.com/user-attachments/assets/a13d98b4-4f1a-460d-b350-e4e4666bf126" />


### Sample report in Google Sheets

<img width="1217" height="722" alt="image" src="https://github.com/user-attachments/assets/525adefd-7fbc-4235-919b-2eebf02869e1" />


---

## How it works

At a high level:

1. **Read config from Google Sheets**  
   - Each row defines a `Product` and `Feature/Topic`.

2. **Loop over each row**

3. **Tavily search**  
   - Builds a query like:  
     `"<Product> <Feature/Topic> (site:reddit.com OR site:twitter.com OR site:x.com) -site:promoted"`  
   - Fetches recent discussions from Reddit and Twitter / X.

4. **Gemini analysis**  
   - Sends the raw search results into Gemini 2.5 Flash Lite.  
   - Prompt forces a structured output:
     - Vibe check  
     - Deep problems  
     - Direct quotes  
     - One strategic recommendation

5. **Write back to Google Sheets**  
   - Updates the same row with:
     - `Date Run`  
     - `AI Report` (the VOC one-pager)

You end up with a sheet that behaves like a small research OS.

---

## Tech stack

- **Orchestration:** n8n
- **Data store:** Google Sheets
- **Search:** Tavily API (Reddit + Twitter / X focused queries)
- **Model:** Google Gemini 2.5 Flash Lite (via n8n’s Gemini node)
- **Format:** `Sentiment Analysis.json` n8n workflow included in this repo

---

## Requirements

- n8n (self hosted or n8n cloud)
- Google account with access to Google Sheets
- Tavily API key
- Google Gemini / PaLM API key
- Basic familiarity with n8n (importing workflows, creating credentials)

---

## Google Sheet setup

Create a sheet with at least these columns:

| Product | Feature/Topic | Date Run | AI Report |
|--------|---------------|----------|-----------|

Example:

| Product  | Feature/Topic                                       | Date Run   | AI Report |
|----------|-----------------------------------------------------|------------|----------|
| Lyft     | "Women+ Connect" safety feature sentiment           |            |          |
| DoorDash | DashPass vs Uber One value for money               |            |          |
| Waymo    | Robotaxi pickup accuracy vs Human Drivers           |            |          |
| Instacart| Grocery Substitution Logic complaints               |            |          |
| Bolt     | Driver Quality standards vs Cheap Pricing           |            |          |
| Uber     | Uber One Cancellation Process difficulty            |            |          |

Leave `Date Run` and `AI Report` empty. The workflow will fill those in.

---

## n8n setup

### 1. Import the workflow

1. Open n8n.
2. Create a new workflow.
3. Use **Import from file**.
4. Select `Sentiment Analysis.json` from this repo.

You should see a workflow with nodes similar to:

- When clicking `Execute workflow`
- Get row(s) in sheet
- Loop Over Items
- Tavily
- Message a model
- Update row in sheet

### 2. Configure credentials

Create the following credentials in n8n:

1. **Google Sheets OAuth2**
   - Go to **Credentials** → **Google Sheets OAuth2 API**.
   - Authorize with the Google account that owns your sheet.

2. **Gemini / Google AI**
   - Use the appropriate n8n credential type (for Google PaLM / Gemini).
   - Paste your API key from Google AI Studio.

3. **Tavily**
   - Either:
     - Store the API key as a generic credential and reference it in the HTTP Request node, or  
     - Keep it in the node body if you are just testing (not recommended for serious use).

Make sure the workflow’s nodes are wired to use these credentials.

### 3. Point the workflow at your sheet

In the **Get row(s) in sheet** and **Update row in sheet** nodes:

1. Select your Google Sheets credential.
2. Set the **Spreadsheet** to the sheet you created.
3. Set the **Sheet** to the correct tab (Sheet1 or whatever you named it).

Check that:

- The read node is pulling `Product` and `Feature/Topic`.
- The update node maps:
  - `row_number` (from the read node)
  - `Date Run` → current date expression
  - `AI Report` → Gemini output (`content.parts[0].text`)

If you changed column names, update the mappings accordingly.

---

## How to run

1. Fill the Google Sheet with rows of `Product` + `Feature/Topic`.
2. In n8n, open this workflow.
3. Click **Execute workflow**.
4. Let it run through all rows.

As it runs, you should see, per row:

- Tavily node returns search results.
- Gemini node returns a structured text report.
- Update node writes `Date Run` and `AI Report` back into the sheet.

Refresh your sheet and you will see a full VOC report per row.

---

## Customization

Some ideas if you want to tweak it:

- **Change data sources**  
  Edit the Tavily query to:
  - Add or remove sites  
  - Focus on specific subreddits  
  - Add language filters

- **Adjust model behavior**  
  Modify the Gemini prompt to:
  - Add a numeric sentiment score  
  - Output JSON instead of markdown  
  - Include “Opportunities” or “Risks” sections

- **Scheduling**  
  Replace the manual trigger with:
  - A Cron node (daily, weekly)  
  - A Webhook node to trigger via API or Slack command

- **Aggregation**  
  Add a second workflow to:
  - Combine multiple feature reports into a single product-level summary  
  - Rank features by negativity or frequency of pain points

---

## Limitations

- This relies on public social chatter. If your feature is niche or new, results may be thin.
- Not a replacement for real user interviews or proper UX research.
- Quality depends heavily on the search query and amount of noise on Reddit / Twitter.

Use it as a fast VOC radar, not as the only source of truth.

---

## Created by: Shariq Iqbal linkedin.com/in/shariqi
