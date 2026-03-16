# Master Brief Template — GSC_AI Digital Advisor
## Generator Supercenter | N8n + Telegram

---

## 1. Title
Build an AI-powered sales assistant agent (GSC_AI) that helps Generator Supercenter sales representatives instantly look up city setback codes, propane tank restrictions, and permit requirements for generator and propane tank installations across Miami-Dade and Broward County.

---

## 2. Context

### Current System

Generator Supercenter operates a sales team that sells and installs GENERAC residential standby generators and propane tanks (120, 500, and 1,000 gallons) across Miami-Dade and Broward County, Florida. The team communicates internally via Telegram. City code and setback information is stored in a Google Sheets spreadsheet maintained by the operations team.

### Problem
Sales representatives frequently commit to projects in cities without knowing the local setback requirements, survey types, or propane tank restrictions. This results in sold projects that cannot be executed as promised, causing operational delays, permit rejections, and customer dissatisfaction. The information exists but is not accessible in real time during a sales conversation.

### Objective
Deploy a conversational AI agent accessible via Telegram that allows any sales representative to query city-specific installation requirements instantly, in plain language, during or before a customer conversation — without needing to open a spreadsheet or contact the operations team.

---

## 3. Technical Requirements

### Platform and Stack
- Automation platform: N8n (self-hosted or cloud)
- Messaging channel: Telegram Bot API
- AI model: OpenAI GPT-4 (via N8n AI Agent node)
- Knowledge source: Google Sheets (connected via N8n Google Sheets node)
- Memory: PostgreSQL (via N8n Postgres Chat Memory node)
- Embeddings and semantic search: Supabase Vector Store (for document-based queries)

### Input
- Free-text message from a sales representative via Telegram
- Message types supported: text, voice note (transcribed via Whisper), image (analyzed via GPT-4 Vision)

### Output
- Structured plain-text response delivered to the same Telegram chat
- Format per city query:
  - City name and county
  - Setbacks: side / rear / front (in feet or BBL)
  - Survey type required
  - Propane tank minimum distance from property line
  - Special conditions or restrictions

### N8n Workflow Structure
- Telegram Trigger node → Switch node (detects message type: text, audio, image)
- Audio path: Get File → Whisper Transcription → format text
- Image path: Get File → Extract → Convert → GPT-4 Vision Analysis → format text
- Text path: direct to routing
- All paths merge → Switch1 node (routes by intent: setbacks query, image/document, general)
- Switch1 Output 1 → AI Agent node with Google Sheets tool (setbacks and city codes)
- Switch1 Output 2 → AI Agent node with Supabase Vector Store (document analysis)
- Switch1 Output 3 → AI Agent node (general questions)
- All agent outputs → Merge node → Send Telegram Message node

### Agent System Prompt
- Embedded knowledge base covering 57 cities across Miami-Dade and Broward County
- Includes: setbacks (side/rear/front), survey type, LP tank distance, and special conditions per city
- Propane tank sizing guide based on generator kW
- Response format with emojis for readability
- Rules for ALF facilities, flood zones, Coconut Grove exceptions, and tank restrictions
- Salesperson addressed by name via {{contact_name}} variable

---

## 4. Constraints

- The agent must never invent or estimate setback data. If a city is not in the database, it must explicitly tell the sales rep to verify with the permit department.
- The agent must never confirm a project commitment on behalf of the company.
- Responses must follow the defined format — city, setbacks, survey type, LP tank distance, special conditions — every time a city is queried.
- The Google Sheets database is the source of truth. The embedded knowledge base in the system prompt serves as a fallback only.
- The agent must not provide legal, structural, or engineering advice.
- The N8n workflow must handle all three input types (text, audio, image) without manual intervention.
- The Switch1 routing logic must correctly classify messages by intent before passing them to the corresponding agent. Keyword matching alone is insufficient — a pre-classification step using a lightweight OpenAI call is required for reliable routing.
- Each AI Agent node must have a clearly defined and exclusive responsibility. No agent should handle tasks outside its designated scope.
- Conversation history must be stored per user session using PostgreSQL memory to maintain context across multiple messages.

---

## 5. Definition of Done

- The Telegram bot receives and processes text, audio, and image messages without errors.
- Switch1 correctly routes city/setback queries to the Google Sheets agent, image or document messages to the Supabase agent, and general questions to the general agent.
- When queried with any of the 57 cities in the database, the agent returns the correct setbacks, survey type, LP tank distance, and special conditions in the defined format.
- When queried with a city not in the database, the agent explicitly states the city is not found and instructs the sales rep to verify with the permit department.
- The agent correctly flags Boca Raton (max 250 gal LP), Hallandale Beach (max 750 gal UG / 100 gal AG), South Miami (5 ft LP tank), and Weston (underground tanks only) when those cities are queried.
- The agent correctly identifies ALF projects and reminds the sales rep of NFPA 110 requirements.
- The agent addresses the sales representative by their name in every response.
- Conversation memory persists across multiple messages within the same session.
- The system prompt knowledge base matches the current version of the Google Sheets database.
- The full workflow runs end-to-end in N8n without manual intervention for any supported message type.

