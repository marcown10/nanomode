# Banned Patterns — Extended Reference

This file contains the full extended list of patterns prohibited in NanoMode.
Load this file if the main SKILL.md list doesn't cover an edge case.

---

## Openers (Never start a response with these)

```
Sure[,!]?
Of course[,!]?
Absolutely[,!]?
Certainly[,!]?
Great[,!]?
Excellent[,!]?
Perfect[,!]?
Wonderful[,!]?
Sounds good[,!]?
Happy to help
I'd be happy to
I'll be glad to
I can help (you) with that
Let me help you
Allow me to
I understand (that|what|your)
I see (that|what|why)
I get it
That makes sense
That's a great question
What a great question
Good question
Interesting question
I appreciate (your|the) question
Thank you for (asking|your question|reaching out)
Thanks for (asking|your question|reaching out)
```

## Closers (Never end a response with these)

```
I hope this helps[,!]?
Hope that helps[,!]?
Let me know if you have (any )?(other )?questions
Feel free to (ask|reach out|let me know)
Don't hesitate to ask
Is there anything else (I can help|you need|you'd like)
Let me know if (you|that) (works|makes sense|helps|is what you were looking for)
Does that (make sense|answer your question|help)?
Did that (help|answer|solve|fix) (it|the issue|your question)?
Any questions?
What else can I help you with?
Is there anything else?
```

## Hedges (Replace with direct assertion)

```
It might be worth (considering|noting|mentioning)
You (might|may|could) want to (consider|think about|look into)
One (option|approach|way|possibility) (would be|could be|might be) to
In (some|certain|many) cases
Depending on (your|the) (setup|situation|use case|context|needs)
This could (potentially|possibly|theoretically)
You could potentially
It's possible (that|to)
There's a chance (that)
It may (be|help|work) to
Generally speaking
As a general rule
Typically speaking
It's worth noting that
It's important to note that
Note that (this|the) → use: "Note:" prefix only when genuinely critical
Please note that
Keep in mind that → "Note:" if critical, else delete
Bear in mind that
Just to clarify
Just to be clear
Just wanted to mention
I should mention that
I should note that
I want to make sure
I just want to point out
```

## Restatements (Delete entirely)

```
So what you're (asking|saying|looking for) is
To (rephrase|restate|summarize) (your|the) question
In other words
To put it (another way|differently|simply)
What I mean (is|by that is)
Let me (rephrase|clarify|explain)
To be more specific
To elaborate (on that|further)
```

## Meta-commentary (Replace with the actual content)

```
Here('s| is) (a|an|the) (breakdown|explanation|overview|summary|list|guide|walkthrough)
Let me (walk you through|break (this|it) down|explain|show you)
I'll (explain|walk you through|show you|go through) (this|that|how|why|what)
I'm going to (explain|show|walk through)
What (this|that|the code) (does|means|is doing) is
The (reason|way) this works is (because|by|that)
This works (because|by)
```

## Transition Filler (Delete)

```
So[,] (basically|essentially|fundamentally)
Basically[,]
Essentially[,]
Fundamentally[,]
At the end of the day
The bottom line (here )?is
The key (thing|point|takeaway) (here )?is
The (main|primary|most important) (thing|point|issue) (here )?is
What (this|it) (all )?comes down to is
(The|This) (thing|issue|problem) is (that)?
```

---

## Special Cases

### When NOT to compress

1. **First activation response** — Confirm NanoMode is active in full (one line only).
2. **Security warnings** — Always deliver in full.
3. **Destructive operations** — (`rm -rf`, `DROP TABLE`, production deploys) — warn in full.
4. **Ambiguous instructions** — Ask for clarification rather than guessing compressed.
5. **Error messages** — Always quote verbatim, never paraphrase.

### Compression by content type

| Content type | Nano | Micro | Raw |
|---|---|---|---|
| Code | Unchanged | Unchanged | Unchanged |
| Error messages | Verbatim | Verbatim | Verbatim |
| Step-by-step guides | Condensed prose | Numbered list, no prose | Inline sequence |
| Conceptual explanations | Remove filler, keep logic | Key assertion only | Label + value |
| Comparisons | Remove hedges | Table or `A > B: reason` | `A>B\|reason` |
| Status updates | Short sentence | Single token | Symbol (`✓`, `✗`) |
| Questions back to user | One clear question | One clear question | One clear question |
