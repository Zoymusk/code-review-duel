# Code Review Duel

Two AI reviewers on **different models** review the same code in one Switch room, check each other's work, and ask a human to pick the better review.

Built for the Flip the Switch Hackathon. Runs entirely on free models, with no API keys and no subscriptions.

**Demo video:** https://youtu.be/Dyu2Fs-55sg

## Why this exists

One AI reviewer can be confidently wrong, and you often can't tell. This project puts two reviewers on different models in one room, so they cross-check each other and a human makes the final call.

In my first test of `int x = 5 / 0;` (Java), the two models disagreed:

- **reviewer-a** (DeepSeek) said it is a compile error. Wrong: it compiles and throws `ArithmeticException` at runtime.
- **reviewer-b** (GLM) said it compiles and throws at runtime. Right.

Model answers vary between runs. On a later run reviewer-a got it right, and on other snippets (a `==` vs `.equals` string bug, a six-bug `GradeBook` class, a `ShoppingCart`, a `Library` class) both models agreed. When they agree, you get extra confidence. When they split, the room shows it and you decide.

## How it works

1. You post a code snippet in `code-review-room`.
2. reviewer-a posts its review (starts with `A:`) and hands over to reviewer-b.
3. reviewer-b posts its review (starts with `B:`) and asks reviewer-a if it agrees.
4. reviewer-a answers "I agree because..." or "I disagree because...".
5. reviewer-b does the same, then asks the human: "Admin, reply A or B to pick."
6. You reply `A` or `B`, and both reviewers acknowledge and stop.

## Why Switch is central

This project does not work without Switch:

- **One room, two agents.** Both reviewers read the same room history, so each can answer the other's review.
- **Agents address each other by name.** The whole chain (review, hand-over, agree/disagree, ask the human) runs through mentions in the room.
- **The human is in the same room.** Your pick is just another message in the thread.
- **Different models, one place.** Switch connects both agents to the same chat without any glue code.

## Setup

You need Docker Desktop, Node.js, and Switch Console.

1. Install OpenCode: `npm install -g opencode-ai`
2. Check the free models: `opencode models` (look for names ending in `-free`)
3. Install Switch Console: https://github.com/sandbox-quantum/switch/releases/latest
4. Start Docker Desktop, open Switch Console, and click Start on the local server.
5. Import `code-review-duel.template.yaml`. First change both `machine:` lines to your computer's name.
6. Open `code-review-room` and post a snippet, tagging both reviewers:

```
@reviewer-a @reviewer-b Review this Java code:
int x = 5 / 0;
System.out.println(x);
```

If a reviewer goes quiet, its free model may be busy. Swap it for another `-free` model from `opencode models`.

## Models used

| Agent | Model |
|---|---|
| reviewer-a | `opencode/deepseek-v4.1-flash-free` |
| reviewer-b | `opencode/glm-5.3-flash-free` |

## What I learned

- **Agents only act when mentioned.** At first reviewer-b waited for reviewer-a and then sat idle, because nothing woke it up. Fixing it meant making reviewer-a end its review with "@reviewer-b your turn."
- **Instructions have to name who speaks next.** Each step in the chain explicitly tells the agent who to mention and what to ask.
- **A room needs a chat app attached.** My first room was agent-only, so there was nowhere to type. Putting it on Mattermost fixed that.
- **Free models vary.** One model was unavailable during testing, and answers differ between runs, so I don't build the demo around a guaranteed disagreement.
- **Reading the other review makes reviews similar.** Letting reviewer-b see reviewer-a's review makes the cross-check easy, but their findings end up close to each other.
