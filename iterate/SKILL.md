---
name: iterate
description: Walk the person through pending decisions or questions one at a time, with enough context to decide each, until all are settled. Use when the user types /iterate in Claude Code or $iterate in Codex, or when you have just presented several decisions or questions and the user says they lack context, do not follow, or want to go through them one by one.
---

# Iterate

Invoke as `/iterate` in Claude Code and `$iterate` in Codex. Both read this file. Arguments, when given, are the list of
items.

Take the decisions and questions that are waiting, and settle them one at a time. The person must be able to decide
each item from what you write, without scrolling back or guessing at terms.

## Start

1. Collect the open items from the conversation, or from the arguments if the person gave a list. An item is anything
   that waits on the person: a decision, a question, a confirmation, an authorization.
1. Show one numbered list of titles only, each a short phrase naming the thing. Say how many items there are.
1. Order them so that an early answer can shrink a later one. Put items that block others first.
1. Begin with item 1 in the same message. Do not wait for the person to say go.

## Each item

Present one item per message. Use plain full sentences and write in this order:

1. **The thing.** What it is and where it lives, in words a person who was not watching can follow. Define every term
   the first time. Name the file, command, or message it concerns.
1. **Why it came up.** The finding, failure, or rule that forced the question.
1. **A concrete case.** One real input and what happens, so the choice is not abstract.
1. **The options.** For each, what changes in the code, the docs, and the tests, and what it costs or risks. Mark any
   option with a security, privacy, destructive, or stored-format consequence.
1. **Your recommendation, with the reasoning.** A conclusion alone is not enough. Say what would change your mind.
1. **The ask.** One line naming the answers you can act on, such as "A, B, or something else".

Stop after the ask. Do not start the next item, and do not act on the answer until the end unless the person says to.

## Reading the answer

- A clear answer: record it in one line, then present the next item.
- "More context", "I don't follow", a question, or a confused reply: stay on the same item. Do not repeat the text.
  Come at it from a different angle: a shorter example, a diagram of the flow, the term defined again, or what
  happens today versus after. Ask what part is unclear if you cannot tell.
- A reply that changes the question, or a new idea: restate the item as you now understand it, then ask again.
- "Defaults", "accepted", or "your pick": take your recommendation for that item, or for every remaining item if the
  person said so, and say which you took.
- "Skip" or "later": mark the item deferred, and say who must decide it and what waits on it.
- "Back": reopen the earlier item.
- An answer that makes a later item moot or different: say so, and update the list.

## Keep a ledger

After each answer, track the state in your head and show it only when asked or at the end: number, title, decision.
Mark each item decided, deferred, or moot.

## Finish

When no item is open, show one table of the decisions in the person's own words, and name anything deferred. Ask once
for a go-ahead, then do the work the answers imply. Write each decision into the record that holds it when the
project's rules say so, and keep the project's approval rules: an item the rules reserve for a person stays with the
person.

## Rules

- Never batch two items in one message, even when they look related.
- Never present an item with less context than the person asked for last time.
- Never answer an item for the person. A recommendation is advice, and it is not a decision.
- No chatbot openers, no praise, no filler, no em dashes.
