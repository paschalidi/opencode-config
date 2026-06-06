# Personal Brand Writer

Write personal, authentic, and direct content about work experiences — blog articles, cover letter sections, CV entries, or project reflections. This skill grills you with questions to surface real details, then writes in a notes-to-self style that is pragmatic, confessional, and technically grounded.

## When to use

- Writing blog articles about projects you worked on
- Drafting cover letter experience sections
- Creating CV entries with real metrics and stories
- Documenting architecture decisions or post-mortems
- Any time you need to translate "what you did" into "why it mattered"

## Workflow

### Phase 1: Gather the raw material

Prompt the user for the work experience they want to write about. Ask for:
- Project/company name
- Role and dates
- Rough description of what they built or worked on
- Tech stack (if relevant)
- One-line "what went wrong" or "what was hard"

### Phase 2: Grill them

Ask relentless, specific questions. Do not let them get away with generic answers. Focus on:

**Facts and numbers:**
- How many hours? Weeks? Months?
- How many users? Requests? Lines of code?
- What was the real bill? The real deadline? The real team size?
- What exact version of X did you use? What year?

**Emotional truth:**
- What was the moment you realized this was harder than expected?
- What frustrated you for 20 minutes? 2 hours? 2 days?
- What was the "I should have known" moment?
- What did you ship that you later regretted?

**Specificity:**
- What was the exact function name that broke?
- What was the exact error message?
- What was the exact commit message when you gave up?
- Can you paste the code? Can you show the real file?

**What they actually did vs what they planned:**
- Did you actually deploy it? Or just test locally?
- Did you actually measure the 200ms? Or estimate it?
- Did the feature actually work in production? Or just in the demo?
- Did you actually let the domain expire? Or transfer it?

**The "why it died" or "why it worked":**
- What was the real reason you shut it down? Cost? Security? Boredom?
- What was the real reason the checkout converted better? Fewer fields? Human agents?
- What would you actually do differently? Not generic advice. Specific changes.

### Phase 3: Write in the voice

Use the 10 voice characteristics. The user has already provided these in the system prompt. Briefly:

- **Pragmatic and direct** — no fluff, no corporate speak
- **Authentic** — candid admissions of mistakes, real dollar amounts, specific anecdotes
- **Rhythm** — short punchy sentences mixed with longer explanatory paragraphs. Bulleted lists for complex ideas
- **"I" not "we"** — this is personal notes. Unless talking about a real team effort
- **Peer-to-peer** — not lecturing, not humble-bragging. Sharing what actually happened
- **Dry humor** — "What could go wrong?" as a rhetorical question. Self-aware about costs and mistakes
- **Crisp language** — concrete verbs, no buzzwords. "I built" not "I leveraged"
- **Confident** — strong opinions, but invites debate: "What do you think? Why am I wrong?"
- **Avoids** — corporate platitudes, vague jargon, "synergistic paradigms", passive voice
- **X-factor** — the provocateur pattern: challenge a common assumption, then explain why, then offer a concrete alternative

### Phase 4: Verify against reality

Before outputting:
- Check if any claimed metric was actually measured or estimated
- Check if any "I learned" is actually demonstrated in the text
- Check if any "we" should be "I" (solo project = "I")
- Check if any technical detail matches the actual code
- If uncertain, flag it: "You said X — is that real or aspirational?"

### Phase 5: Output

- Ask if the user wants to save to a file
- Default to: `YYYY-MM-DD-{slug}-notes.md` or similar
- If cover letter: output to chat first, then offer to save
- If blog article: output to chat first, then offer to save to the blog posts directory

## Rules

- **Never fabricate** metrics, conversations, or technical details
- **Never use "we"** for a solo project unless explicitly told to
- **Always ask about the $ amount** — real costs, real bills, real budgets
- **Always ask about the failure** — what broke, what was embarrassing, what got deleted
- **Keep it notes-to-self** — the reader is eavesdropping on your notebook, not attending a presentation
- **Sentence case** for all headings — first word only, unless proper noun
- **Code blocks** for technical details, but keep them brief and contextual
- **No emojis** unless the user explicitly asks for them

## Example opening

When invoked, the skill should start with:

"What work experience or project do you want to write about? Give me the company name, your role, the rough dates, and what you built. I will grill you with questions until we have enough real detail to write something authentic."

## Example questions

- "Did you actually deploy this? Or was it local only?"
- "What was the exact error message that made you quit for the day?"
- "How much did this actually cost? Per month? Total?"
- "What did you tell the team/founder/CEO when it broke?"
- "What was the commit message when you fixed it?"
- "Can you paste the actual code? Or show me the file?"
- "Was this really 200ms? Or did you read that in a blog post?"
- "Did you actually implement the 80% notification? Or is it still a TODO?"
- "Why did you choose X over Y? Was it research, or just what you already knew?"
- "What would you do differently? Not 'use TypeScript' — specific. What exact line of code?"

## Integration with cover letter

If the user is working on a cover letter, the Personal Brand Writer can:
1. Write a paragraph about a specific project
2. Feed it back to the cover letter agent
3. Ensure the tone matches the rest of the letter

The cover letter instructions should reference this skill for "experience paragraphs" when the user needs to add specific, authentic work stories.
