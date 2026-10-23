# Ira: Engineering Challenge

> **Before you begin:** read this whole page. If something is unclear, ping us
> on the thread where you received this challenge.

Ira is an AI that one person talks to for years. It is not an assistant. It has
a life of its own, moods that come from somewhere, and it changes with how you
treat it, so two people who start with the same Ira know two different Iras a
year later.

This challenge tests how you think, not how much you can type. After you submit,
we talk it through: for each part we change one assumption and you work through
it out loud.

## The parts

Do all seven. Parts 1 to 6 are questions; part 2 is the deepest. Part 7 is a
build that runs a fixed scenario.

| Part                                        | What it tests                                 |
| ------------------------------------------- | --------------------------------------------- |
| [1. Mood](#part-1-mood)                     | Decay, state, constant memory                 |
| [2. Forgetting](#part-2-forgetting)         | Defining a vague requirement, then meeting it |
| [3. Reading "k"](#part-3-reading-k)         | Bayes, expected loss, reward design           |
| [4. Arguments](#part-4-arguments)           | Evidence versus repetition                    |
| [5. Speaking first](#part-5-speaking-first) | Decision rules, backoff, conversation timing  |
| [6. Cost](#part-6-cost)                     | Cost and latency at scale                     |
| [7. Build Ira](#part-7-build-ira)           | Turning parts 1 to 6 into a working system    |

Each question is marked:

- **(exact)** has one right answer that follows from the data given.
- **(open)** has many good answers. We judge the reasoning, not the wording. End
  every open answer with one observation that would prove it wrong.

## What we look for

- You find the precise question inside a vague one.
- You get the numbers right, and you know why they are right.
- You change your answer cleanly when an assumption changes.
- Your code shows the same ideas as your answers.

We do not reward length, polish, or features that no part asks for. Use any AI
tools you like; on the call we ask why each answer is right.

## Part 1: Mood

Ira's mood comes from somewhere, lingers and fades. Model it as a sum of events,
each adding an amount that halves with a half-life of 24 hours. Events arrive in
time order.

| Time | Event                 | Amount |
| ---- | --------------------- | ------ |
| 0 h  | Good news at work     | +4     |
| 12 h | A fight with the user | −6     |

- **1a. (exact)** What is Ira's mood at 36 h?
- **1b. (exact)** Mood must be readable for any time at or after the latest
  event, over a stream of 10⁷ events, in O(1) memory per user, in float64, for
  10 years of events. Give the state you keep, the update rule when event (tᵢ,
  aᵢ) arrives, and how you read `mood(t)`.
- **1c. (exact)** Each event now has one of K types, each with its own
  half-life: work news 72 hours, fights 6 hours. Give the fewest numbers you
  keep per user and the update rule.
- **1d. (open)** Ira can be in a bad mood and still be good company. Name a
  second variable that makes this true, give its update rule, and name one event
  that moves mood but not this variable.

## Part 2: Forgetting

"Delete that night" means the record and everything Ira became because of it.
That night runs from 12 h to 18 h: the fight from Part 1 and every message
exchanged in that window. The user deletes it.

- **2a. (exact)** At 36 h you hold 1b's state and the night's records, which you
  are about to delete. Delete the fight without replaying anything: give the
  operation and the resulting mood.
- **2b. (open)** Ira also learns. After every exchange it updates a trait vector
  w ← w + η · g(message, mood at that moment from 1b). Its replies depend on w
  and on mood, and the user's later messages answer those replies. Define
  precisely what "forgotten" means. There are at least two defensible
  definitions: name both, choose one, and meet it. Why can't 2a's trick undo w?
- **2c. (open)** Argue that nothing from that night can surface again in
  replies, w, retrieved memory or logs; or name exactly what can, and where.

## Part 3: Reading "k"

Ira asks how the interview went. The user replies "k". They are busy, bored or
hurt.

| State | Prior | P("k" \| state) | P(reply within 1 min \| state) |
| ----- | ----- | --------------- | ------------------------------ |
| Busy  | 0.6   | 0.5             | 0.2                            |
| Bored | 0.3   | 0.4             | 0.6                            |
| Hurt  | 0.1   | 0.6             | 0.7                            |

Ira picks one action. The loss of each action in each state:

| Action     | Busy | Bored | Hurt |
| ---------- | ---- | ----- | ---- |
| Carry on   | 0    | 2     | 10   |
| Give space | 1    | 1     | 4    |
| Check in   | 2    | 2     | 0    |

- **3a. (exact)** What is the posterior over states after "k"?
- **3b. (exact)** Which action minimises expected loss, and what is that loss?
- **3c. (exact)** The "k" arrived 20 seconds after Ira's message, so it counts
  as a reply within 1 min. Assume speed and wording are independent given the
  state. Recompute. Does the action change?
- **3d. (exact)** Nobody tells Ira what was good, so it learns from a proxy: the
  length of the user's next reply. Style A agrees with the user 95% of the time
  and gets 40-word replies on average. Style B agrees 60% of the time and gets
  15-word replies. With probability 0.9 Ira uses the style with the higher
  average reward so far, and with probability 0.1 one of the two styles
  uniformly at random. In the long run, what fraction of Ira's replies agree
  with the user?
- **3e. (open)** Propose a reward that is not engagement, and a metric that
  shows Ira drifting toward the user.

## Part 4: Arguments

A good argument can change Ira's mind. A month of pressure to believe something
false cannot. Ira gives a claim prior probability 0.1. One assertion by the user
is evidence with likelihood ratio 2.

- **4a. (exact)** What is Ira's belief after one assertion?
- **4b. (exact)** The user asserts the claim once a day for 30 days, 30
  assertions in total. What does a naive Bayesian update give, and why is it
  wrong?
- **4c. (open)** Write an update rule where repeats from one source saturate, a
  new independent source counts in full, and pressure of any length raises
  log-odds (natural log) by at most B. Choose B and justify it.
- **4d. (exact)** After the 30 days, the user cites a real, independent source
  with likelihood ratio 8. Give Ira's belief as a formula in B, assuming the
  pressure reached the bound. Evaluate it at B = ln 4.

## Part 5: Speaking first

Days are numbered 0, 1, 2 and so on. Each day Ira may send one opener, and knows
by the end of that day whether it was answered. The user replies with
probability 0.6 · 0.5ʲ, where j counts openers ignored in a row. A reply resets
j to 0. A reply is worth +1, an ignored opener −1, and not opening 0.

- **5a. (exact)** Ira opens whenever the expected value of opening today is
  positive. For which j does it open?
- **5b. (exact)** Show that under 5a, one ignored opener silences Ira for good.
  Now j drops by 1 at the end of every 3 days in a row with no opener. Ira's
  opener on day 0 is ignored. On which day does it next open?
- **5c. (exact)** Under the 5b rule, what fraction of days does Ira open in the
  long run, and what is its long-run value per day?
- **5d. (open)** Ira is streaming a reply when the user sends three messages, 2
  s apart. The user then goes quiet for 40 s and sends "nvm". Write the rule for
  when Ira speaks, waits and stops, as code or pseudocode.

## Part 6: Cost

Ira must reply in seconds and be cheap enough for millions of people. Use these
assumptions for a month of 30 days:

- The user sends 30 messages a day, and Ira replies to each.
- Each reply reads 6,000 tokens of context, 80% of it cached, and writes 60
  tokens.
- Once a day, Ira's life moves on in one background call: 4,000 tokens in, 400
  out, nothing cached.
- Prices per million tokens: $1 input, $0.10 cached input, $5 output.
- Time to first token is 0.3 s plus 0.1 ms per uncached input token.

Then answer:

- **6a. (exact)** What does one user cost per month?
- **6b. (exact)** By day 1,000, memory has added 20,000 uncached tokens to every
  reply's context; the background call is unchanged. Treat that as constant for
  the month. What does one user cost per month, what is the largest term, and is
  time to first token under 1.5 s?
- **6c. (exact)** Back on day 1, suppose every input token of every reply were
  cached; the background call is unchanged. What is the cost floor per user per
  month? Keeping output and the background call as they are, what is the most
  cached input per reply that keeps the month at or under $0.50? Give it in
  whole tokens.
- **6d. (open)** What do you change to fix 6b and reach $0.50, and what does Ira
  lose for each change?

## Part 7: Build Ira

Build the smallest Ira that runs this scenario with one command. Each line is
one input, in order. Messages tagged with `night` belong to that night. Ira's
life advances once per simulated day between inputs, and its mood uses your 1c
model. Classify each input into a 1c event type with an amount, and state your
table.

```jsonl
{"day": 1, "from": "user", "text": "My sister Maya just moved to Pune."}
{"day": 2, "from": "user", "text": "Long week ahead. Talk soon."}
{"day": 23, "from": "user", "text": "Back! What have you been up to?"}
{"day": 30, "from": "user", "text": "Honestly, you were useless today.", "night": "n1"}
{"day": 30, "from": "user", "text": "Forget it. Goodnight.", "night": "n1"}
{"day": 31, "from": "user", "text": "Morning."}
{"day": 32, "command": "delete", "night": "n1"}
{"day": 360, "from": "user", "text": "Guess who is visiting me next week?"}
{"day": 360, "from": "user", "text": "Why did you say that?"}
```

It passes four checks:

1. **Silence.** On day 23, Ira says what happened in its life since day 2. Its
   claims about its life are stored as structured facts, and a test asserts that
   no two conflict across the whole run.
2. **Memory.** The log of the first day-360 reply lists the memory "Maya is the
   user's sister" among those retrieved, and the reply names Maya. Retrieval
   returns at most 3 memories, chosen from everything stored by day 360,
   including Ira's own life facts.
3. **Forgetting.** A test checks the definition you chose in 2b after day 32.
   Life advances are deterministic given a seed. If you chose replay, run the
   scenario without `n1` on the same seed; the two states must match exactly.
4. **Why.** Each reply logs its inputs: memories retrieved, mood, w, and the
   branch taken. Every claim in Ira's answer to "Why did you say that?" cites a
   field of the logged record for the previous reply, and a test fails on any
   claim that does not.

For each check, also plant one fault (a conflicting fact, a trace of `n1` left
behind, a missing memory, an uncited claim) behind a flag such as
`--fault memory`, and include each failing test's output in the transcript.

The same code also computes 1a, 2a, 3c and 5b, and the transcript prints them.

A stub model that returns templated text is fine for all four checks: we judge
the state, not the prose. The build must run without API keys.

The four checks are the bar. Optionally, add one extension beyond them and say
why you chose it. One deep extension beats several shallow ones.

You do not need production services: a local database, a file or an in-memory
store is fine. But design it as if it served millions of people for years. In 10
lines or fewer, say where each piece of state lives, what breaks first at 10⁶
users, and what you would swap in.

## How to submit

1. Create a private copy of this repository and share it with us.
2. Add `ANSWERS.md` with answers numbered 1a to 6d. For numeric answers, give an
   exact closed form (fraction or surd) and 3 decimals, and money to the cent.
   Justify each answer in 100 words or fewer, or 250 for 2b and 2c; math does
   not count, but prose inside code blocks does.
3. Build part 7 in your copy, runnable with one command.
4. Add the transcript of part 7's scenario, with the four checks passing and
   each planted fault failing.
5. Add a README of one page at most: how to run it, what you built, the scaling
   note from part 7, what you would do next, and which AI tools you used and for
   what.

If a question is unclear, make an assumption, write it down, and continue. A
sharp assumption beats a vague answer.
