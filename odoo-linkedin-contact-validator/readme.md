# Odoo Contact Validator & Enricher via LinkedIn

Automatically validate existing contacts and enrich companies with key executives using LinkedIn data through Proxycurl API. Keep your CRM data fresh and complete without manual verification.

---

## Problem Solved

- **Contact Decay**: People change jobs every 2-3 years, making CRM data stale
- **Verification Burden**: Manually checking LinkedIn for each contact is time-consuming
- **Missing Executives**: Companies in Odoo often lack key decision-maker contacts
- **Outdated Information**: Job titles and responsibilities change without notification

---

## Solution Overview

Two automated workflows that integrate Odoo CRM with LinkedIn data:

### Workflow 1: Contact Validator
- ✅ Verifies if contacts still work at their listed company
- ✅ Detects job title changes
- ✅ Discovers LinkedIn URLs for contacts without them
- ✅ Flags contacts who have left the company
- ✅ Updates Odoo with verification status and timestamps

### Workflow 2: Company Enricher
- ✅ Finds key executives (CEO, CTO, CDO, CIO, CSO, VP, Directors)
- ✅ Creates new contacts in Odoo automatically
- ✅ Links executives to parent companies
- ✅ Captures LinkedIn URLs, emails, and job titles
- ✅ Ranks executives by seniority level

---

## Key Features

### Anti-Bot Protection
- **Uses official API**: Proxycurl legally accesses LinkedIn data
- **No scraping**: Avoids LinkedIn's aggressive anti-bot measures
- **Rate limiting**: 5-10 second delays between requests
- **Quota management**: Respects API limits and costs

### Intelligent Processing
- **Name matching**: 80% threshold for person identification
- **Company normalization**: Strips Inc/LLC/Ltd for accurate matching
- **Seniority ranking**: Prioritizes C-Suite over VP over Directors
- **Duplicate prevention**: Skips recently verified contacts

### Odoo Integration
- **Direct API calls**: Read/Write to res.partner model
- **Custom fields**: Tracks verification status and enrichment source
- **Batch processing**: Handles 50-100 contacts per run
- **Error handling**: Continues on individual failures

### Reporting
- **Summary reports**: Markdown-formatted run summaries
- **Google Sheets logging**: Track historical runs
- **Actionable insights**: Identifies contacts needing attention

---

## Workflows Included

```
odoo-linkedin-contact-validator/
├── odoo-contact-validator.json        # Validate existing contacts
├── odoo-company-enricher.json         # Add executives to companies
├── REFINED_REQUIREMENTS.md            # Detailed specifications
├── ODOO_SETUP_GUIDE.md               # Odoo configuration guide
├── LIMITATIONS_AND_ALTERNATIVES.md    # (if you want to add)
└── readme.md                          # This file
```

---

## Requirements

### 1. Proxycurl API Account
```
Website: https://nubela.co/proxycurl/
Cost: $10-49/month (starter plans)
Per Request: $0.10-0.50

Why Proxycurl?
- Legal LinkedIn data access
- No anti-bot issues
- Structured JSON responses
- No LinkedIn account needed
```

### 2. Odoo Instance
```
Version: 14.0+
Requirements:
- API access enabled
- Custom fields on res.partner
- Administrator privileges
```

### 3. n8n Instance
```
Version: 1.0+
Nodes: HTTP Request, Code, Wait, Merge
Credentials: HTTP Basic Auth, HTTP Header Auth
```

### 4. Google Sheets (Optional)
For logging run history and generating reports.

---

## Quick Start

### Step 1: Set Up Proxycurl

1. Sign up at https://nubela.co/proxycurl/
2. Get your API key from dashboard
3. Note your monthly credit allowance

### Step 2: Configure Odoo

1. Enable Developer Mode in Odoo
2. Create custom fields (see ODOO_SETUP_GUIDE.md):
   - `x_linkedin_url`
   - `x_last_verified`
   - `x_verification_status`
   - `x_enrichment_source`
   - `x_seniority_level`
   - `x_last_enriched`
   - `x_needs_enrichment`
3. Enable API access for a user

### Step 3: Import Workflows

