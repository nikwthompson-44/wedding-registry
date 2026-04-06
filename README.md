# Wedding Gift Registry

Simple gift registry backed by a Google Sheet, embedded via iframe on our Webflow wedding site.

## Setup

### 1. Create the Google Sheet

1. Create a new Google Sheet
2. Rename the first tab to **Registry**
3. Add these headers in row 1:

| A | B | C | D | E | F |
|---|---|---|---|---|---|
| Item_Name | Image_1 | Image_2 | Description | URL | Taken |

4. Add gift items starting at row 2. Leave the `Taken` column blank.

**Image URLs** must be direct links. For Google Drive images, use this format:
```
https://drive.google.com/uc?id=YOUR_FILE_ID
```
(Right-click the file in Drive > "Get link" > copy the file ID from the share URL)

### 2. Add the Apps Script

1. In the Google Sheet, go to **Extensions > Apps Script**
2. Delete any default code and paste the following:

```javascript
function doGet(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Registry');
  var data = sheet.getDataRange().getValues();
  var items = [];

  for (var i = 1; i < data.length; i++) {
    var row = data[i];
    if (!row[0]) continue;
    items.push({
      row: i + 1,
      name: row[0],
      image1: row[1],
      image2: row[2],
      description: row[3],
      url: row[4],
      taken: row[5] === true || row[5] === 'TRUE'
    });
  }

  return ContentService
    .createTextOutput(JSON.stringify({ items: items }))
    .setMimeType(ContentService.MimeType.JSON);
}

function doPost(e) {
  var body = JSON.parse(e.postData.contents);
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Registry');
  var currentValue = sheet.getRange(body.row, 6).getValue();

  if (currentValue === true || currentValue === 'TRUE') {
    return ContentService
      .createTextOutput(JSON.stringify({ success: false, error: 'Already taken' }))
      .setMimeType(ContentService.MimeType.JSON);
  }

  sheet.getRange(body.row, 6).setValue(true);

  return ContentService
    .createTextOutput(JSON.stringify({ success: true }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

3. Click **Deploy > New deployment**
4. Type: **Web app**
5. Execute as: **Me**
6. Who has access: **Anyone**
7. Click **Deploy** and authorize when prompted
8. Copy the deployment URL

### 3. Configure the frontend

Open `index.html` and find the `CONFIG` object near the top of the `<script>` block:

```javascript
const CONFIG = {
  SCRIPT_URL: 'YOUR_GOOGLE_APPS_SCRIPT_URL_HERE',
  PAYPAL_URL: 'YOUR_PAYPAL_ME_URL_HERE'
};
```

Replace both placeholder values.

### 4. Test locally

Open `index.html` in a browser (double-click the file or run `npx serve .`).

### 5. Deploy to GitHub Pages

```bash
gh repo create wedding-registry --public --source=. --push
```

Then go to the repo on GitHub > **Settings > Pages** > Source: **main** branch, root folder.

### 6. Embed in Webflow

Add an **Embed** block on your Webflow page with:

```html
<iframe
  src="https://YOUR_USERNAME.github.io/wedding-registry/"
  style="width:100%; min-height:800px; border:none;"
  loading="lazy"
></iframe>
```

## Updating the registry

- **Add/edit/remove items**: Edit the Google Sheet directly. The page loads fresh data on every visit.
- **Un-claim an item** (emergency): Clear the `Taken` cell in the sheet.
- **Update the Apps Script**: If you change the script code, you must create a new deployment version via **Deploy > Manage deployments > Edit > New version**. The URL stays the same.
