# ✈️ AI Trip Planner Chatbot v2

A conversational AI travel assistant that turns a single natural language message into a complete trip package — top places, a validated day-by-day itinerary, and a categorised budget breakdown — with full session memory that lets you modify any detail mid-conversation.

---

## 🆕 What's New in v2

| Feature | v1 | v2 |
|---|---|---|
| `import re` | ❌ Missing — crashed on launch | ✅ Fixed |
| `estimate_budget` output | Raw number string | Full breakdown: accommodation, food, transport, activities, total |
| `generate_itinerary` | Basic, often truncated | Style-aware (budget/luxury/normal hints), validated with one retry |
| Destination extraction | Single fragile regex | 2-pattern cascade + stopword filter |
| Days extraction | Digit only | Digit + word form (`five days`) |
| Memory | 3 fields | 6 fields — destination, days, preference, places, itinerary, budget |
| Modification | None | Detects `"change to 7 days"`, `"switch to luxury"`, `"go to Dubai"` |
| `clear_fn` | Broken state | Properly resets memory + chat history |
| Budget JSON | No fallback | Strips code fences, fallback to regex number extraction |
| Gradio UI | Basic | Memory debug panel, welcome message, tips bar |

---

## 📌 Overview

This project uses **OpenAI GPT-4o-mini** to power three specialised tools and wraps them in a **Gradio** chatbot interface. A regex-based extraction layer parses user intent from natural language, and a Python dictionary maintains trip state across the entire conversation session.

---

## 🏗️ Architecture

```
┌─────────────────────── UI Layer ──────────────────────────┐
│   User Input  ──▶  Gradio Chat UI  ──▶  Chat History      │
└──────────────────────────┬────────────────────────────────┘
                           │
                           ▼
               ┌─────────────────────┐
               │  Trip Planner Agent │
               └──────┬──────────────┘
          ┌───────────┘           └──────────────┐
          ▼                                      ▼
┌──── Extraction Layer ────┐      ┌──── LLM Tool Layer ─────┐
│ extract_destination()    │      │ get_places()             │
│ extract_days()           │      │   └─ GPT-4o-mini         │
│ extract_preference()     │      │ generate_itinerary()     │
│ detect_modification()    │      │   └─ GPT-4o-mini         │
└──────────┬───────────────┘      │ estimate_budget()        │
           │                      │   └─ GPT-4o-mini         │
           ▼                      └────────────┬─────────────┘
   ┌───────────────────────────────────────┐   │
   │  Memory Dict                          │◀──┘
   │  destination · days · preference      │
   │  places · itinerary · budget          │
   └──────────────────┬────────────────────┘
                      │
                      ▼
            validate_itinerary()
            (retry once if days missing)
                      │
                      ▼
            format_response()
            Places · Itinerary · Budget
                      │
                      ▼
            Gradio Chat UI  ◀─── formatted reply
```

### Components

| Component | Function | Technology |
|---|---|---|
| `extract_destination()` | Parses destination from natural language | Python regex — 2-pattern cascade |
| `extract_days()` | Extracts trip duration (digit + word form) | Python regex |
| `extract_preference()` | Detects budget/luxury/normal style | Keyword matching |
| `detect_modification()` | Identifies mid-conversation plan changes | Python regex |
| Memory Dict | Persists all trip state across turns | Python dictionary (session-scoped) |
| `get_places()` | Fetches top 5 tourist spots | GPT-4o-mini |
| `generate_itinerary()` | Builds a style-aware day-by-day plan | GPT-4o-mini |
| `estimate_budget()` | Returns itemised INR cost breakdown | GPT-4o-mini → JSON parse |
| `validate_itinerary()` | Ensures all days present; retries once | String parsing |
| `format_response()` | Assembles the final formatted reply | Python f-strings |
| Gradio UI | Chat interface with memory debug panel | Gradio Blocks |

---

## ✨ Features

