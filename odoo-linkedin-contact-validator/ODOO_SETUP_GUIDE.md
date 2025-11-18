# Odoo Custom Fields Setup Guide

This guide helps you configure the required custom fields in Odoo for the LinkedIn Contact Validator and Company Enricher workflows.

## Prerequisites

- Odoo instance (v14+)
- Administrator access
- Developer mode enabled

---

## Enable Developer Mode

1. Go to **Settings** → **General Settings**
2. Scroll to **Developer Tools**
3. Click **Activate the developer mode**

---

## Adding Custom Fields to res.partner (Contacts)

### Method 1: Via UI (Recommended)

1. Go to **Settings** → **Technical** → **Models**
2. Search for `res.partner`
3. Click on the model
4. Go to **Fields** tab
5. Click **Create** for each field below

### Method 2: Via Python (Advanced)

Add to your custom module's `models/res_partner.py`:

```python
from odoo import models, fields

class ResPartner(models.Model):
    _inherit = 'res.partner'

    # LinkedIn Integration Fields
    x_linkedin_url = fields.Char(
        string='LinkedIn URL',
        help='LinkedIn profile URL for this contact'
    )

    x_linkedin_company_url = fields.Char(
        string='LinkedIn Company URL',
        help='LinkedIn company page URL (for companies only)'
    )

    # Verification Fields
    x_last_verified = fields.Datetime(
        string='Last Verified',
        help='Date when employment status was last verified'
    )

    x_verification_status = fields.Selection([
        ('unverified', 'Not Verified'),
        ('verified', 'Verified'),
        ('left_company', 'Left Company'),
        ('unable_to_verify', 'Unable to Verify')
    ], string='Verification Status', default='unverified')

    x_verification_notes = fields.Text(
        string='Verification Notes',
        help='Notes from the last verification attempt'
    )

    # Enrichment Fields
    x_enrichment_source = fields.Selection([
        ('manual', 'Manual Entry'),
        ('linkedin_api', 'LinkedIn API'),
        ('import', 'CSV Import')
    ], string='Enrichment Source', default='manual')

    x_seniority_level = fields.Selection([
        ('c_suite', 'C-Suite'),
        ('vp_director', 'VP/Director'),
        ('manager', 'Manager'),
        ('individual', 'Individual Contributor')
    ], string='Seniority Level')

    x_last_enriched = fields.Datetime(
        string='Last Enriched',
        help='Date when company was last enriched with contacts'
    )

    x_needs_enrichment = fields.Boolean(
        string='Needs Enrichment',
        default=False,
        help='Flag to indicate company needs contact enrichment'
    )

    x_enrichment_notes = fields.Text(
        string='Enrichment Notes',
        help='Notes from the last enrichment attempt'
    )
```

---

## Field Definitions (Manual Setup)

### For Contacts (Individual Partners)

| Field Name | Technical Name | Type | Description |
|------------|----------------|------|-------------|
| LinkedIn URL | x_linkedin_url | Char | Contact's LinkedIn profile URL |
| Last Verified | x_last_verified | Datetime | When employment was last checked |
| Verification Status | x_verification_status | Selection | Current verification state |
| Verification Notes | x_verification_notes | Text | Details from last verification |
| Enrichment Source | x_enrichment_source | Selection | How contact was added |
| Seniority Level | x_seniority_level | Selection | Executive level |

### For Companies (Company Partners)

| Field Name | Technical Name | Type | Description |
|------------|----------------|------|-------------|
| LinkedIn Company URL | x_linkedin_company_url | Char | Company's LinkedIn page URL |
| Last Enriched | x_last_enriched | Datetime | When company was last enriched |
| Needs Enrichment | x_needs_enrichment | Boolean | Flag for pending enrichment |
| Enrichment Notes | x_enrichment_notes | Text | Details from last enrichment |

---

## Step-by-Step UI Field Creation

### Example: Creating x_linkedin_url

1. Navigate to **Settings** → **Technical** → **Fields**
2. Click **Create**
3. Fill in:
   - **Field Name**: LinkedIn URL
   - **Technical Name**: x_linkedin_url
   - **Model**: res.partner
   - **Field Type**: Char
   - **Help Text**: LinkedIn profile URL for this contact
