# AI Job Application Assistant

**Milestone I: Multi-Agent Workflow and System Design**  
**Author:** Otis Anthony Service

---

## 1. System Overview & Coordination Mechanism

The **AI Job Application Assistant** is an agentic system designed to automate, personalize, and optimize the highly repetitive job application process. Unlike traditional single-prompt LLM interactions, this system decomposes the workflow into four specialized, collaborative agents to ensure deeper reasoning, tool usage, and iterative quality control.

### Coordination Mechanism
The architecture operates via a **hybrid coordination model** combining a fixed pipeline (A &rarr; B &rarr; C) with a dynamic evaluation loop (generate&ndash;critique&ndash;revise).

```
[User Input] ➔ [Resumé Analyzer] ➔ [Job Matcher + Search API] 
                                            │
                                            ▼
[Final Output] ◀ [Critic/Reviewer] ◀ [Cover Letter Generator]
                       │                     ▲
                       └─ (Needs Revision) ──┘
```

1. **Sequential Extraction & Matching:** The user uploads a raw resumé. The **Resumé Analyzer** parses and structures the profile. This structured profile is passed sequentially to the **Job Matcher**, which uses specialized search tools to fetch external job listing endpoints.
2. **Dynamic Revision Loop:** Once a target job is selected, the **Cover Letter Generator** creates an initial draft. The **Critic Agent** acts as a strict quality gate. If the draft fails to meet the target quality criteria, the Critic flags the state as `Needs Revision` and routes control back to the Generator alongside explicit feedback. If approved, the pipeline terminates and delivers the final asset.

---

## 2. Agent Specifications & I/O Formats

### i. Resumé Analyzer
* **Role:** Parse raw user experience, identify key technical and soft skills, and highlight potential structural gaps or missing keywords.
* **Input Format:** Plain Text extracted from user-provided PDF/Text files.
* **Output Format:** Structured JSON Object:
  ```json
  {
    "extracted_skills": ["Python", "SQL", "Information Security"],
    "strengths": ["Strong technical foundation in database management"],
    "gaps_identified": ["Needs more metrics-driven impact statements"],
    "suggested_keywords": ["NIST 2.0", "Access Control"]
  }
  ```

### ii. Job Matcher (Tool-Using Agent)
* **Role:** Query external web search engines or job APIs using parsed skills to find and rank relevant open positions.
* **Tools Used:** Web Search API (e.g., SerpAPI / Bing Search API).
* **Input Format:** JSON Object (Output from Resumé Analyzer) + User Preferences String.
* **Output Format:** Ranked JSON List of job matches:
  ```json
  [
    {
      "rank": 1,
      "title": "Cybersecurity Analyst",
      "company": "SecureTech",
      "description": "Requires foundational Python and database security knowledge...",
      "source_url": "https://jobs.example.com/101",
      "match_score": 94
    }
  ]
  ```

### iii. Cover Letter Generator
* **Role:** Synthesize the applicant's analyzed strengths with the target job description to write a professional, highly personalized cover letter.
* **Input Format:** Selected JSON Object (Job Details) + JSON Object (Analyzed Resumé Profile).
* **Output Format:** Full-text String containing the drafted cover letter.

### iv. Critic / Reviewer Agent
* **Role:** Act as a quality validator to eliminate hallucinations, enforce formal business tone, and verify alignment between the candidate's profile and the job description.
* **Input Format:** String (Drafted Cover Letter) + JSON Object (Target Job Description).
* **Output Format:** Workflow State JSON Object:
  ```json
  {
    "status": "Needs Revision", 
    "evaluation_scores": {
      "professionalism": 9, 
      "alignment": 6, 
      "completeness": 8
    },
    "constructive_feedback": "The cover letter fails to reference the database security requirements listed in the job post. Inject Python/SQL experience explicitly."
  }
  ```

---

## 3. Communication & State Management

To support an interactive workflow where user input is required at intermediate steps (e.g., selecting a job from the matched list), communication is handled via a **Centralized State Orchestrator** pattern built in pure Python using LangChain components.

* **State Object:** A single persistent Python dictionary stores the session variables (`resume_data`, `matched_jobs`, `selected_job`, `current_draft`, `critic_feedback`, and `iteration_count`).
* **Message Passing:** Agents remain completely decoupled. They do not call each other directly. Instead, the central Python Orchestrator reads the current state, invokes the appropriate agent wrapper, updates the state with the agent's output, and determines the next logical transition based on explicit routing rules.

---

## 4. Error Handling & Quality Mitigation Strategies

Multi-agent systems often suffer from compounding errors and infinite looping. This system implements strict runtime programmatic guardrails:

* **Loop Halting Condition (Infinite Loop Mitigation):** To prevent an expensive or infinite execution loop between the Cover Letter Generator and the Critic/Reviewer, the orchestrator increments an `iteration_count` state variable. If the Critic flags a status of `Needs Revision` a third consecutive time, the orchestrator breaks the loop, records a warning log, and pushes the highest-scoring draft to the user alongside a disclaimer recommending manual refinement.
* **Low-Quality Content Remediation:** If a generation pass yields an alignment score below the threshold (e.g., < 7/10), the orchestrator passes the precise content of `constructive_feedback` directly into the system prompt of the Cover Letter Generator for its next run, compelling self-correction based on clear rubrics.
* **API Failure & Rate Limit Resiliency:** The Job Matcher relies on external web scraping and APIs, which are subject to failure or strict rate caps. The system uses Python `try-except` blocks wrapping all API requests. If an API request fails, the system catches the exception, logs it, and gracefully falls back to a locally cached mock database of generic listings, prompting the user via the interface to copy-paste their desired target job description manually so the pipeline remains uninterrupted.

---

## Setup & Run

Quick steps to prepare and run the project locally (PowerShell):

```powershell
# Create and activate virtual environment (if not already created)
python -m venv .venv
.\.venv\Scripts\Activate.ps1

# Install dependencies
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install -r requirements.txt

# Run the example application
.\.venv\Scripts\python.exe app.py

# Or run with Streamlit UI
.\.venv\Scripts\python.exe -m streamlit run app.py
```

Copy `.env.example` to `.env` and populate any required API keys before using the Job Matcher or LLM integrations.

