# Odoo Contact Validator & Enrichment System

## Executive Summary

Automatically validate and enrich Odoo contacts by verifying employment status through LinkedIn data, and discover key executives for companies missing contact information.

---

## Business Problem

1. **Contact Decay**: People change jobs every 2-3 years on average
2. **Stale CRM Data**: Contacts in Odoo may no longer work at listed companies
3. **Missing Relationships**: Companies in Odoo lack key executive contacts
4. **Manual Verification**: Time-consuming to manually check LinkedIn for each contact

---

## Solution Overview

### Two Main Workflows:

**Workflow 1: Contact Validation**
- Read contacts from Odoo
- Check if they still work at the listed company via LinkedIn
- Update Odoo with verification status
- Flag contacts that have left the company

**Workflow 2: Contact Enrichment**
- Read companies from Odoo that lack contacts
- Find key executives on LinkedIn (CEO, CTO, CDO, CIO, CSO, Board Members)
- Create new contacts in Odoo with enriched data

---

## CRITICAL: LinkedIn Anti-Bot Challenges

### Why Direct LinkedIn Scraping Won't Work

1. **Aggressive Protection**
   - Cloudflare Enterprise
   - Browser fingerprinting
   - Behavioral analysis
   - IP reputation scoring
   - CAPTCHA challenges
   - Account suspension for automation

2. **Legal Issues**
   - LinkedIn actively sues scrapers (hiQ Labs v. LinkedIn)
   - Terms of Service explicitly prohibit scraping
   - Can result in legal action and IP bans

3. **Technical Barriers**
   - JavaScript-heavy React application
   - Dynamic content loading
   - Session-based authentication
   - Anti-automation tokens
   - Rate limiting per account

---

## Recommended Approach: Use Legitimate APIs

### Option 1: Proxycurl API (RECOMMENDED)
```
URL: https://nubela.co/proxycurl/
Cost: $10-49/month (10-100 credits)
Per Request: $0.10-0.30

Benefits:
✅ Legal access to LinkedIn data
✅ Structured JSON responses
✅ Person profile lookups
✅ Company employee search
✅ Email finder
✅ No LinkedIn account needed
✅ No anti-bot issues
```

**Example API Call:**
```bash
curl -X GET \
  'https://nubela.co/proxycurl/api/v2/linkedin?url=https://linkedin.com/in/username' \
  -H 'Authorization: Bearer YOUR_API_KEY'
```

### Option 2: Apollo.io API
```
URL: https://www.apollo.io/
Cost: $49-99/month
Per Request: Included in plan

Benefits:
✅ Built-in LinkedIn data
✅ Contact enrichment
✅ Company search
✅ Email verification
✅ CRM integrations
```

### Option 3: Clearbit API
```
URL: https://clearbit.com/
Cost: Contact for pricing
Per Request: ~$0.05-0.10

Benefits:
✅ Company enrichment
✅ Person enrichment
✅ Email-to-LinkedIn matching
✅ Reliable and fast
```

### Option 4: Hunter.io + LinkedIn Sales Navigator
```
Cost: $49+/month each
Benefits:
✅ Email finder
✅ Domain search
✅ LinkedIn data (via Sales Navigator)
```

---

## Workflow Architecture

### Workflow 1: Validate Existing Contacts

```
Schedule Trigger (Weekly)
    ↓
Read Contacts from Odoo
    ↓
Filter: Only active contacts
    ↓
Rate Limit (respect API limits)
    ↓
For Each Contact:
    ↓
    ├─ Has LinkedIn URL?
    │   ├─ YES → Fetch Profile via Proxycurl
    │   │         ↓
    │   │         Check Current Company
    │   │         ↓
    │   │         Company Matches?
    │   │         ├─ YES → Mark as "Verified"
    │   │         └─ NO → Mark as "Possibly Left"
    │   │
    │   └─ NO → Search Person via Company Name
    │             ↓
    │             Match Found?
    │             ├─ YES → Verify & Update LinkedIn URL
    │             └─ NO → Mark as "Unable to Verify"
    ↓
Update Contact in Odoo
    ↓
Generate Report (Google Sheet/Email)
```

### Workflow 2: Enrich Companies with Executives

```
Schedule Trigger (Monthly)
    ↓
Read Companies from Odoo
    ↓
Filter: Companies with < 2 contacts
    ↓
Rate Limit (1 request per 5 seconds)
    ↓
For Each Company:
    ↓
    Search Company on LinkedIn via Proxycurl
        ↓
        Get Employee List (filter by seniority)
            ↓
            Target Roles: CEO, CTO, CDO, CIO, CSO, VP, Director
                ↓
                For Each Executive:
                    ↓
                    Create Contact in Odoo
                        ↓
                        Fields:
                        - Name
                        - Title/Role
                        - Email (if available)
                        - LinkedIn URL
                        - Company Link
                        - Added via automation flag
    ↓
Log Results & Send Summary
```

