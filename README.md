# MediCare — Patient Follow-Up Agent

An agentic AI prototype that autonomously reviews chronic-disease patient records, flags clinical
risk, and generates a prioritised follow-up plan for a care-coordination team. Built with
**LangGraph** and **Google Gemini**, exposed over a **FastAPI** REST API, and demonstrated
end-to-end in a Jupyter/Colab notebook.

---

## Problem

MediCare Clinic manages 1,000+ chronic-disease patients (Type 2 Diabetes, Hypertension, Heart
Failure, COPD, CKD, Anemia, and others). Patients who miss appointments or whose lab values are
worsening often go unnoticed until their next scheduled visit — leading to avoidable complications
and hospital readmissions. This project builds an AI agent that reviews records, identifies at-risk
patients, and produces prioritised action plans automatically.

---

## How it works

The agent is a **LangGraph `StateGraph`** with two nodes and a conditional edge:

```
        +----------+   tool calls?   +-----------+
START ->|  agent   |----- yes ------>|   tools   |
        | (Gemini) |<----------------| (ToolNode)|
        +----------+                 +-----------+
             | no (done)
             v
            END
```

- The **agent node** (Gemini) reads the conversation and decides the next move.
- The **tools node** runs the tools the agent requests and feeds the results back.
- `tools_condition` loops until the agent stops calling tools.

**The LLM makes every clinical decision.** The tools only return objective evidence — a lab value
compared to its clinical reference range, abnormal vitals, and missed/overdue-visit flags. The
priority (High/Medium/Low), the actions, and the rationale are decided by Gemini, so no decision
logic is hard-coded.

**Tools:** `get_patient` · `compute_risk_indicators` · `list_missed_appointments` · `save_action_plan`

---

## What it covers

| Task | Description |
|---|---|
| 1 · Data Loading & EDA | Load and clean the 100-patient snapshot; cohort stats, diagnosis distribution, missed-appointment cohort |
| 2 · Agent Tools | Four LangChain `@tool`s returning records and objective risk evidence |
| 3 · Agentic Loop | LangGraph `StateGraph` orchestrating Gemini function-calling |
| 4 · Single-Patient Analysis | Deep-dive one patient into a priority + action plan |
| 5 · Missed-Appointment Follow-Up | Triage the missed-appointment cohort into a prioritised worklist |
| + FastAPI REST API | The agent exposed over HTTP endpoints |

---

## Tech stack

- **LangGraph** — agent orchestration (`StateGraph`, `ToolNode`, `tools_condition`)
- **Google Gemini** — LLM reasoning and function calling (via `langchain-google-genai`)
- **FastAPI** — REST API layer
- **pandas / matplotlib** — data analysis and visualisation

---

## Run it

### Google Colab (recommended)
1. Open `MediCare_Followup_Agent_LangGraph.ipynb` in Colab.
2. Get a free Gemini API key: https://aistudio.google.com/apikey
3. Run the setup cell and paste the key when prompted (or add it to Colab Secrets as `GEMINI_API_KEY`).
4. Upload `patient_data.csv` when the upload cell runs.
5. Runtime -> Run all.

### Local
```bash
pip install -r requirements.txt
export GEMINI_API_KEY=your_key_here
jupyter notebook MediCare_Followup_Agent_LangGraph.ipynb
```

---

## API endpoints

| Method & path | Purpose |
|---|---|
| `GET /health` | Status, model, patient count |
| `GET /patients?limit=` | Patient list |
| `GET /patients/{id}/risk` | Objective risk evidence for one patient |
| `POST /agent/analyze/{id}` | Run the agent on one patient and return an action plan |

---

## Repository structure

```
├── MediCare_Followup_Agent_LangGraph.ipynb   # main notebook (Tasks 1-5 + FastAPI)
├── patient_data.csv                          # 100-patient dataset
├── requirements.txt
└── README.md
```

---

## Notes

- Runs top-to-bottom once a Gemini API key is set and the dataset is uploaded.
- `temperature=0` for reproducibility.
- Clinical reference ranges are documented facts used only to surface evidence — the triage
  decision is the LLM's.
- Why LangGraph: it provides durable state, checkpointing, and easy branching / multi-agent
  extension — the right foundation as the agent grows beyond a single linear loop.

---


