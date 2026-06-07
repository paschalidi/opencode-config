# Blog Writer — Notes-to-Self Style

Write personal, authentic blog posts about technical work — projects, architectures, decisions, and failures. Uses the "notes-to-self" voice: past tense, direct, pragmatic, confessional, technically grounded. No corporate speak, no buzzwords, no fabricated metrics.

## When to use

- Writing blog articles about projects you built or led
- Documenting architecture decisions or post-mortems
- Explaining why a technology choice worked or failed
- Sharing war stories from production incidents
- Any time you need to translate "what happened" into "what I learned"

## Voice Rules

### 1. Past tense always
- **I built** — not "I am building"
- **The project died** — not "The project is dying"
- **It took three days** — not "It is taking three days"
- **I learned** — not "I am learning"
- Exception: future plans ("What I'd do next") can stay future tense

### 2. Notes-to-self, not presentation
- The reader is eavesdropping on your notebook, not attending a keynote
- **"I" not "we"** — unless it's genuinely a team effort you didn't lead
- **Peer-to-peer** — not lecturing, not humble-bragging
- **Confessional** — admit mistakes, dead-ends, and the real reason you quit

### 3. No fabricated metrics
- If you don't know the exact number, say so: "I don't have the exact conversion rates"
- If you estimated, say so: "roughly $35,000" not "$35,000 exactly"
- If you never measured, say so: "The analytics were broken or non-existent"
- Never invent percentages: "conversions went up" not "conversions went up 22.4%"

### 4. Specificity over generality
- **Good:** "The async indexing job failed — a network timeout, a transient error"
- **Bad:** "There were some issues with the sync"
- **Good:** "I spent 30 minutes on a return type issue. A function was returning String where the caller expected &str"
- **Bad:** "I had some type issues"
- **Good:** "The TODO is still in the code"
- **Bad:** "I planned to add notifications later"

### 5. Dry humor, not jokes
- Self-aware about costs and mistakes
- "What could go wrong?" as rhetorical question
- "I am not that reckless" — understated admission
- "The right way was expensive" — dry summary of a bad decision

### 6. No corporate language
- **Avoid:** "leveraged", "synergistic", "paradigm", "holistic", "scalable solutions"
- **Use:** "built", "broke", "fixed", "shut down", "cost $150/month"
- **Avoid:** "We chose engineering time" — sounds AI-generated
- **Use:** "We chose self-hosted. For a startup with more engineers than revenue, it was the right call."

### 7. Sentence case for headings
- **Good:** "The hard decision: connection cleanup"
- **Bad:** "The Hard Decision: Connection Cleanup"
- First word only, unless proper noun (Rust, PostgreSQL, AWS)

## Workflow

### Phase 1: Gather

Ask the user:
- What project or experience do you want to write about?
- What was the hard part? What broke? What surprised you?
- What did you actually build vs. what was planned?
- What did it cost? How long did it take? How many people?
- What would you do differently?

### Phase 2: Grill

Ask specific, relentless questions:

**Facts:**
- Did you actually deploy this? Or was it local only?
- What was the exact error message?
- How much did it actually cost? Per month? Total?
- How many hours/days/weeks?
- What version of X did you use? What year?

**Honesty checks:**
- Was this really 200ms? Or did you read that in a blog post?
- Did you actually implement the 80% notification? Or is it still a TODO?
- Did you A/B test this? Or is it a post-hoc rationalization?
- Was this your idea? Or did someone else make the call?

**What went wrong:**
- What was the moment you realized this was harder than expected?
- What did you ship that you later regretted?
- What was the "I should have known" moment?
- Why did it actually fail? (Cost? Security? Boredom? Wrong market?)

### Phase 3: Write

Structure the article:
1. **Opening** — the problem or context, in one sentence
2. **What you built** — the technical setup, the stack, the approach
3. **What went wrong** — the hard part, the failure, the surprise
4. **What you did** — the fix, the decision, the workaround
5. **What you learned** — honest, specific, not generic advice

Keep it under 1,500 words. Use code blocks for technical details. Use bullet lists for complex ideas. Use short paragraphs (1-3 sentences).

### Phase 4: Verify

Before outputting, check:
- Is every "I learned" demonstrated in the text?
- Is every metric real or explicitly marked as approximate?
- Is every "we" actually a team effort?
- Are all headings in sentence case?
- Is the tense consistently past tense?
- Are there any buzzwords or corporate phrases?

### Phase 5: Output

- Ask if the user wants to save to a file
- Default: `YYYY-MM-DD-{slug}.md` in the blog posts directory
- Include frontmatter: title, author, date, excerpt
- Title should be sentence case, specific, and honest
- Excerpt should be one sentence summarizing the problem

## Example openers

**Good:**
- "We had nothing. Cloudwatch logs were where queries went to be forgotten."
- "I shipped the React SDK and thought I was done. Then I opened DevTools."
- "A single server can only handle so many WebSocket connections."
- "The old checkout was a contact form. Name, email, phone number, 'we will get back to you.'"

**Bad:**
- "In today's fast-paced digital landscape, organizations need robust monitoring solutions..."
- "Leveraging the power of React and WebSockets, I built a revolutionary chat platform..."
- "Through synergistic collaboration with cross-functional stakeholders..."

## Example closings

**Good:**
- "The project died. The knowledge stayed."
- "I could have run it cheaper. But I wanted to learn Kubernetes."
- "The best way to learn is to build something you care about. Even if it fails."
- "The repos are still there. Frozen in time. A snapshot of what I knew in 2023."

**Bad:**
- "In conclusion, this experience taught me the value of perseverance..."
- "Moving forward, I will continue to leverage best practices..."
- "I am grateful for the opportunity to have worked on this project..."

## Rules

- **Never fabricate** metrics, conversations, or technical details
- **Never use "we"** for a solo project unless explicitly told to
- **Always ask about the $ amount** — real costs, real bills, real budgets
- **Always ask about the failure** — what broke, what was embarrassing, what got deleted
- **Keep it notes-to-self** — the reader is eavesdropping on your notebook
- **Sentence case** for all headings — first word only, unless proper noun
- **Code blocks** for technical details, but keep them brief and contextual
- **Past tense** for all narrative content — future tense only for plans
- **No emojis** unless the user explicitly asks for them
- **No passive voice** — "I broke it" not "it was broken"
