# Google Sheet URL Event Scraper

## Overview

This n8n workflow automates web scraping of URLs from a Google Sheet and intelligently extracts event-related information based on your specified topics. Perfect for monitoring multiple websites for events, conferences, webinars, and gatherings related to your areas of interest.

## Key Features

- **100% Free Scraping** - Uses n8n's built-in HTTP Request node (no paid services required)
- **Intelligent Event Detection** - Automatically identifies events using keyword matching and context analysis
- **Topic-Based Filtering** - Only extracts events relevant to your specified topics
- **Confidence Scoring** - Rates each extracted event as "high" or "medium" confidence
- **Automated HTML Processing** - Cleans and extracts text from HTML content
- **Google Sheets Integration** - Easy setup with read/write to Google Sheets
- **Batch Processing** - Processes multiple URLs in a single run
- **Error Handling** - Gracefully handles failed requests and missing data

## Use Cases

- Monitor industry websites for relevant conferences and events
- Track competitor events and webinars
- Aggregate events from multiple sources
- Research industry gatherings related to specific topics
- Build an automated event discovery system
- Create curated event lists for newsletters or reports

## Workflow Structure

```
Manual Trigger
    ↓
Read URLs (Tab 1) + Read Topics (Tab 2)
    ↓
Merge Data
    ↓
Split URLs (Loop through each)
    ↓
Scrape URL Content (HTTP Request - FREE)
    ↓
Extract Events from Content
    ↓
Filter Events (Has Events?)
    ↓
Split Events
    ↓
Format Event Data
    ↓
Write to Results Sheet (Tab 3)
```

## Google Sheet Setup

### Required Structure

Create a Google Sheet with **3 tabs**:

#### Tab 1: URLs
| Column | Description | Required | Example |
|--------|-------------|----------|---------|
| url | Full URL to scrape | Yes | https://techcrunch.com/events/ |
| name | Source name/description | No | TechCrunch Events |

**Example:**
```
url                                    | name
https://techcrunch.com/events/        | TechCrunch Events
https://www.eventbrite.com/d/online/  | Eventbrite Online
https://www.meetup.com/find/          | Meetup Events
```

#### Tab 2: Topics
| Column | Description | Required | Example |
|--------|-------------|----------|---------|
| topic | Topic keyword to search for | Yes | artificial intelligence |

**Example:**
```
topic
artificial intelligence
machine learning
blockchain
web3
startup
technology conference
```

#### Tab 3: Results (Auto-populated)
This tab will be automatically filled with results:

| Column | Description |
|--------|-------------|
| source_url | Original URL that was scraped |
| source_name | Name of the source |
| event_description | Extracted event text/description |
| related_topics | Topics that matched in this event |
| confidence | Confidence level (high/medium) |
| scraped_at | Timestamp when data was scraped |

## Installation & Setup

### Step 1: Import Workflow
1. Open n8n
2. Go to **Workflows** → **Import**
3. Upload the `gsheet-url-event-scraper.json` file
4. The workflow will be created with all nodes connected

### Step 2: Create Google Sheet
1. Create a new Google Sheet
2. Create 3 tabs named: **URLs**, **Topics**, **Results**
3. Add column headers as shown above
4. Fill in your URLs in Tab 1
5. Fill in your topics in Tab 2
6. Leave Tab 3 empty (will be auto-populated)

### Step 3: Connect Google Sheets
You'll need to configure 3 Google Sheets nodes:

1. **Read URLs from Sheet**
   - Click the node
   - Add Google Sheets credentials (if not already connected)
   - Select your Google Sheet
   - Select the "URLs" tab
   - Save

2. **Read Topics from Sheet**
   - Click the node
   - Select the same Google Sheet
   - Select the "Topics" tab
   - Save

3. **Write to Results Sheet**
   - Click the node
   - Select the same Google Sheet
   - Select the "Results" tab
   - Save

### Step 4: Test the Workflow
1. Click **Execute Workflow** button
2. Check the "Results" tab in your Google Sheet
3. Review extracted events

## How It Works

### 1. Data Collection
The workflow reads all URLs from your Google Sheet (Tab 1) and all topics from Tab 2.

### 2. URL Scraping
For each URL:
- Sends a free HTTP GET request
- Retrieves the HTML content
- Handles redirects (up to 5)
- Times out after 30 seconds

### 3. Content Extraction
The workflow:
- Strips HTML tags and scripts
- Decodes HTML entities
- Cleans whitespace
- Extracts plain text content

### 4. Event Detection
Searches for sentences containing:

**Event Keywords:**
- event, conference, meeting, workshop, seminar, webinar
- summit, forum, symposium, exhibition, fair, festival
- ceremony, celebration, gathering, session, presentation
- Date/time indicators (months, days, years)
- Location indicators

**AND** one or more of your specified topics

### 5. Confidence Scoring
- **High**: Event matches 2+ topics
- **Medium**: Event matches 1 topic

### 6. Results Storage
Extracted events are written to the Results tab with:
- Source information
- Event description
- Related topics
- Confidence score
- Timestamp

## Customization Options

### Modify Event Keywords
Edit the `Extract Events from Content` node to customize event detection:

```javascript
const eventKeywords = [
  'event', 'conference', 'meeting', // Add your keywords here
  // ... more keywords
];
```

### Adjust Timeout
Change request timeout in the `Scrape URL Content` node:
```json
"timeout": 30000  // milliseconds (30 seconds)
```