---

## Odoo API Integration

### Authentication
```
Method: API Key or OAuth2
Endpoint: https://your-odoo-instance.com/web/dataset/call_kw

Required Permissions:
- Read/Write Contacts (res.partner)
- Read/Write Companies (res.partner)
- Custom Fields for verification status
```

### Required Odoo Fields

**On Contact (res.partner):**
- `linkedin_url` (char)
- `last_verified_date` (datetime)
- `verification_status` (selection: verified, unverified, left_company, unable_to_verify)
- `enrichment_source` (char: manual, linkedin_api)
- `job_title` (char)

**On Company (res.partner):**
- `linkedin_company_url` (char)
- `last_enriched_date` (datetime)
- `needs_enrichment` (boolean)

---

## Rate Limiting Strategy

### Proxycurl API Limits
```
Starter: 100 credits/month = 100 lookups
Professional: 500 credits/month
Business: 2000 credits/month

Recommended Pacing:
- 1 request every 3-5 seconds
- Maximum 100-200 requests per day
- Weekly validation cycle for contacts
- Monthly enrichment cycle for companies
```

### Implementation
```javascript
// Rate limiter: 1 request per 5 seconds
const DELAY_MS = 5000;

// Daily quota tracking
const MAX_DAILY_REQUESTS = 100;
let dailyRequestCount = 0;

// Check quota before request
if (dailyRequestCount >= MAX_DAILY_REQUESTS) {
  throw new Error('Daily API quota reached');
}
```

---

## Data Model

### Contact Validation Result
```json
{
  "odoo_contact_id": 12345,
  "contact_name": "John Doe",
  "company_name": "TechCorp",
  "linkedin_url": "https://linkedin.com/in/johndoe",
  "current_company_linkedin": "TechCorp Inc",
  "status": "verified|left_company|unable_to_verify",
  "confidence": 95,
  "last_checked": "2025-01-15T10:30:00Z",
  "new_company": null | "NewCompany Inc",
  "notes": "Title changed from Software Engineer to Senior Engineer"
}
```

### Executive Enrichment Result
```json
{
  "odoo_company_id": 789,
  "company_name": "TechCorp",
  "executives_found": [
    {
      "name": "Jane Smith",
      "title": "Chief Technology Officer",
      "linkedin_url": "https://linkedin.com/in/janesmith",
      "email": "jane.smith@techcorp.com",
      "seniority": "CXO",
      "created_in_odoo": true,
      "odoo_contact_id": 12346
    }
  ],
  "enriched_at": "2025-01-15T10:30:00Z"
}
```

---

## Security & Compliance

### Data Protection
- Store API keys securely (n8n credentials)
- Log all API requests for auditing
- Respect GDPR/CCPA for contact data
- Don't store LinkedIn passwords

### API Key Management
```
DO:
✅ Use environment variables
✅ Rotate keys regularly
✅ Monitor usage
✅ Set up alerts for anomalies

DON'T:
❌ Hardcode API keys
❌ Share keys across environments
❌ Ignore rate limits
❌ Use personal LinkedIn accounts for automation
```

---

## Cost Estimation

### Monthly Costs (Example: 500 Contacts, 100 Companies)

**Proxycurl:**
- Contact validation: 500 × $0.10 = $50
- Company enrichment: 100 × $0.30 = $30
- **Total: ~$80/month**

**Apollo.io:**
- Professional plan: $99/month
- Includes contact + company data
- **Total: $99/month**

**Clearbit:**
- Contact for pricing
- Typically $50-200/month for similar volume

---

## Implementation Priority

1. **Phase 1** (Week 1): Set up Odoo custom fields and API access
2. **Phase 2** (Week 2): Implement contact validation workflow
3. **Phase 3** (Week 3): Add company enrichment workflow
4. **Phase 4** (Week 4): Testing and monitoring
5. **Phase 5** (Ongoing): Weekly/monthly scheduled runs

---

## Success Metrics

- **Data Quality**: % of contacts verified as current
- **Coverage**: % of companies with key executive contacts
- **Efficiency**: Hours saved vs manual verification
- **Cost**: Cost per verified/enriched contact
- **Accuracy**: False positive/negative rate

---

## Alternative: Semi-Automated Approach

If API costs are prohibitive, consider:

1. **Export Odoo contacts to CSV**
2. **Use browser extension** (PhantomBuster, Dux-Soup) with rate limiting
3. **Manual spot-checking** of flagged contacts
4. **LinkedIn Sales Navigator** for company research (manual)
5. **Import updated data back to Odoo**

This reduces automation but stays within LinkedIn ToS when using official tools.
