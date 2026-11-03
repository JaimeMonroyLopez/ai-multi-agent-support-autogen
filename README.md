# Multi-Agent Support Team with AutoGen (100% Local AI)

A team of **3 AI agents** that handles customer support tickets together: one sorts the ticket, one researches the solution, and one writes the reply. It runs **entirely on your computer** with Ollama, so there are no API costs and the data stays private.

## What does it do?

```
Customer ticket: "I'm getting API error 401..."
        │
        ▼
 TriageSpecialist    →  Category: Technical · Priority: Medium
        │
        ▼
 ResearchSpecialist  →  Root cause + step-by-step solution + docs
        │
        ▼
 ResponseSpecialist  →  Friendly, professional reply to the customer
```

## How does it work?

1. **Each agent has one job**, defined in its system message (like a job description).
2. **Sequential pipeline:** the ticket goes Triage → Research → Response. Each agent receives the previous agent's work.
3. **Group chat:** for a ticket with several problems, the agents work in the same conversation, coordinated by a `GroupChatManager`.
4. **Local model:** AutoGen talks to Ollama (`llama3.2`) through an OpenAI-compatible URL (`http://localhost:11434/v1`).

## Key concepts

| Concept | In simple words |
|---|---|
| **Multi-agent system** | Several AIs, each with one specialty, working as a team. |
| **AutoGen** | A framework for creating agents and making them talk to each other. |
| **Ollama** | Runs AI models on your own computer, with no internet and no cost per use. |
| **Group chat** | A shared conversation where a "manager" decides who speaks next. |

## What I found (and fixed)

Running the original lab code exposed **two bugs**. The notebook keeps the original code and adds a fixed version next to it:

| | Original code | Fixed version |
|---|---|---|
| Complete tickets | **0 / 3** (Research answer was empty) | **3 / 3** |
| Group chat (4 issues) | 426 s, 48 repetitive messages | **32 s**, 4 messages in order |
| Avg time per ticket | — | **18 s** on an RTX 4090 |

1. **Empty answers.** The previous agent's output was sent with the role `assistant`, so the model "thought" it had already answered. **Fix:** send it inside a `user` message.
2. **The group chat looped.** A small model (3B) had to choose who speaks next and kept choosing badly. **Fix:** a fixed order (`round_robin`) and a length limit per reply.

The lab's business numbers didn't add up either: it claims **$65K** in savings, but its own formula gives **$639K**. The notebook recalculates them using the measured times and a more realistic human-review step.

## How to run it

```bash
# 1. Install Ollama (https://ollama.com) and download the model
ollama pull llama3.2

# 2. Install AutoGen (needs Python 3.10–3.13)
pip install -r requirements.txt

# 3. Open the notebook
jupyter notebook supportflow.ipynb
```

## Files

- `supportflow.ipynb`: the lab, with real outputs (original code + fixes)
- `supportflow.html`: the same notebook, viewable in a browser
- `supportflow_results.json`: every agent's output from the run

## What I learned

- **Run the code and measure; don't trust the claims.** The original reported "100% success", but in reality 0 of 3 tickets were complete.
- **Small local models need more structure:** fixed turn order, short replies, clear messages.
- **Message roles matter** (`user` vs `assistant`). A wrong role can silently break an agent.
- **Local AI is free and private**, but the model can still invent details (like URLs), so a real knowledge base would be the next step.