- 💬 Natural language input — just describe your trip
- 🗺️ Top 5 tourist spots fetched per destination
- 🗓️ Validated day-by-day itinerary with style-specific activity hints
- 💰 Itemised budget estimate: accommodation, food, transport, activities
- 🧠 Session memory across conversation turns — no need to repeat yourself
- 🔄 Mid-conversation modification: change days, style, or destination on the fly
- 🖥️ Gradio chat UI with memory debug panel and tip bar

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| [OpenAI API](https://platform.openai.com/) — GPT-4o-mini | All three LLM tools |
| [Gradio ≥ 4.0](https://gradio.app/) | Chat interface |
| Python `re` (stdlib) | Regex-based NLP extraction |
| Python `json` (stdlib) | Budget JSON parsing with fallback |

---

## ⚙️ Setup

### 1. Clone the repository
```bash
git clone https://github.com/your-username/ai-trip-planner.git
cd ai-trip-planner
```

### 2. Install dependencies
```bash
pip install openai "gradio>=4.0.0"
```

### 3. Set your OpenAI API key
In the notebook, replace the placeholder in **Cell 3**:
```python
os.environ["OPENAI_API_KEY"] = "your-openai-api-key-here"
```
Or use an environment variable:
```bash
export OPENAI_API_KEY="your-openai-api-key-here"
```

### 4. Run
Open `TripPlanner_v2.ipynb` in Jupyter or Google Colab and run all cells.  
A Gradio link will appear — open it in your browser.

> **Colab users:** The last cell includes `share=True` — you'll get a public URL automatically.

---

## 💬 Usage

Type naturally — the agent figures out the details:

| Input | What's Extracted |
|---|---|
| `Plan a trip to Goa for 4 days` | destination=Goa, days=4 |
| `Budget trip to Paris for 7 days` | destination=Paris, days=7, preference=budget |
| `Luxury vacation in Dubai for 5 days` | destination=Dubai, days=5, preference=luxury |
| `Change to 7 days` | Modifies days, regenerates itinerary |
| `Switch to luxury` | Modifies style, regenerates budget |
| `Go to Switzerland` | New destination, resets plan |
| `Reset` | Clears all memory |

**Sample output:**
```
╔══════════════════════════════════════╗
   ✈️  TRIP PLAN  —  GOA
╚══════════════════════════════════════╝

📍 Destination : Goa
🗓️  Duration    : 4 days
🎒 Travel Style : Budget

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📌 TOP PLACES TO VISIT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  • Baga Beach
  • Dudhsagar Falls
  • Fort Aguada
  • Basilica of Bom Jesus
  • Anjuna Flea Market

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🗓️  DAY-BY-DAY ITINERARY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Day 1: Arrive → Baga Beach → sunset at Fort Aguada
  Day 2: Basilica of Bom Jesus → Old Goa churches
  Day 3: Day trip to Dudhsagar Falls
  Day 4: Anjuna Flea Market → depart

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
💰 BUDGET ESTIMATE (Budget Style)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  🏨 Accommodation : ₹4,000
  🍽️  Food           : ₹3,200
  🚌 Transport      : ₹2,500
  🎟️  Activities      : ₹1,500
  ─────────────────────────────
  💰 TOTAL          : ₹11,200
```

---

## 📁 Project Structure

```
ai-trip-planner/
│
├── TripPlanner_v2.ipynb     # Main notebook (v2 — fully working)
├── TripPlanner.ipynb        # Original v1 (for reference)
└── README.md                # This file
```

---

## ⚠️ Important Notes

- **Never commit your API key.** Use environment variables or `python-dotenv` with a `.env` file.
- Memory is session-scoped — it resets when the Gradio session ends or the kernel restarts.
- Budget estimates are LLM-generated approximations — treat them as ballpark figures.
- Default duration is 3 days if the user doesn't mention a number.

---

## 📄 License

This project is open-source and available under the [MIT License](LICENSE).
