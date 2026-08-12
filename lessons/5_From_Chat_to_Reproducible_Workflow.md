# Agentic AI for Research Workflows: From Chat to Reproducible Workflow

### Learning Objectives

- Build an interactive dashboard from an analysis using an agent.
- Generate a written specification of a conversational workflow.
- Reproduce an analysis from its specification in a fresh conversation.
- Explain why analysis decisions must live in files, not chat transcripts.

### Icons Used in This Notebook
🔔 **Question**: A quick question to help you understand what's going on.<br>
🥊 **Challenge**: Interactive exercise. We'll work through these in the workshop!<br>
💡 **Tip**: How to do something a bit more efficiently or effectively.<br>
⚠️ **Warning:** Heads-up about tricky stuff or common mistakes.<br>

### Sections
1. [Build a Dashboard](#section1)
2. [Write It All Down](#section2)
3. [The Fresh Start Test](#section3)

<a id='section1'></a>

# Build a Dashboard

Research outputs don't have to be static images pasted into slides. Let's turn today's work into something people can interact with. In a new conversation, use the following prompt

```
Build an interactive dashboard in an HTML file from this project's data and model:
- The delay rate by departure hour, with a toggle to compare SFO, OAK, and SJC.
- A dropdown to pick a destination and see the delay profile for that route.
- The overall delay distribution.
Make it clean and readable. Tell me how to open it.
```

Open yours. Look at three or four around the room. Do they look different?

🔔 **Question:** Who would you share this dashboard with in your own research, and what would they be able to do that a PDF figure doesn't let them do?

<a id='section2'></a>

# Write It All Down

Today's analysis exists in two places: your files, and a long chat conversation full of decisions. Chat transcripts are where research decisions go to die.

So, let's extract them:

```
Write a file called WORKFLOW.md that documents this entire analysis in detail, so that someone who has never seen this conversation could reproduce it exactly.

Include: the data source and how it was downloaded, every filtering and cleaning step with exact column definitions, the weather data source and how it was joined, the exact model specification including the train/test split, how the model is evaluated, and what the dashboard shows. Also include a section listing every decision you made in this project that I did not explicitly ask for.
```

## 🥊 Challenge 5: Audit the Spec

Read `WORKFLOW.md`, especially that last section. Look for decisions you didn't know were made. Some places to check:

- What happened to **cancelled and diverted** flights? They have no arrival delay. Were they dropped? Counted as on time?
- How were **missing values** handled, and how many rows did that remove?
- Scheduled departure time was stored as a number like `517`, meaning 5:17 am. Did the time handling deal with that correctly?
- In the weather join: whose **time zone**? And is joining on the actual weather at departure fair, if a real forecast wouldn't know it?

If any of these aren't in `WORKFLOW.md`, ask the agent to investigate and update the file.

⚠️ **Warning:** Every one of these was a research decision, made silently, by default. Defaults are where analyses go wrong — with or without AI.

<a id='section3'></a>

# The Fresh Start Test

Start a **brand-new conversation** in Codex. The agent that did all today's work is gone. It remembers nothing. Only your files remain.

In the new conversation:

```
Read WORKFLOW.md. Build this analysis as a reproducible pipeline of numbered Python scripts that run in order: 01_download.py, 02_clean.py, 03_weather.py, 04_train.py, 05_evaluate.py,
06_dashboard.py.

Keep it simple — plain scripts, no frameworks. Then run the whole pipeline from start to finish and report the final accuracy and baseline.
```

## Did It Reproduce?

Compare the pipeline's accuracy to this afternoon's number.

- **Match?** Congratulations - your analysis survives without you, and without the chat.
- **Don't match?** Even better - you just learned your `WORKFLOW.md` was missing a decision. Find it, fix the spec, run it again. That gap is exactly what a collaborator (or you, in six months) would have hit.

💡 **Tip:** Notice the agent in the fresh conversation still followed your project rules: it read `AGENTS.md`.

🔔 **Question:** In your own research, what currently lives only in your head - or only in a chat window - that belongs in a `WORKFLOW.md`?

# Key Points

- Agents can turn an analysis into an **interactive dashboard**.
- A workflow spec (`WORKFLOW.md`) extracts decisions out of the chat and into a durable, reviewable file.
- Convert your analyses into reproducible scripts using the agent.