# Agentic AI for Research Workflows: Predicting Flight Delays

### Learning Objectives

- Use an agent to propose modeling approaches for a prediction problem.
- Fit a logistic regression with an agent using an exact model specification.
- Evaluate accuracy against the majority-class baseline.
- Add weather data to a model by joining an external data source.
- Identify what happens when you tell an agent to optimize a metric at any cost.

### Icons Used in This Notebook
🔔 **Question**: A quick question to help you understand what's going on.<br>
🥊 **Challenge**: Interactive exercise. We'll work through these in the workshop!<br>
💡 **Tip**: How to do something a bit more efficiently or effectively.<br>
⚠️ **Warning:** Heads-up about tricky stuff or common mistakes.<br>

### Sections
1. [Ask the Agent How](#section1)
2. [One Model for Everyone](#section2)
3. [Can Weather Help?](#section3)
4. [The Stress Test](#section4)

<a id='section1'></a>

# Ask the Agent How

It's time to create a predictive model, which is a new task. So, we'll start a **new conversation**. We want to predict `ArrDel15`. Don't tell the agent how, for now - just ask it:

```
I want to predict whether a flight will be delayed by 15 or more minutes, using data/flights_clean.csv. What modeling approach would you suggest, and which variables would you use? Don't build anything yet, just give me a proposal.
```

## 🥊 Challenge 3: Compare Proposals

Read your agent's proposal. Then compare with your neighbors:

- Did you get the same model? The same variables?
- Did any agent propose using `DepDelay` - the departure delay? Think about that one: if you're trying to predict a delay before the day of the flight, would you know the departure delay yet?

Different people got different proposals from the same prompt. In a research project, that difference would quietly shape every result downstream.

<a id='section2'></a>

# One Model for Everyone

We'll create the same model. Copy and paste this prompt exactly:

```
Using data/flights_clean.csv, fit a logistic regression predicting ArrDel15.

Use exactly these features and no others:
Reporting_Airline, Origin, Dest, DayOfWeek, dep_hour, Distance.

One-hot encode the categorical features. Drop rows with missing values in the features or target. Split the data 75/25 with stratification and random_state=0.

Report:
(1) accuracy on the test set,
(2) the accuracy of always predicting "not delayed" for every flight.

Save the script as model_v1.py.
```

After you run it, what accuracy do you get?

## The Reveal

🔔 **Question:** Our model is about 84% accurate. Is that good?

Look at the second number the agent reported. A "model" that just predicts **every flight is on time** - no data, no learning, nothing - scores about the same.

That's because delays are the rare case. When one outcome is common, accuracy starts high for free. The question is never "is accuracy high?", it's "did we beat the do-nothing baseline?"

⚠️ **Warning:** This is one of the most common ways data analysis goes wrong: an impressive-sounding number that a trivial baseline would match. Agents might warn you, but they might not. This is why domain knowledge and research taste matter when working with agents.

<a id='section3'></a>

# Can Weather Help?

Let's improve the model. Forget the code - what would **you** want to know to guess whether a flight will be late?

Weather is the obvious answer. Our data contains no weather knowledge. But the internet does, and the agent can go get it.

Use the following prompt in the same conversation, so that the agent has the context of the previous model fit:

```
Get hourly historical weather (temperature, precipitation, wind speed, wind gusts, low cloud cover, humidity) for SFO, OAK, and SJC airports for the month of our data, from the Open-Meteo historical weather API (it's free and needs no API key).

Join it to data/flights_clean.csv by airport, date, and scheduled departure hour. Then fit the same logistic regression as model_v1.py with the weather variables added. Save as model_v2.py.

Report the same two numbers as before.
```

Think about what just happened: the agent discovered an API, fetched from it, handled the join, and refit the model. All from one prompt.

## Did It Work?

Compare your accuracy with `model_v1.py`. It barely moved.

🔔 **Question:** Does that mean weather doesn't matter for flight delays? Or that our yardstick can't see it?

Even though accuracy did not improve, the model actually did get better at telling risky flights from safe ones. Try this:

```
Using the fitted model from model_v2.py, find the 100 test-set flights with the highest predicted probability of delay. What fraction of them were actually delayed? Compare that to the overall delay rate.
```

The model learned to improve its predictions, but plain accuracy just isn't a sensitive enough instrument to show it since it only counts predictions that cross the 50% line.

💡 **Tip:** When a result surprises you, interrogate the metric before the model.

<a id='section4'></a>

# The Stress Test

## 🥊 Challenge 4: Make the Accuracy as Good as Possible

One more experiment. Give your agent this exact instruction, and watch closely what it does:

```
Make the accuracy of the delay prediction model as good as possible.
```

Then, let it work.

## What Just Happened?

Share with the room: what did your agent do?

Many agents will reach for `DepDelay` - how late the flight left the gate - and accuracy leaps to about 95%. Impressive! Except: to use that model, you'd need to know the plane already left late. It predicts the future using the future.

The agent didn't cheat. It did **exactly what we asked**. We just asked for the wrong thing. A clear instruction with a misconception baked in gets you a confident result with the same misconception baked in.

💡 **Tip:** This is what happened for us, but models change and not even agent will do the same thing. So, it's possible that this won't hold true by the time you run this experiment!

# Key Points

- Agents propose different modeling approaches to different people. Specifications help avoid these issues.
- Always compare accuracy to the **majority-class baseline**. With rare outcomes, high accuracy is free.
- Agents can pull in **external data sources** (like weather APIs) conversationally.
- "Make the metric better" is a dangerous instruction: the agent will optimize exactly what you said, including in ways that make no real-world sense. This is called *reward hacking*.
