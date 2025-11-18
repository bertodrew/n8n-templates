# Google Sheet Template

Create a Google Sheet with the following structure:

## Tab 1: URLs

| url | name |
|-----|------|
| https://techcrunch.com/events/ | TechCrunch Events |
| https://www.eventbrite.com/d/online/technology--events/ | Eventbrite Tech |
| https://www.meetup.com/find/?keywords=technology | Meetup Tech Events |
| https://www.conference-board.org/events | Conference Board |
| https://events.linuxfoundation.org/ | Linux Foundation Events |

## Tab 2: Topics

| topic |
|-------|
| artificial intelligence |
| machine learning |
| blockchain |
| web3 |
| cybersecurity |
| cloud computing |
| data science |
| software development |
| startup |
| technology conference |
| innovation |
| digital transformation |

## Tab 3: Results

Leave this tab empty. The workflow will automatically populate it with these columns:

| source_url | source_name | event_description | related_topics | confidence | scraped_at |
|------------|-------------|-------------------|----------------|------------|------------|
| (auto-filled) | (auto-filled) | (auto-filled) | (auto-filled) | (auto-filled) | (auto-filled) |

## How to Use

1. **Copy the structure above** into a new Google Sheet
2. **Fill Tab 1** with URLs you want to scrape
3. **Fill Tab 2** with topics you're interested in
4. **Leave Tab 3** empty - it will be filled automatically
5. **Run the n8n workflow** and check Tab 3 for results

## Tips

- **URLs**: Add any publicly accessible websites that list events
- **Topics**: Be specific enough to filter relevant events, but not too narrow
- **Multiple topics per event**: Events matching multiple topics get a "high" confidence score
- **Update regularly**: Add new URLs and topics as needed

## Example Results

After running the workflow, Tab 3 might contain:

| source_url | source_name | event_description | related_topics | confidence | scraped_at |
|------------|-------------|-------------------|----------------|------------|------------|
| https://techcrunch.com/events/ | TechCrunch Events | Join us for the AI Summit 2025, featuring leaders in artificial intelligence and machine learning on March 15th | artificial intelligence, machine learning | high | 2025-01-15T10:30:00.000Z |
| https://eventbrite.com/ | Eventbrite Tech | Blockchain and Web3 Developer Workshop - April 20, 2025 at San Francisco Convention Center | blockchain, web3 | high | 2025-01-15T10:31:00.000Z |
| https://meetup.com/ | Meetup Tech Events | Monthly Technology Meetup - Join fellow innovators for networking and presentations | technology conference, innovation | high | 2025-01-15T10:32:00.000Z |
