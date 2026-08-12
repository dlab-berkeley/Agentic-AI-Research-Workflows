# Agentic AI for Research Workflows: Getting the Data

### Learning Objectives

- Describe the flight delay prediction problem we solve in this workshop.
- Delegate a multi-step data acquisition task to an agent.
- Inspect the steps an agent took to complete a task.

### Icons Used in This Notebook
🔔 **Question**: A quick question to help you understand what's going on.<br>
💡 **Tip**: How to do something a bit more efficiently or effectively.<br>
⚠️ **Warning:** Heads-up about tricky stuff or common mistakes.<br>

### Sections
1. [The Problem: Will My Flight Be Late?](#section1)
2. [The Manual Way](#section2)
3. [The Agentic Way](#section3)
4. [Look at What It Did](#section4)

<a id='section1'></a>

# The Problem: Will My Flight Be Late?

How many times have you stressed about making your connection because your flight is delayed? Today, we're going to build a model that predicts it: **will a flight arrive 15 or more minutes late?**

The U.S. Department of Transportation defines a flight as "delayed" at exactly that threshold, and the government publishes data on every single domestic flight: the **Bureau of Transportation Statistics (BTS) On-Time Performance data**.

The dataset is quite robust - and, like most real research data, a little unwieldy:

- It covers **every domestic flight** by every major U.S. airline, updated monthly, going back decades.
- Each flight has over 100 columns: scheduled and actual times, the delay broken down by cause (weather? security? the airline itself?), cancellations, diversions, even the aircraft's tail number.
- It doesn't explain itself. The columns have names like `CRSDepTime` and `OriginWac`. There's a codebook you have to read to know what anything means, and lookup tables to turn codes into names.

That's a lot of data. We'll keep things manageable by working with one month of departures from the three Bay Area airports: **SFO**, **OAK**, and **SJC**.

<a id='section2'></a>

# The Manual Way

First, try downloading the data by yourself. Think about what steps you need to take to do this. Then, imagine having to preprocess the data. How many steps and lines of code would you need to write in order to explore the columns and subselect the data?

🔔 **Question:** If you did all this by hand today, and a collaborator asked you in three months exactly what you downloaded and how you filtered it - could you answer?

<a id='section3'></a>

# The Agentic Way

Now, delegate the task to the agent. Getting the data is a new task, so start a **new conversation** in your project. Then, copy and paste this prompt:

```
Download one month of BTS On-Time Performance data from:

https://transtats.bts.gov/PREZIP/On_Time_Reporting_Carrier_On_Time_Performance_1987_present_2025_3.zip

Unzip it into a data/ folder. Then filter it to flights departing from SFO, OAK, or SJC, and save the result as data/flights_bayarea.csv. Tell me how many rows the raw file had and how many remain after filtering.
```

The agent will ask for approval as it goes. Read what it proposes before you approve.

⚠️ **Warning:** The raw file is big (about 250 MB unzipped). Give it a minute.

Everyone should end with roughly the same row count. If your number is way off, ask the agent to double-check its filter.

<a id='section4'></a>

# Look at What It Did

Expand the agent's work in the conversation and look at the individual steps: the download command, the unzip, the filtering code it wrote, the row counts.

Compare that with the manual version: clicking around a website leaves no record at all. The agent leaves a **trace of every step** - you can scroll back and see exactly what it did.

Note that there are only two records of what just happened:

- **The conversation in the app.** It shows every command and every decision - but it's not part of your project. It lives in Codex, not in your folder.
- **The files the agent wrote.** Check your `data/` folder. Depending on what your agent did, the code that downloaded and filtered the data might be saved as a script - or it might have been run once and kept nowhere at all.

🔔 **Question:** Look in your project folder. Did your agent leave behind the code it used? Or just the output file?

💡 **Tip:** A trace in a chat is better than no trace - but it's not a reproducible workflow. You can often make sure there's a record of what was done by instructing the agent to write a Python script and then run it.

🔔 **Question:** The agent probably made at least one decision you didn't tell it to make. Can you spot one in its steps?

# Key Points

- Our task: predict whether a flight arrives **15+ minutes late**, using BTS government data.
- An agent can take over multi-step chores when gathering and processing data.
- The agent's steps are **inspectable** in the conversation, and the agent can be prompted to leave reproducible traces of the actions it takes.
