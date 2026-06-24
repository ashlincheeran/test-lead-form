# Engage Lead Form Integration

Use this skill whenever you need to build or update a marketing landing page that submits leads into the Engage CRM (api.cencorpcms.com).

---

## How it works

The flow is:

```
User fills HTML form → Browser POSTs JSON → Engage API → Lead created in CRM
```

No Zapier. No backend. Just a plain HTML file with a `fetch()` call.

---

## The three IDs you always need

### 1. Bearer Token (API Key) — authenticates your app
- This is a **JWT token** that lives in your Engage account settings
- It is **shared across all forms** — you only have one per Engage account/environment
- Where to find it: Engage → Settings → API Keys (or copy from the Zapier webhook Headers section)
- Format: `Authorization: Bearer <token>`

### 2. Asset ID — identifies the project/property in Engage
- Stays the same for all forms under the same project
- Example: `63171df635754b46a4294287`
- Where to find it: Engage → your project/asset → URL or settings

### 3. Form ID — unique per form you create
- **Changes every time you create a new form** in Engage
- Where to find it: After saving a form in Engage, grab the hex ID from the URL
  - e.g. `https://app.cencorpcms.com/forms/6a3b686d0c7d4fe74ba1c8d1/edit` → ID is `6a3b686d0c7d4fe74ba1c8d1`

---

## Engage form setup checklist (do this in Engage first)

1. **Form Details tab**
   - Form Type: **Dynamic** (cannot change after saving)
   - Status: Active

2. **Form Fields tab** — add your fields, note the **Field Name** for each
   - Field names with spaces become underscores in the API (e.g. `Full Name` → `Full_Name`)

3. **Query String tab** — add: `utm_id`, `utm_source`, `utm_medium`, `utm_campaign`, `utm_term`, `utm_content`

4. **Form Actions tab** — add **Create Lead** action and map:
   - Phone Number → Phone field (required)
   - Source → default: `Website`
   - Contact Type → default: `Buyer` or `Seller`
   - Referred By → default: `Better Home`
   - Campaign Name → default: your campaign name

5. Save and copy the **Form ID** from the URL

---

## API payload structure

```json
POST https://api.cencorpcms.com/forms/entries
Authorization: Bearer <your-bearer-token>
Content-Type: application/json

{
  "asset": "<asset-id>",
  "form": "<form-id>",
  "formData": {
    "Full_Name": "John Smith",
    "Email": "john@example.com",
    "Phone": "+971501234567"
  },
  "queryStringData": {
    "utm_id": "",
    "utm_source": "google",
    "utm_medium": "cpc",
    "utm_campaign": "campaign-name",
    "utm_term": "",
    "utm_content": ""
  }
}
```

**Important:** Field name keys in `formData` must exactly match the Field Name set in Engage Form Fields, with spaces replaced by underscores.

---

## Field name mapping rule

| What you set in Engage "Field Name" | Key to use in formData |
|--------------------------------------|------------------------|
| `Full Name`                          | `Full_Name`            |
| `Phone`                              | `Phone`                |
| `Email`                              | `Email`                |
| `property_location`                  | `property_location`    |
| `own_this_property`                  | `own_this_property`    |

Rule: spaces → underscores. Everything else stays as-is.

---

## HTML template

When asked to build a new lead form, use this structure. Replace the three constants at the top of the script:

```html
<script>
  const ASSET_ID  = 'YOUR_ASSET_ID';
  const FORM_ID   = 'YOUR_FORM_ID';         // changes per form
  const API_TOKEN = 'YOUR_BEARER_TOKEN';    // same across all forms
  const API_URL   = 'https://api.cencorpcms.com/forms/entries';

  // UTM params are read automatically from the page URL query string
  function getUtmParams() {
    const p = new URLSearchParams(window.location.search);
    return {
      utm_id:       p.get('utm_id')       || '',
      utm_source:   p.get('utm_source')   || '',
      utm_medium:   p.get('utm_medium')   || '',
      utm_campaign: p.get('utm_campaign') || '',
      utm_term:     p.get('utm_term')     || '',
      utm_content:  p.get('utm_content')  || '',
    };
  }

  async function submitLead(formData) {
    const res = await fetch(API_URL, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer ' + API_TOKEN,
      },
      body: JSON.stringify({
        asset: ASSET_ID,
        form:  FORM_ID,
        formData: formData,
        queryStringData: getUtmParams(),
      }),
    });
    if (!res.ok) throw new Error(await res.text());
    return res.json();
  }
</script>
```

---

## Common errors and fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `401 Please authenticate first` | Missing or wrong Bearer token | Add/correct the `Authorization` header |
| `Full Name is not a valid form field` | Space in field name key | Use underscore: `Full_Name` |
| `Phone is required` | Field name key mismatch | Check exact Field Name in Engage form settings |
| `400 Bad Request` | Form ID wrong or form not saved | Confirm form ID from Engage URL after saving |

---

## Adding UTM tracking to ad URLs

Append UTM params to your landing page URL and they'll be captured automatically:

```
https://yourpage.vercel.app?utm_source=facebook&utm_medium=paid&utm_campaign=BH-June-2026
```

These map directly to the UTM fields in Engage on the lead record.
