# Enterprise DE Lookup

A CloudPage-based tool for searching and exploring Data Extensions across your Salesforce Marketing Cloud Business Unit, without needing to navigate through folders manually.

---

## What It Does

- **Search DEs by Name or External Key** using a keyword lookup
- **Displays the full folder path** of every matching Data Extension
- **Shows key metadata** including whether each DE is Sendable and Testable
- **Highlights your search term** in results for quick scanning
- **Exports results to CSV** for auditing or documentation
- **Paginates large result sets** (25 per page) to keep the UI fast
- **Handles large instances gracefully** using WSProxy batch retrieval to avoid timeouts on BUs with hundreds or thousands of automations

---

## Files

| File | Description |
|------|-------------|
| `root.html` | CloudPage layout shell. Contains all CSS styling and client-side JavaScript for loading state, pagination, and CSV export |
| `lookup.html` | CloudPage content slot. Contains the search form and SSJS logic for retrieving and rendering results |

---

## Setup Instructions

### Step 1: Create the CloudPage

1. In SFMC, go to **Web Studio → CloudPages**
2. Create a new **Collection** (or use an existing one)
3. Add a new **Landing Page**

### Step 2: Set up the Layout (root.html)

1. In your Landing Page, go to the **Code View** tab
2. Paste the entire contents of `root.html` into the editor
3. Save    

### Step 3: Add the Content (lookup.html)

1. Go to the **Default** tab
2. Add a new **HTML Content** block into the layout's content slot
3. Paste the entire contents of `lookup.html` into the content block
4. Save

### Step 4: Publish

1. Click **Publish** on your Landing Page
2. Copy the published CloudPage URL
3. That URL is your tool. Share it with your team!

> **Note:** The tool uses `Request.GetQueryStringParameter()` to read form submissions, so it requires the page to be **published and accessed via its live URL**. Preview mode will show the form but won't return results (this is expected SFMC behaviour).

> ⚠️ **Security Warning:** Once published, this CloudPage is publicly accessible to anyone with the URL. It exposes the names, folder paths, and metadata of Data Extensions inside your Business Unit. **Do not share the link publicly or with anyone outside your organisation.** Treat the URL as you would any internal tool or sensitive resource. If broader access control is needed, consider restricting access through your network or implementing an additional authentication layer.

---

## How to Use

1. Open the published CloudPage URL in your browser
2. Select your **search type** from the dropdown:
   - **Name or Keyword** searches DE names containing your term
   - **External Key** searches by CustomerKey
3. Type your **search term** in the text field
4. Click **Search**
5. Results appear in a table showing:
   - DE name (with your search term highlighted)
   - Full folder path
   - External Key
   - Sendable status
   - Testable status

### Exporting Results

Click the **↓ Export CSV** button above the results table to download all matching DEs as a CSV file. Useful for audits, documentation, or sharing with your team.

### Pagination

If your search returns more than 25 results, use the **Prev / Next** buttons at the bottom to navigate through pages.

---

## How It Works (Technical Overview)

### Timeout Fix: WSProxy Batch Retrieval

The original version used `DataExtension.Retrieve()` which pulls all records in a single call. This causes SFMC to time out on large Business Units.

This version uses **WSProxy with `getNextBatch()`** to retrieve DEs in paginated batches:

```javascript
var api  = new Script.Util.WSProxy();
var data = api.retrieve("DataExtension", cols, filter);

while (data.HasMoreRows) {
  data = api.getNextBatch("DataExtension", data.RequestID);
  // process next batch...
}
```

This means the tool works reliably even on enterprise instances with thousands of Data Extensions.

### Folder Path Resolution

For each DE found, the tool recursively walks up the folder tree using `Folder.Retrieve()` to build the full path (e.g. `Data Extensions > Campaigns > Email > Newsletter`). A local cache prevents redundant API calls for shared parent folders.

### Search Highlighting

Matching text is highlighted in yellow directly in the results table, making it easy to see why each DE was returned. Especially useful for broad keyword searches.

---

## Requirements

- Salesforce Marketing Cloud account with **Web Studio / CloudPages** access
- User must have permission to retrieve Data Extensions and Folders in the target Business Unit

---

## Limitations

- Search is **case-insensitive** but uses a `LIKE` operator. It matches any DE whose name/key *contains* your search term.
- Folder path resolution adds API calls per unique folder. On very large result sets this may still be slow. Consider narrowing your search term if results are slow to load.
- Only searches within the **current Business Unit**. For multi-BU setups, deploy a separate instance per BU.

---

## Credits

Crafted by **Karla Roman**  
[karlaoncloud.space](https://www.karlaoncloud.space) · [GitHub](https://github.com/ksolroman) · [LinkedIn](https://linkedin.com/in/ksolroman)

*Built with SSJS, WSProxy, and CloudPages on Salesforce Marketing Cloud.*
