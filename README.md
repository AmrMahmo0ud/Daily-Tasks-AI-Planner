# Daily Tasks — AI Project Planner (n8n Workflow)

An automated daily planning workflow built in n8n. Every morning at 9 AM it reads your active projects from a Google Sheet, analyzes deadlines vs progress using Google Gemini AI, and sends a prioritized Arabic work plan directly to your Telegram.

---

## What It Does

Every day at 9:00 AM the workflow:

1. Reads all your current projects from a Google Sheet (task name, start date, deadline, progress %)
2. Aggregates all rows into a single structured input
3. Sends the data to **Google Gemini AI** with a time management prompt that:
   - Calculates days remaining until each deadline
   - Compares progress % against time remaining
   - Flags any project in the "danger zone" (low progress + close deadline)
   - Allocates more time in today's schedule to overdue projects
   - Generates a motivating, structured Arabic daily work plan
4. Sends the plan to **Telegram**

---

## Workflow Diagram

```
Schedule Trigger (9:00 AM daily)
        │
        ▼
Get rows ← Google Sheet (AI Planner)
   • Task / Project
   • Start Date
   • End Date / Deadline
   • Progress %
        │
        ▼
Aggregate — combine all rows into one item
        │
        ▼
Basic LLM Chain (Google Gemini)
   • Calculates days remaining per project
   • Detects danger zone projects
   • Builds prioritized daily schedule
   • Outputs motivating Arabic work plan
        │
        ▼
Telegram — sends plan to your chat
```

---

## Nodes Breakdown

| Node | Type | Purpose |
|---|---|---|
| Schedule Trigger | Trigger | Fires every day at 9:00 AM |
| Get row(s) in sheet | Google Sheets | Reads all project rows from the AI Planner sheet |
| Aggregate | Aggregate | Combines all rows into a single item for the LLM prompt |
| Basic LLM Chain | LangChain | Generates the daily plan using Gemini |
| Google Gemini Chat Model | AI Model | Powers the LLM chain (gemini-3.1-flash-lite) |
| Send a text message | Telegram | Delivers the plan to your Telegram chat |

---

## AI Planning Logic

The Gemini prompt acts as an Agile project management expert and does the following analysis automatically:

**Step 1 — Deadline proximity**
Calculates the number of days remaining until each project's deadline based on today's date.

**Step 2 — Progress vs time comparison**
- If a project is **close to its deadline but has low progress** → marked as high priority, allocated more hours in today's plan
- If a project is **on track or ahead of schedule** → allocated normal or light time for continuity

**Step 3 — Report generation**
- Opens with a **danger zone alert** if any project is behind
- Lists today's tasks with suggested time slots (e.g. "4 PM – 6 PM")
- Explains the reason for each time allocation
- Written in clear, motivating Arabic (Modern Standard Arabic)

---

## Google Sheet Structure

The workflow reads from a sheet named **AI Planner** with the following columns:

| Column | Description | Example |
|---|---|---|
| `Task/ Project` | Project or task name | Website Redesign |
| `Start Date` | When the project started | 01/06/2025 |
| `End Date/Deadline` | Target completion date | 30/07/2025 |
| `Progress` | Current completion percentage | 45% |

Each row is one project. Add as many rows as needed — the workflow reads all of them automatically.

---

## Sample Output (Telegram Message)

```
📋 خطة عمل اليوم — الثلاثاء 15 يوليو 2025

⚠️ تنبيه: مشروع "تصميم الموقع" في منطقة الخطر!
نسبة الإنجاز 30% فقط وتبقى 5 أيام على الـ Deadline.

📌 جدول اليوم:

1. تصميم الموقع — من 9 صباحاً لـ 12 ظهراً (3 ساعات)
   ✦ تم تخصيص وقت أطول بسبب التأخر الحاد في الإنجاز.

2. تطوير API — من 1 لـ 3 مساءً (ساعتان)
   ✦ المشروع يسير بشكل جيد، وقت عادي للاستمرارية.

3. إعداد التقارير — من 4 لـ 5 مساءً (ساعة)
   ✦ نسبة إنجاز ممتازة، مراجعة خفيفة فقط.

💪 ركز على الأولويات واحتسب كل دقيقة. أنت قادر!
```

---

## Prerequisites

Before importing the workflow you need:

- An **n8n** instance (self-hosted or cloud)
- A **Google account** with Google Sheets access
- A **Google Gemini API** key
- A **Telegram bot** and your chat ID

---

## Setup Instructions

### 1. Import the workflow

In n8n: **Workflows → Import from file** → select the JSON file.

### 2. Configure credentials

Set up the following credentials in n8n (**Settings → Credentials**):

| Credential | Used By | What to add |
|---|---|---|
| `Google Sheets OAuth2 API` | Get row(s) in sheet | OAuth2 with your Google account |
| `Google Gemini (PaLM) API` | Gemini Chat Model | Your Gemini API key |
| `Telegram API` | Send a text message | Your Telegram bot token |

### 3. Create the Google Sheet

Create a new Google Sheet named `AI Planner` with these columns in row 1:

```
Task/ Project | Start Date | End Date/Deadline | Progress
```

Fill in your current projects. The workflow reads all rows every morning.

Copy the Sheet ID from the URL and update it in the **Get row(s) in sheet** node.

### 4. Set your Telegram chat ID

In the **Send a text message** node, replace `chatId` with your own Telegram chat ID. Get it by messaging [@userinfobot](https://t.me/userinfobot) on Telegram.

### 5. Activate the workflow

Toggle the workflow to **Active**. It will now run automatically every day at 9:00 AM.

---

## Customization

| What to change | Where |
|---|---|
| Trigger time | Schedule Trigger node → `triggerAtHour` |
| Planning language or tone | Basic LLM Chain node → edit the prompt |
| Danger zone threshold | Basic LLM Chain node → adjust prompt criteria |
| Add more project columns | Google Sheet + Aggregate node + prompt |
| Telegram recipient | Send a text message node → `chatId` |
| AI model | Google Gemini Chat Model node → `modelName` |

---

## Notes

- The workflow is set to **inactive** by default — activate it manually when ready.
- The Aggregate node is essential — it merges all project rows into a single item so the LLM receives all projects in one prompt rather than processing them one by one.
- Progress values in the sheet should be consistent (e.g. always `45%` or always `0.45`) — the LLM handles both but consistency improves accuracy.
- The prompt includes today's date from the Schedule Trigger's `Readable date` field, which Gemini uses to calculate days remaining automatically.

---

## License

MIT
