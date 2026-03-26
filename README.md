# 🕵️‍♂️ AI Lead Gen & Prospecting Engine (n8n Workflow)

This is a high-performance **autonomous AI prospecting engine** built on n8n. It doesn't just scrape names; it performs deep, real-time business research to find actionable gaps and create personalized outreach.

---

## 💎 Why This Workflow?
Most lead generation tools provide cold, dead data. This engine provides **Intelligence**:
*   **Live Web Research**: Powered by Tavily API to bypass LLM training data cut-offs.
*   **Automation Gap Analysis**: Identifies 3 specific **n8n automation opportunities** for the target business (e.g., Supply Chain sync, CRM automation, Financial reporting).
*   **Hyper-Personalized Ice-breakers**: Generates high-conversion opening lines based on the company’s latest news (e.g., "I saw your recent $1B factory expansion...").
*   **Contact Hunting**: Automatically extracts Websites, LinkedIn profiles, and Phone numbers.

---

## 🛠️ System Architecture
The workflow follows a robust **linear logic** for 100% reliability:
1.  **Telegram Trigger**: Input a business name (e.g., "Siemens" or "Wayco Valencia").
2.  **AI Planner**: Formulates a multi-step search strategy.
3.  **HTTP Researcher**: Scrapes the live web for the latest data & contacts.
4.  **Deep Analyst**: Processes the data to find "pain points" and automation needs.
5.  **Data Formatter**: A custom AI layer that cleans the data before storage.
6.  **Google Sheets Integration**: Automatically appends clean, structured leads to your CRM.
7.  **Telegram Response**: Delivers a structured executive summary to your phone.

---

## 🚀 How to Setup
1.  **Import the JSON**: Download the `.json` file from this repo and import it into your n8n instance.
2.  **Connect Credentials**:
    *   OpenAI (GPT-4o)
    *   Tavily API (for Web Search)
    *   Telegram Bot API
    *   Google Sheets (OAuth2)
3.  **Create your Sheet**: Create a Google Sheet with columns: `Date`, `Company`, `Industry`, `Automation Gaps`, `Ice-breaker`, `Website`, `Contacts`.
4.  **Execute**: Send a company name to your Telegram bot and watch the magic happen!

---

## 👔 Business Applications
*   **AI Automation Agencies (AAA)**: Scale your outbound sales with zero manual research.
*   **Sales Teams**: Prepare for meetings with a full business audit in 30 seconds.
*   **Consultants**: Identify high-value upsell opportunities before the first call.

---
**Developed by Yevhenii — AI Automation Developer & Systems Architect**