4. Click **Save**

### Example: Creating x_verification_status

1. Navigate to **Settings** → **Technical** → **Fields**
2. Click **Create**
3. Fill in:
   - **Field Name**: Verification Status
   - **Technical Name**: x_verification_status
   - **Model**: res.partner
   - **Field Type**: Selection
   - **Selection Values**:
     ```
     [('unverified', 'Not Verified'),
      ('verified', 'Verified'),
      ('left_company', 'Left Company'),
      ('unable_to_verify', 'Unable to Verify')]
     ```
   - **Default Value**: unverified
4. Click **Save**

---

## Adding Fields to Form View

After creating fields, add them to the contact form view:

### Method 1: Via Studio (Easiest)

1. Open **Contacts** app
2. Open any contact
3. Click **Studio** icon (if you have Odoo Studio)
4. Drag and drop new fields onto the form

### Method 2: Via XML

Create a view extension in your custom module:

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_partner_form_linkedin" model="ir.ui.view">
        <field name="name">res.partner.form.linkedin</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="arch" type="xml">
            <page name="internal_notes" position="before">
                <page string="LinkedIn Data" name="linkedin_data">
                    <group string="LinkedIn Information">
                        <field name="x_linkedin_url" widget="url"/>
                        <field name="x_linkedin_company_url" widget="url"
                               attrs="{'invisible': [('is_company', '=', False)]}"/>
                    </group>
                    <group string="Verification Status"
                           attrs="{'invisible': [('is_company', '=', True)]}">
                        <field name="x_verification_status"/>
                        <field name="x_last_verified"/>
                        <field name="x_enrichment_source"/>
                        <field name="x_seniority_level"/>
                    </group>
                    <group string="Verification Notes"
                           attrs="{'invisible': [('is_company', '=', True)]}">
                        <field name="x_verification_notes" nolabel="1"/>
                    </group>
                    <group string="Enrichment Status"
                           attrs="{'invisible': [('is_company', '=', False)]}">
                        <field name="x_last_enriched"/>
                        <field name="x_needs_enrichment"/>
                    </group>
                    <group string="Enrichment Notes"
                           attrs="{'invisible': [('is_company', '=', False)]}">
                        <field name="x_enrichment_notes" nolabel="1"/>
                    </group>
                </page>
            </page>
        </field>
    </record>
</odoo>
```

---

## Odoo API Configuration

### Enable API Access

1. Go to **Settings** → **Users & Companies** → **Users**
2. Select your API user
3. Go to **Preferences** tab
4. Generate **API Key** (or use password)

### Test API Connection

```bash
curl -X POST \
  https://your-odoo-instance.com/web/dataset/call_kw \
  -H 'Content-Type: application/json' \
  -u 'api_user:api_key' \
  -d '{
    "jsonrpc": "2.0",
    "method": "call",
    "id": 1,
    "params": {
      "model": "res.partner",
      "method": "search_read",
      "args": [],
      "kwargs": {
        "domain": [["is_company", "=", true]],
        "fields": ["id", "name"],
        "limit": 5
      }
    }
  }'
```

---

## n8n Credentials Setup

### 1. Odoo HTTP Basic Auth

1. In n8n, go to **Credentials**
2. Click **Add Credential**
3. Search for **HTTP Basic Auth**
4. Fill in:
   - **Name**: Odoo API
   - **User**: your_api_user
   - **Password**: your_api_key (or password)

### 2. Proxycurl API Key

1. In n8n, go to **Credentials**
2. Click **Add Credential**
3. Search for **HTTP Header Auth**
4. Fill in:
   - **Name**: Proxycurl API
   - **Header Name**: Authorization
   - **Header Value**: Bearer YOUR_PROXYCURL_API_KEY

### 3. Environment Variables

In your n8n environment (`.env` file or container config):

```env
ODOO_URL=https://your-odoo-instance.com
```

Or set in n8n:
1. Go to **Settings** → **Credentials**
2. Add credential type **Environment Variables**
3. Set `ODOO_URL`

---

## Verification Workflow Filter Logic

The workflow uses this Odoo domain filter:

```python
# Contacts needing verification:
[
    ("is_company", "=", False),      # Individual contacts only
    ("parent_id", "!=", False),       # Must have a parent company
    ("active", "=", True)             # Active contacts only
]