1. In n8n, go to **Workflows** → **Import**
2. Import `odoo-contact-validator.json`
3. Import `odoo-company-enricher.json`

### Step 4: Configure Credentials

1. **Odoo API** (HTTP Basic Auth):
   - User: your_api_user
   - Password: your_api_key_or_password

2. **Proxycurl API** (HTTP Header Auth):
   - Header Name: Authorization
   - Header Value: Bearer YOUR_API_KEY

3. **Environment Variables**:
   - ODOO_URL: https://your-odoo-instance.com

### Step 5: Test with Small Batch

1. Modify workflow to limit to 2-3 contacts/companies
2. Run manually using "Manual Trigger"
3. Verify results in Odoo
4. Check Google Sheets for logs

### Step 6: Enable Scheduling

1. Activate the Schedule Trigger
2. Contact Validator: Weekly (Monday 9AM)
3. Company Enricher: Monthly (1st at 10AM)
4. Adjust based on your API quota and needs

---

## Cost Estimation

### Contact Validation
```
Contacts: 200/month
API Cost: $0.10/contact
Weekly runs: 50 contacts/week
Monthly Cost: 200 × $0.10 = $20
```

### Company Enrichment
```
Companies: 20/month
API Cost: $0.30-0.50/company
Monthly runs: 20 companies
Monthly Cost: 20 × $0.40 = $8
```

### Total Monthly Cost: ~$28-35

---

## Workflow Details

### Contact Validator Flow

```
Weekly Schedule (Monday 9AM)
    ↓
Fetch Contacts from Odoo
    ↓
Filter (needs verification, >30 days old)
    ↓
For Each Contact (max 50):
    ↓
    Has LinkedIn URL?
    ├─ YES → Fetch Profile via Proxycurl
    │         ↓
    │         Check Current Employment
    │         ↓
    │         Match Company Name?
    │         ├─ YES → Status: Verified
    │         └─ NO → Status: Left Company
    │
    └─ NO → Search by Name at Company
              ↓
              Match Found?
              ├─ YES → Verified + LinkedIn URL Discovered
              └─ NO → Unable to Verify
    ↓
Update Contact in Odoo
    ↓
Generate Summary Report
    ↓
Log to Google Sheet
```

### Company Enricher Flow

```
Monthly Schedule (1st at 10AM)
    ↓
Fetch Companies Without Contacts
    ↓
Filter (needs enrichment, >60 days old)
    ↓
For Each Company (max 20):
    ↓
    Search Executives via Proxycurl
        ↓
        Rank by Seniority (CEO > CTO > VP > Director)
            ↓
            Take Top 5 Executives
                ↓
                For Each Executive:
                    ↓
                    Create Contact in Odoo
                        ↓
                        Link to Parent Company
    ↓
Update Company Enrichment Status
    ↓
Generate Enrichment Report
    ↓
Log to Google Sheet
```

---

## Verification Statuses

| Status | Meaning | Action Required |
|--------|---------|-----------------|
| `verified` | Confirmed still employed | None - data is fresh |
| `left_company` | No longer at listed company | Update or remove contact |
| `unable_to_verify` | Insufficient data | Manual verification needed |
| `unverified` | Never checked | Will be verified on next run |

---

## Executive Seniority Levels

The enricher prioritizes these roles (in order):

1. **C-Suite**: CEO, CTO, CIO, CDO, CSO, CFO, COO, CMO
2. **Founders**: Founder, Co-Founder, Owner
3. **Leadership**: President, VP, Vice President
4. **Management**: Director, Head of, Manager

Each company receives up to 5 top executives.

---

## Customization Options

### Change Verification Frequency
In `odoo-contact-validator.json`:
```javascript
// Weekly trigger
"triggerAtDay": [1], // 1 = Monday
"triggerAtHour": 9   // 9 AM
```

### Adjust Batch Size
```javascript
const DAILY_LIMIT = 50;  // Change to 100 for larger batches
const batch = contactsToVerify.slice(0, DAILY_LIMIT);
```

### Modify Company Name Matching
```javascript
// Current: partial match
if (linkedinCompany.includes(companyNameNormalized) ||
    companyNameNormalized.includes(linkedinCompany)) {

// Stricter: exact match only
if (linkedinCompany === companyNameNormalized) {
```

