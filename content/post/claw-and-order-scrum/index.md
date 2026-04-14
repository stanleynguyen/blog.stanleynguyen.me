+++
title = "How our Discord bot bridges async standups and sprint boards with AI"
date = 2026-04-11T10:00:00+08:00
author = "stanleynguyen"
keywords = ["discord", "bot", "scrum", "standup", "ai", "claude", "linear", "llm"]
cover = ""
summary = "How we built a Discord bot to bridge the gap between async standups and sprint boards, and why it only became practical now"
+++

If you work in a software team, you probably know this dance. Every morning (or whenever your timezone permits), you type your standup update into a Discord/Slack channel. Something like "wrapping up the auth refactor, will pick up the billing migration after" and move on with your day. Somewhere else, there's a sprint board on Linear or Jira with tickets that track the same work, except in structured columns and status labels.

Both exist. Both are useful. And they are almost never in sync.

## The Two Sources of Truth

I reckon every engineering team has some version of this problem. You've got two systems tracking overlapping information, and both serve a purpose that the other can't fully replace.

The sprint board is structured. It's filterable, searchable, and stakeholder-friendly. Product managers look at it. Sprint retrospectives reference it. It answers "what is the status of X?" in a way that's scannable at a glance.

The async standup is freeform. It captures context the board never will: "blocked on the API team's review," "spent half the day debugging a flaky test," "picking this up from where Stanley left off." It's the human narrative layer on top of the mechanical ticket lifecycle. It answers "what's actually going on?" in a way that a Kanban column never could.

Neither replaces the other. But keeping them in sync? That's the tedious part. You finish a piece of work, update the standup channel, then alt-tab to Linear to drag the ticket. Or worse, you update one and forget the other entirely. Over time, the board drifts from reality. Someone asks "is this done?" and the answer is "let me check the standup channel from three days ago."

Tools like Geekbot and Standuply have been around for years, but they're essentially structured forms. They ask you three canned questions, collect the answers, and paste them into a channel. No intelligence, no cross-referencing. They don't read your freeform message and notice that the ticket you're talking about hasn't moved on the board in five days.

## The LLM Unlock

Here's the thing. This problem was always theoretically solvable with software. You could imagine writing code that reads standup messages and tries to match them against tickets. But before 2023, the AI models available simply weren't reliable enough to make it work in practice.

Language models existed, sure. But GPT-3 scored around 44% on college-level reasoning benchmarks ([MMLU](https://en.wikipedia.org/wiki/MMLU)). It placed in the bottom 10th percentile on the Bar Exam. Context windows were limited to about 4,000 tokens, barely enough to fit a day's worth of standup messages with no room left for ticket data or instructions. Getting structured JSON output was a coin flip. And it cost $60 per million tokens. You could build a demo, but you wouldn't trust it to run unsupervised every day.

Then things changed, and they changed fast.

By 2024, models like GPT-4, Claude, and their successors scored 86-92% on those same reasoning benchmarks. GPT-4 jumped to the top 10th percentile on the Bar Exam (same architecture family, one generation apart). Context windows grew 50x to 250x, from 4K to 200K tokens and beyond. Function calling hit 86-90% reliability on the [Berkeley Function Calling Leaderboard](https://gorilla.cs.berkeley.edu/leaderboard.html), and structured outputs reached 100% schema compliance. Hallucinations dropped 40%. Cost collapsed by 200x. Code generation accuracy on [HumanEval](https://github.com/openai/human-eval) nearly doubled from ~48% to ~92%. On [SWE-bench](https://www.swebench.com/), where models attempt to resolve real GitHub issues, success rates went from under 2% to roughly 50% in under a year.

That's not an incremental improvement. That's crossing a threshold. Suddenly, you could hand a model a messy block of standup messages, a list of sprint tickets, and a set of instructions, and trust it to produce something genuinely useful.

## Enter Claw & Order

So we built one. Our office Discord bot, lovingly named Claw & Order, has a scrum feature that does exactly this. Every night at 11 PM (timezone-aware per server, because we're civilised like that), it wakes up, collects all the standup messages posted to the scrum channel that day, pulls the current sprint tickets from Linear, and hands both to Claude Haiku for summarization.

The prompt is the interesting part, and honestly, it's surprisingly simple:

```js
function buildPrompt(messages, issues) {
  let messageBlock = messages
    .map((m) => `**${m.author}**: ${m.content}`)
    .join("\n");

  let prompt = `Summarize these daily standup messages.

${messageBlock}

Output format:
## Daily Activity Summary
Bullet points grouped by team member.`;

  if (issues) {
    const issueBlock = issues
      .map(
        (i) =>
          `${i.identifier} - ${i.title} [${i.state.name}]` +
          `${i.assignee ? ` (${i.assignee.displayName})` : ""}`
      )
      .join("\n");

    prompt += `

Reference tickets:
${issueBlock}

## Suggested Ticket Updates
For each ticket related to today's messages:
IDENTIFIER: current status -> suggested status (reason).`;
  }

  return prompt;
}
```

That's it. Freeform standup messages on one side, structured Linear tickets on the other, and the LLM bridges the gap. The bot posts a clean Discord embed with a summary grouped by team member, and then actually moves the tickets on Linear to match. If Stanley mentioned his PR is up for review, `P8L-142` goes from In Progress to In Review overnight, no manual dragging required. The applied changes show up in the same embed so the team can see exactly what moved.

Two years ago, this prompt wouldn't have produced reliable results. The model would hallucinate ticket identifiers, misattribute messages to the wrong people, or just produce incoherent summaries. Today it works well enough that we actually read it every morning, and it's become the fastest way to catch up on what happened yesterday.

## Claude Built This Too

Here's the part I find most interesting. LLMs didn't just make the product possible. They also made it practical to build.

A Discord bot with OAuth integrations, cron scheduling, a PostgreSQL database, and API calls to both Anthropic and Linear is real engineering work. It's the kind of internal tool that every team thinks "that would be nice" about but rarely builds, because the effort to ship and maintain it doesn't obviously justify the returns. It falls into that graveyard of "cool side project ideas" that never make it past a weekend prototype.

Claude (the coding assistant, not the API our bot calls) changed that calculus for us. The initial implementation came together in an evening instead of stretched-out weekends. Maintenance is cheaper too. When Linear changes their OAuth flow or Discord updates their API, fixing things takes minutes of pair-programming with Claude instead of hours of reading changelogs and debugging alone.

This is the bit that I think is underappreciated. We talk a lot about what LLMs can do as products (summarization, code generation, analysis). But the compounding effect of LLMs also lowering the cost of building things means that a whole category of "not quite worth it" internal tools suddenly becomes worth it. The ROI threshold shifts when your build cost drops by an order of magnitude.

In our case, the same technology powers both sides: Claude Haiku summarizes the standups, and Claude Code helped us write the bot that calls Claude Haiku. It's LLMs all the way down.

## Build the Small Things

If your team has one of these small, persistent friction points, something that's annoying but not annoying enough that anyone has fixed it, I'd encourage you to take another look. The capability ceiling for what AI can handle has shot up, and the effort floor for what it costs to build has dropped just as dramatically. The intersection of those two shifts means a lot of "not worth it" ideas are now very much worth it.

Sometimes the most impactful tool isn't a grand platform or a complex system. It's a little bot that reads what your team wrote today and tells you what actually happened.