# Companies needing enrichment:
[
    ("is_company", "=", True),        # Companies only
    ("active", "=", True),            # Active companies
    ("child_ids", "=", False)         # No child contacts
]
```

---

## Testing the Setup

### 1. Verify Fields Exist

```bash
curl -X POST https://your-odoo-instance.com/web/dataset/call_kw \
  -H 'Content-Type: application/json' \
  -u 'api_user:api_key' \
  -d '{
    "jsonrpc": "2.0",
    "method": "call",
    "id": 1,
    "params": {
      "model": "ir.model.fields",
      "method": "search_read",
      "args": [],
      "kwargs": {
        "domain": [
          ["model", "=", "res.partner"],
          ["name", "like", "x_linkedin"]
        ],
        "fields": ["name", "ttype", "field_description"]
      }
    }
  }'
```

### 2. Test Creating a Contact

```bash
curl -X POST https://your-odoo-instance.com/web/dataset/call_kw \
  -H 'Content-Type: application/json' \
  -u 'api_user:api_key' \
  -d '{
    "jsonrpc": "2.0",
    "method": "call",
    "id": 1,
    "params": {
      "model": "res.partner",
      "method": "create",
      "args": [{
        "name": "Test Contact",
        "is_company": false,
        "parent_id": 1,
        "x_linkedin_url": "https://linkedin.com/in/test",
        "x_verification_status": "verified"
      }],
      "kwargs": {}
    }
  }'
```

### 3. Test Updating a Contact

```bash
curl -X POST https://your-odoo-instance.com/web/dataset/call_kw \
  -H 'Content-Type: application/json' \
  -u 'api_user:api_key' \
  -d '{
    "jsonrpc": "2.0",
    "method": "call",
    "id": 1,
    "params": {
      "model": "res.partner",
      "method": "write",
      "args": [
        [YOUR_CONTACT_ID],
        {
          "x_verification_status": "left_company",
          "x_verification_notes": "No longer at company"
        }
      ],
      "kwargs": {}
    }
  }'
```

---

## Common Issues & Solutions

### Issue: "Field x_linkedin_url does not exist"

**Solution**: Field wasn't created. Follow the creation steps above.

### Issue: "Access Denied"

**Solution**: Check API user permissions. User needs:
- Read/Write access to res.partner
- Technical Features group (for custom fields)

### Issue: "Invalid field type"

**Solution**: Ensure Selection fields have proper value format:
```python
[('key1', 'Label 1'), ('key2', 'Label 2')]
```

### Issue: "JSONRPC Error"

**Solution**: Check JSON syntax in workflow. Use a JSON validator.

---

## Security Considerations

1. **API User Permissions**: Create a dedicated API user with minimal necessary permissions
2. **IP Whitelisting**: Restrict API access to your n8n server IP
3. **Rate Limiting**: Odoo may have rate limits; respect them
4. **Audit Logging**: Enable Odoo audit logs to track API changes
5. **Data Privacy**: Ensure LinkedIn data usage complies with GDPR/CCPA

---

## Next Steps

1. ✅ Create custom fields in Odoo
2. ✅ Add fields to contact form view
3. ✅ Configure API credentials in n8n
4. ✅ Test API connection
5. ✅ Import n8n workflows
6. ✅ Run test execution with 2-3 contacts
7. ✅ Schedule regular executions
8. ✅ Monitor results and costs

---

## Support

- **Odoo Documentation**: https://www.odoo.com/documentation/
- **Odoo API Reference**: https://www.odoo.com/documentation/14.0/developer/reference/external_api.html
- **n8n Community**: https://community.n8n.io/
- **Proxycurl Docs**: https://nubela.co/proxycurl/docs/