### Target Different Roles
```javascript
const priorityRoles = [
  'ceo', 'chief executive',
  'cto', 'chief technology',
  // Add or remove roles as needed
  'sales manager',
  'marketing director'
];
```

---

## Error Handling

### API Errors
- Workflows continue on individual failures
- Errors logged with timestamps
- Summary report includes error counts

### Rate Limiting
- 5-second delay for contact validation
- 10-second delay for company enrichment
- Automatic quota tracking

### Missing Data
- Contacts without LinkedIn: searched by name
- Companies without results: flagged for manual research
- Incomplete profiles: partial data captured

---

## Monitoring & Reporting

### Google Sheets Log (Contact Validation)
```
| run_date | total_processed | verified | left_company | unable_to_verify |
|----------|-----------------|----------|--------------|------------------|
| 2025-01-15 | 50 | 42 | 5 | 3 |
```

### Google Sheets Log (Company Enrichment)
```
| run_date | companies_processed | companies_enriched | contacts_created |
|----------|---------------------|-------------------|------------------|
| 2025-01-01 | 20 | 18 | 87 |
```

### Summary Reports
Markdown-formatted reports include:
- Total processed
- Success/failure breakdown
- Specific actions required
- Cost estimation

---

## Privacy & Compliance

### Data Protection
- ✅ Uses official API (legal access)
- ✅ Stores only public LinkedIn data
- ✅ Respects API terms of service
- ✅ No credential storage for LinkedIn accounts

### GDPR/CCPA Considerations
- Document data processing activities
- Obtain consent for contact storage (if required)
- Provide data deletion capabilities
- Regular data audits

### API Key Security
- Store keys in n8n credentials (encrypted)
- Rotate keys periodically
- Monitor for unusual usage
- Set up billing alerts

---

## Limitations

### What This System Can Do
✅ Verify employment via LinkedIn profiles
✅ Find executives by company name
✅ Update CRM data automatically
✅ Generate verification reports
✅ Respect API rate limits

### What This System Cannot Do
❌ Access private LinkedIn profiles
❌ Bypass LinkedIn authentication
❌ Verify contacts without any identifiers
❌ Guarantee 100% accuracy (false positives possible)
❌ Process thousands of contacts daily (API costs)

---

## Troubleshooting

### "API returned 403"
**Cause**: Invalid API key or exceeded quota
**Solution**: Check Proxycurl dashboard for key validity and remaining credits

### "Odoo connection failed"
**Cause**: Invalid credentials or URL
**Solution**: Test API connection with curl command (see ODOO_SETUP_GUIDE.md)

### "Field does not exist"
**Cause**: Custom fields not created in Odoo
**Solution**: Follow ODOO_SETUP_GUIDE.md to create required fields

### "No contacts found"
**Cause**: Filter criteria too strict
**Solution**: Adjust Odoo domain filter in workflow

### "Rate limit exceeded"
**Cause**: Too many requests too quickly
**Solution**: Increase delay between requests (5s → 10s)

---

## Future Enhancements

1. **Email Notifications**: Send alerts when contacts leave companies
2. **Slack Integration**: Post daily verification summaries
3. **AI Classification**: Use LLM to categorize verification results
4. **Duplicate Detection**: Check for existing contacts before creating
5. **Historical Tracking**: Track job changes over time
6. **Company Matching**: Improve fuzzy matching for company names

---

## Resources

- **Proxycurl Documentation**: https://nubela.co/proxycurl/docs/
- **Odoo API Reference**: https://www.odoo.com/documentation/
- **n8n Community**: https://community.n8n.io/
- **LinkedIn Data Usage**: https://www.linkedin.com/legal/professional-community-policies

---

## License

This workflow is provided for educational and business use. Ensure compliance with:
- LinkedIn Terms of Service
- Proxycurl Terms of Service
- Local data protection laws (GDPR, CCPA)
- Your organization's data policies

---

## Support

For issues or questions:
1. Check the troubleshooting section above
2. Review the ODOO_SETUP_GUIDE.md for configuration
3. Visit n8n Community forums
4. Contact Proxycurl support for API issues
