# FreeJobAlert n8n Workflow

This workflow automatically fetches the latest job listings from **FreeJobAlert**, extracts **job titles, links, and qualifications**, and sends **only new job alerts** to your **Telegram** chat.

---

## 🚀 Features

- Fetches latest job listings from FreeJobAlert
- Extracts:
  - **Job Title**
  - **Job Link**
  - **Qualification**
- Cleans and formats qualification text
- **Deduplicates** jobs using workflow static data
- Sends **only new job alerts** to Telegram
- Avoids repeated notifications

---

## 🛠 Workflow Steps

1. **Schedule Trigger**
   - Runs every 12 hours (can be adjusted)

2. **Pull FreeJobAlert Homepage**
   - HTTP Request node fetching the website

3. **Job Listings (HTML Extract)**
   - Extracts arrays of job titles and links

4. **Split Jobs**
   - Turns the arrays into **individual items** for processing

5. **Fetch Job Page → Extract Qualification**
   - Fetches each job’s page
   - Extracts the qualification field

6. **Code Node (Clean & Deduplicate)**
   - Cleans `qualification` text:
     - Removes `Qualification`, `*`, and newlines
   - Filters only **new jobs** using:
     ```js
     const workflowStaticData = $getWorkflowStaticData('global');
     if (!workflowStaticData.jobs) workflowStaticData.jobs = [];

     const currentLinks = items.map(item => item.json.link);
     const newItems = items.filter(item => !workflowStaticData.jobs.includes(item.json.link));

     workflowStaticData.jobs = [...new Set([...workflowStaticData.jobs, ...currentLinks])];

     return newItems;
     ```

7. **Telegram → Send Message**
   - Sends a formatted message for each new job:
     ```
     📢 New Job Alert!

     🔹 {{$json["title"]}}
     🔗 {{$json["link"]}}
     🎓 {{$json["qualification"]}}
     ```
   - **Parse Mode = None** (to avoid Markdown errors)

---

## 💬 Telegram Setup

1. **Create a Bot**
   - Open Telegram → `@BotFather`
   - `/newbot` → Choose a name and username
   - Copy your **Bot Token**

2. **Get Chat ID**
   - Send `/start` to your bot
   - Visit:
     ```
     https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates
     ```
   - Look for `"chat":{"id":...}` → That’s your Chat ID

3. **Configure in n8n**
   - Add **Telegram node** with:
     - **Bot Token** = BotFather token
     - **Chat ID** = your ID (e.g., `1096596645`)
     - **Parse Mode** = None

---

## 📝 Notes

- First run → Sends all jobs  
- Next runs → Sends only **new jobs**  
- Deduplication is persistent using **workflow static data**  
- If FreeJobAlert changes HTML structure, update the **CSS selectors** in HTML nodes

---

## 📤 Example Telegram Message

```

📢 New Job Alert!

🔹 Manager - Sales
🔗 https://www.freejobalert.com/job1
🎓 Graduation in any discipline | 4-year Degree (Agriculture / etc)

```

## ✅ Tips

- To send to **WhatsApp**, replace Telegram node with:
  - **Twilio → Send WhatsApp** node  
  - or **HTTP Request** node with **WhatsApp Cloud API**  
- You can extend the workflow to:
  - Save jobs in Google Sheets
  - Email alerts
  - Push notifications
