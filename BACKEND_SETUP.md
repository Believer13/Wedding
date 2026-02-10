# Backend Setup Instructions

## Option 1: Google Sheets (Recommended for Wedding Invitations)

### Step 1: Create a Google Sheet
1. Go to [Google Sheets](https://sheets.google.com)
2. Create a new spreadsheet
3. Name it "Wedding RSVP Responses"
4. In the first row, add these headers:
   - A1: `Timestamp`
   - B1: `Name`
   - C1: `Attendance`
   - D1: `Drink Preferences`

### Step 2: Create Google Apps Script
1. In your Google Sheet, click **Extensions** → **Apps Script**
2. Delete any code in the editor
3. Paste the following code:

```javascript
function doPost(e) {
  try {
    // Get the active spreadsheet
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    
    // Parse the incoming data
    var data = JSON.parse(e.postData.contents);
    
    // Add a new row with the data
    sheet.appendRow([
      data.timestamp,
      data.name,
      data.attendance,
      data.drinks
    ]);
    
    // Return success response
    return ContentService.createTextOutput(JSON.stringify({
      'result': 'success',
      'data': data
    })).setMimeType(ContentService.MimeType.JSON);
    
  } catch(error) {
    // Return error response
    return ContentService.createTextOutput(JSON.stringify({
      'result': 'error',
      'error': error.toString()
    })).setMimeType(ContentService.MimeType.JSON);
  }
}
```

### Step 3: Deploy the Script
1. Click **Deploy** → **New deployment**
2. Click the gear icon ⚙️ next to "Select type"
3. Choose **Web app**
4. Configure:
   - **Description**: Wedding RSVP Form
   - **Execute as**: Me
   - **Who has access**: Anyone
5. Click **Deploy**
6. Click **Authorize access** and grant permissions
7. **Copy the Web App URL** - it will look like:
   ```
   https://script.google.com/macros/s/AKfycby.../exec
   ```

### Step 4: Update Your HTML File
1. Open `index.html`
2. Find this line (around line 426):
   ```javascript
   const GOOGLE_SCRIPT_URL = 'YOUR_GOOGLE_SCRIPT_URL_HERE';
   ```
3. Replace `YOUR_GOOGLE_SCRIPT_URL_HERE` with your Web App URL
4. Save the file

### Step 5: Test
1. Open your `index.html` in a browser
2. Fill out the form and submit
3. Check your Google Sheet - the response should appear!

---

## Option 2: Simple Node.js Backend (For More Control)

### Step 1: Create Backend Files
Create a new folder called `backend` and add these files:

**package.json:**
```json
{
  "name": "wedding-rsvp-backend",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "cors": "^2.8.5",
    "body-parser": "^1.20.2",
    "sqlite3": "^5.1.6"
  }
}
```

**server.js:**
```javascript
const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');
const sqlite3 = require('sqlite3').verbose();

const app = express();
const PORT = 3000;

// Middleware
app.use(cors());
app.use(bodyParser.json());

// Database setup
const db = new sqlite3.Database('./rsvp.db', (err) => {
  if (err) console.error(err);
  else console.log('Database connected');
});

// Create table
db.run(`CREATE TABLE IF NOT EXISTS responses (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL,
  attendance TEXT NOT NULL,
  drinks TEXT,
  timestamp TEXT NOT NULL
)`);

// API endpoint to save RSVP
app.post('/api/rsvp', (req, res) => {
  const { name, attendance, drinks, timestamp } = req.body;
  
  const sql = 'INSERT INTO responses (name, attendance, drinks, timestamp) VALUES (?, ?, ?, ?)';
  db.run(sql, [name, attendance, drinks, timestamp], function(err) {
    if (err) {
      res.status(500).json({ error: err.message });
    } else {
      res.json({ success: true, id: this.lastID });
    }
  });
});

// API endpoint to get all RSVPs
app.get('/api/rsvps', (req, res) => {
  db.all('SELECT * FROM responses ORDER BY timestamp DESC', [], (err, rows) => {
    if (err) {
      res.status(500).json({ error: err.message });
    } else {
      res.json(rows);
    }
  });
});

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

### Step 2: Install and Run
```bash
cd backend
npm install
npm start
```

### Step 3: Update HTML
Replace the GOOGLE_SCRIPT_URL with:
```javascript
const API_URL = 'http://localhost:3000/api/rsvp';
```

And update the fetch call:
```javascript
const response = await fetch(API_URL, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify(formData)
});

const result = await response.json();
if (result.success) {
  alert('Спасибо! Ваш ответ принят.');
  form.reset();
}
```

---

## Option 3: Formspree (Easiest Paid Option)

1. Go to [Formspree.io](https://formspree.io)
2. Create a free account
3. Create a new form
4. Copy your form endpoint URL
5. Update the fetch URL in your HTML

---

## Viewing Responses

### For Google Sheets:
- Simply open your Google Sheet to see all responses in real-time

### For Node.js Backend:
- Go to `http://localhost:3000/api/rsvps` to see all responses as JSON
- Or create a simple admin page to view the data

---

## Recommendation

For a wedding invitation, **Google Sheets (Option 1)** is the best choice because:
- ✅ Free
- ✅ No server maintenance
- ✅ Easy to share with wedding planners
- ✅ Can export to Excel
- ✅ Real-time updates
- ✅ No coding experience needed

Just follow the Google Sheets setup steps above!
