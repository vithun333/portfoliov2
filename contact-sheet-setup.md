# Connect the contact form to your Google Sheet

Your forms already POST to a Google Apps Script endpoint. You just need to create that
script (2 minutes), deploy it, and paste the URL into the two HTML files.

Sheet: https://docs.google.com/spreadsheets/d/1VqyIJASZcKhnz0x_QMiZ_byXOI3g8hIavlexuTlAPTA/edit

---

## Step 1 — Add a header row (optional but nice)
Open the sheet and put these in row 1: **Timestamp | Name | Email | Message**

## Step 2 — Open the script editor
In the sheet: **Extensions → Apps Script**. Delete anything in the editor and paste this:

```javascript
function doPost(e) {
  try {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];
    sheet.appendRow([
      new Date(),
      e.parameter.name || '',
      e.parameter.email || '',
      e.parameter.message || ''
    ]);
    return ContentService
      .createTextOutput(JSON.stringify({ result: 'success' }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ result: 'error', error: String(err) }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

Click the **Save** icon.

## Step 3 — Deploy as a Web App
1. Top-right: **Deploy → New deployment**
2. Click the gear icon → choose **Web app**
3. Set:
   - **Execute as:** Me (your account)
   - **Who has access:** **Anyone**   ← important, so visitors can submit
4. Click **Deploy**, authorize when prompted (allow the permissions).
5. Copy the **Web app URL** — it looks like
   `https://script.google.com/macros/s/AKfy....../exec`

## Step 4 — Paste the URL into the site
In **both** `index.html` and `mobile.html`, find this line:

```javascript
const SHEET_ENDPOINT="PASTE_YOUR_GOOGLE_APPS_SCRIPT_URL_HERE";
```

Replace the placeholder with your copied URL (keep the quotes):

```javascript
const SHEET_ENDPOINT="https://script.google.com/macros/s/AKfy....../exec";
```

## Step 5 — Test
Open the site, submit the contact form. A new row should appear in the sheet within a
couple seconds, and the form shows a "Thanks — your message has been sent" confirmation.

---

### Notes
- If you ever change the script, redeploy with **Deploy → Manage deployments → edit (pencil)
  → Version: New version → Deploy** (the URL stays the same).
- Submissions are sent quietly in the background; the visitor doesn't need an email app.
- Want an email alert on each submission too? Add this inside `doPost`, before the return:
  `MailApp.sendEmail('vithun.ux@gmail.com','New portfolio message from '+e.parameter.name, e.parameter.email+'\n\n'+e.parameter.message);`
