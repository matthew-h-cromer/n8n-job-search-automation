# n8n Job Search Automation

An n8n workflow that searches LinkedIn daily, evaluates every new posting against your resume and career goals using Claude, scores each one on fit and likelihood of success, and delivers a prioritized report to your inbox every morning.

## How It Works

Every morning, the workflow:

1. **Searches LinkedIn** using your targeted search queries across multiple pages of results
2. **Deduplicates** results across searches and filters out any jobs you've already seen
3. **Reads the full job description** for each new posting
4. **Evaluates every job with Claude**, scoring it on two dimensions:
   - **Fit** (1-10): How well does this role align with your background and career interests?
   - **Chance** (1-10): Realistically, what are your odds of getting an interview?
   - Plus a verdict (Apply / Skip), reasoning, strengths, and gaps
5. **Saves everything to a Google Sheet** as a running database you can track over time
6. **Emails you a prioritized daily digest** sorted by the roles most worth pursuing

## Setup

### 1. Set up n8n
Sign up for [n8n Cloud](https://n8n.io). The free tier is generous and more than enough to get started. Alternatively, you can [self-host n8n](https://docs.n8n.io/hosting/).

### 2. Import the workflow
In n8n, go to **Workflows > Import from File** and select `n8n-job-search-automation.json`.

### 3. Add your credentials
The workflow needs API access to three services. n8n will prompt you to connect each one the first time you activate the workflow:

- **Anthropic** - for Claude evaluations. Get an API key at [console.anthropic.com](https://console.anthropic.com/)
- **Google Sheets** (OAuth2) - for storing results and deduplication
- **Gmail** (OAuth2) - for the daily digest email

### 4. Create your tracking spreadsheet
Set up a Google Sheet with the following columns (these are the columns the workflow writes to):

| Column | Description |
|---|---|
| Company | Company name |
| Title | Job title |
| Short Description | AI-generated summary of the role |
| Status | "Will Apply" or "Will Not Apply" |
| Salary Min ($k) | Minimum salary |
| Salary Max ($k) | Maximum salary |
| Link | URL to the posting |
| Notes | AI assessment: fit score, chance score, and reasoning |
| Full Description | Complete job description text |

You can add any additional columns you find useful for tracking (e.g. Date Applied, Interviewed, etc.) — the workflow won't overwrite them.

Then update the Google Sheets nodes in the workflow to point to your spreadsheet.

### 5. Customize the search queries
Open the **Code in JavaScript** node at the start of the workflow and replace the search terms with your own:

```javascript
const SEARCH_TERMS = [
  `"product manager" developer experience`,
  `"product manager" robotics`,
  `"product manager" manufacturing software`,
  `"product manager" marketplace`,
  `"product manager" ai`
]
```

Think about what keywords define your target roles. The `"quoted terms"` ensure LinkedIn treats them as exact phrases.

### 6. Write your own prompt
This is the most important step. Open the **Basic LLM Chain** node and replace the placeholder resume and career narratives with yours.

The prompt has two sections you need to customize:

**Resume**: Paste your full resume text.

**Career Narratives**: Write 3-6 short paragraphs describing the types of roles you're targeting and why your background makes you a fit. These give the LLM context beyond your resume. Example:

> **Developer Platforms & APIs**
> I built a B2B developer platform end-to-end: API, SDK, documentation, developer portal. Strong fit for developer tools, API products, SDK teams, and developer experience roles.

Spend time on this. The quality of the output depends entirely on how well you describe what you're looking for and why you're a fit.

### 7. Update the email address
Search for `your.email@example.com` in the workflow and replace it with your actual email address in both Gmail nodes (the daily report and the error notification).

### 8. Update the Google Sheet link in the email
In the **Send a message1** Gmail node, replace `YOUR_GOOGLE_SHEET_ID` in the email body with your actual Google Sheet ID so the daily digest links to your spreadsheet.

### 9. Test before scheduling
Run the workflow manually a few times using the **When clicking 'Execute workflow'** trigger to verify results look right. Then set the **Schedule Trigger** to whatever cadence makes sense (default is daily at 8 AM).

## License

MIT