### Change Confidence Logic
Modify confidence scoring in the `Extract Events from Content` node:
```javascript
confidence: relatedTopics.length > 2 ? 'high' : 'medium'  // Require 3+ topics for high
```

### Add More Data Points
Extend the `Format Event Data` node to capture additional information:
- Add date extraction
- Extract location information
- Capture contact details
- Add custom fields

## Troubleshooting

### No Events Found
**Possible Causes:**
- Topics are too specific
- URLs don't contain event-related content
- Event keywords don't match the content

**Solutions:**
- Add more general topics
- Review the event keywords list
- Check URLs manually to verify they contain events

### Scraping Fails
**Possible Causes:**
- URL requires authentication
- Website blocks automated requests
- Timeout is too short
- Website is down

**Solutions:**
- Increase timeout value
- Add delay between requests
- Check URL accessibility in a browser
- Some websites may block scraping (use alternative sources)

### Duplicate Results
**Possible Causes:**
- Running workflow multiple times
- Same content appears on multiple URLs

**Solutions:**
- Clear Results tab before re-running
- Add deduplication logic in a Code node
- Use "Append or Update" mode with matching columns

### Google Sheets Connection Issues
**Solutions:**
- Re-authenticate Google Sheets credentials
- Check sheet permissions (must have edit access)
- Verify tab names match exactly
- Ensure sheet ID is correct

## Performance Considerations

### Processing Speed
- Each URL takes ~2-5 seconds to scrape
- 10 URLs = ~30-60 seconds total
- 100 URLs = ~5-10 minutes total

### Optimization Tips
1. **Batch Processing**: Process URLs in smaller batches
2. **Scheduling**: Use a Schedule trigger for regular updates
3. **Caching**: Store previously scraped URLs to avoid duplicates
4. **Error Handling**: Add error handling nodes for failed requests

## Advanced Enhancements

### Add Error Handling
Insert an error trigger node to catch and log failed scrapes:
```
Scrape URL Content → [Error Trigger] → Log to Sheet
```

### Add Scheduling
Replace Manual Trigger with Schedule Trigger:
- Daily: `0 0 * * *`
- Weekly: `0 0 * * 0`
- Hourly: `0 * * * *`

### Add Deduplication
Before writing results, add a node to check for duplicates:
1. Read existing results
2. Compare event descriptions
3. Only write new events

### Add Notifications
Send alerts when new events are found:
- Email notification
- Slack message
- Discord webhook
- SMS via Twilio

### Enhanced Event Extraction
Use AI/LLM for better extraction:
1. Add OpenAI/Anthropic node
2. Send content to LLM
3. Ask for structured event extraction
4. Parse JSON response

**Example Prompt:**
```
Extract all events from this text. For each event, provide:
- Event name
- Date
- Location
- Description
- URL
Return as JSON array.
```

## Example Results

After running the workflow, your Results tab might look like:

| source_url | source_name | event_description | related_topics | confidence | scraped_at |
|------------|-------------|-------------------|----------------|------------|------------|
| https://techcrunch.com/events/ | TechCrunch | Annual AI Summit on March 15, 2025 featuring the latest in artificial intelligence and machine learning | artificial intelligence, machine learning | high | 2025-01-15T10:30:00Z |
| https://eventbrite.com/ | Eventbrite | Blockchain Developer Workshop - Learn Web3 development from industry experts | blockchain, web3 | high | 2025-01-15T10:31:00Z |
| https://meetup.com/ | Meetup | Technology Conference 2025 - Join us for a day of innovation | technology conference | medium | 2025-01-15T10:32:00Z |

## Limitations

### Website Restrictions
- Some websites block automated scraping
- JavaScript-heavy sites may not render properly
- Dynamic content may not be captured
- Rate limiting may occur with frequent requests

### Event Detection Accuracy
- Keyword-based detection may have false positives
- Complex event formats may be missed
- Non-English content may not be detected
- Context may be lost in extraction

### Solutions:
- Use multiple source URLs
- Combine with AI/LLM for better accuracy
- Manual review of medium confidence results
- Adjust keywords based on your domain

## Best Practices

1. **Start Small**: Test with 2-3 URLs first
2. **Review Results**: Check accuracy before scaling up
3. **Update Topics**: Refine topics based on results
4. **Regular Updates**: Schedule regular runs for fresh data
5. **Clean Data**: Periodically clear old results
6. **Monitor Performance**: Track execution times and errors
7. **Backup Data**: Export Google Sheet regularly

## Privacy & Legal

**Important Considerations:**
- ✅ Only scrape publicly accessible content
- ✅ Respect robots.txt files
- ✅ Don't overload servers with requests
- ✅ Review website terms of service
- ❌ Don't scrape private/authenticated content
- ❌ Don't use scraped data for commercial purposes without permission

## Support & Resources

### n8n Documentation
- [Google Sheets Node](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.googlesheets/)
- [HTTP Request Node](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.httprequest/)
- [Code Node](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.code/)

### Community
- [n8n Community Forum](https://community.n8n.io/)
- [n8n Discord](https://discord.gg/n8n)

## Version History

- **v1.0** (2025-01-15)
  - Initial release
  - Basic event detection
  - Google Sheets integration
  - Free HTTP scraping

## License

This workflow is provided as-is for educational and personal use.

## Credits

Created for the n8n community. Feel free to modify and share!

---

**Need Help?** Open an issue or reach out to the n8n community for support.
