# 🦷 Dental Appointment Management System

A **multi-agent conversational AI system** for managing dental appointments using **LangGraph, LangChain, and Llama-3 (Groq)**. This project demonstrates how multiple specialized AI agents collaborate to handle real-world tasks like booking, cancelling, and rescheduling appointments through natural language.

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python) ![LangChain](https://img.shields.io/badge/LangChain-🦜-green) ![LangGraph](https://img.shields.io/badge/LangGraph-multi--agent-orange) ![Llama-3](https://img.shields.io/badge/Llama--3-Groq-purple)

---

## Overview

A chat-based AI assistant that helps patients and clinic staff manage dental appointments through natural language.

Users can:
- Check **available appointment slots** and doctor schedules
- **Book new appointments** with preferred doctors
- **Cancel existing appointments**
- **Reschedule appointments** to different time slots
- View **patient appointment history**

---

## Architecture

The system follows a **Supervisor → Specialist Agents** pattern where a central coordinator classifies intent and routes each request to the appropriate agent.

```
                    ┌──────────────┐
                    │   Supervisor │ ← Intent classification & routing
                    └──────┬───────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
   ┌─────────────┐ ┌─────────────┐ ┌───────────────┐
   │ Info Agent  │ │   Booking   │ │ Cancellation  │
   │             │ │    Agent    │ │    Agent      │
   └─────────────┘ └─────────────┘ └───────────────┘
          │
          ▼
   ┌───────────────┐
   │  Rescheduling │
   │    Agent      │
   └───────────────┘
```

### Agent Responsibilities

| Agent | Responsibility |
|-------|----------------|
| **Supervisor** | Classifies intent (`get_info`, `book`, `cancel`, `reschedule`, `end`) and routes to the correct agent |
| **Info Agent** | Queries available slots, doctor schedules, and patient appointment history |
| **Booking Agent** | Collects booking details, checks availability, and creates appointments |
| **Cancellation Agent** | Handles cancellation requests with confirmation |
| **Rescheduling Agent** | Moves appointments to new time slots after verifying availability |

---

## Tech Stack

| Component | Technology |
|-----------|------------|
| Agent Orchestration | LangGraph |
| LLM Framework | LangChain |
| Language Model | Llama-3 (Groq API) |
| Backend | Python |
| Data Storage | CSV (Pandas) |
| Data Validation | Pydantic |

---

## Project Structure

```
Dental-Appointment-System/
│
├── main.py                          # Entry point — interactive CLI
├── doctor_availability.csv          # Appointment data store
├── requirements.txt
├── README.md
│
└── dental_agent/
    ├── agent.py                     # Main agent definition & tools
    ├── config/
    │   └── settings.py              # Configuration & environment vars
    ├── models/
    │   └── state.py                 # State schema definitions
    ├── tools/
    │   ├── csv_reader.py            # Read operations (query tools)
    │   └── csv_writer.py            # Write operations (mutation tools)
    ├── agents/
    │   ├── supervisor.py
    │   ├── info_agent.py
    │   ├── booking_agent.py
    │   ├── cancellation_agent.py
    │   └── rescheduling_agent.py
    └── workflows/
        └── graph.py                 # LangGraph workflow definition
```

---

## Installation

### Prerequisites

- Python 3.10+
- A [Groq API key](https://console.groq.com/keys)

### Steps

1. **Clone the repository:**
   ```bash
   git clone https://github.com/YOUR_USERNAME/ai-dental-appointment-agent.git
   cd ai-dental-appointment-agent
   ```

2. **Create and activate a virtual environment:**
   ```bash
   python -m venv venv
   source venv/bin/activate        # macOS/Linux
   # venv\Scripts\activate         # Windows
   ```

3. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

4. **Set up environment variables** — create a `.env` file in the project root:
   ```env
   GROQ_API_KEY=your_api_key_here
   MODEL_NAME=llama-3.3-70b-versatile
   TEMPERATURE=0
   ```

---

## Usage

Start the interactive CLI:

```bash
python main.py
```

### Example Interactions

**Check available slots:**
```
You: Show available slots for an orthodontist
Agent: Here are the available orthodontist appointments:
  1. 5/10/2026 9:00  — Dr. Emily Johnson
  2. 5/10/2026 10:00 — Dr. Emily Johnson
  3. 5/12/2026 14:00 — Dr. Emily Johnson
```

**Book an appointment:**
```
You: Book patient 1000082 with Emily Johnson on 5/10/2026 9:00
Agent: The slot is available! Appointment confirmed:
  - Patient ID: 1000082
  - Doctor: Dr. Emily Johnson
  - Date/Time: 5/10/2026 9:00
  - Specialization: Orthodontist
```

**Check patient appointments:**
```
You: What appointments does patient 1000048 have?
Agent: Patient 1000048 has the following appointments:
  1. 5/8/2026 9:00 — Dr. John Doe (General Dentist)
```

**Cancel an appointment:**
```
You: Cancel appointment for patient 1000082 at 5/10/2026 9:00
Agent: Appointment for patient 1000082 on 5/10/2026 at 9:00 has been cancelled.
```

**Reschedule an appointment:**
```
You: Reschedule patient 1000082 from 5/10/2026 9:00 to 5/12/2026 10:00
Agent: New slot is available! Appointment rescheduled:
  - Patient ID: 1000082
  - New Date/Time: 5/12/2026 10:00
  - Doctor: Dr. Emily Johnson
```

---

## Data Model

Appointments are stored in `doctor_availability.csv`:

| Field | Description |
|-------|-------------|
| `date_slot` | Appointment date and time (`M/D/YYYY H:MM`) |
| `specialization` | Type of dental specialist |
| `doctor_name` | Name of the dentist |
| `is_available` | `true` if the slot is open, `false` if booked |
| `patient_to_attend` | Patient ID if booked; empty if available |

### Supported Specializations

- General Dentist · Oral Surgeon · Orthodontist · Cosmetic Dentist
- Prosthodontist · Pediatric Dentist · Emergency Dentist

---

## Key Concepts Demonstrated

### Multi-Agent Systems
Multiple AI agents collaborate to complete tasks, each with a focused role.

### LLM Tool Calling
Agents interact with external tools to query and modify structured data.

### State Management
LangGraph maintains conversation state across all agents — including message history, routing decisions, collected parameters, and tool results.

### Intent Classification
The Supervisor uses structured output parsing to return a JSON object with `intent`, `next_agent`, and `reasoning`, routing each message appropriately.

### Principle of Least Privilege
Each agent only has access to the tools it needs — the Info Agent can query slots but cannot book them.

### Data Abstraction Layer
CSV tools abstract the data layer, making it straightforward to swap in a real database without touching agent code.

---

## License

This project is intended for educational and demonstration purposes.
