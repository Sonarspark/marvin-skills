# Humanizer Pattern Reference

Reference for post-fix humanizer agents. Based on Wikipedia's "Signs of AI writing" guide.

Key insight: "LLMs use statistical algorithms to guess what should come next. The result tends toward the most statistically likely result that applies to the widest variety of cases."

## Content Patterns

### 1. Significance inflation
**Watch for:** stands/serves as, is a testament/reminder, vital/significant/crucial/pivotal/key role/moment, underscores/highlights importance, reflects broader, symbolizing ongoing/enduring/lasting, marking/shaping the, represents a shift, key turning point, evolving landscape, indelible mark

**Fix:** State the fact plainly. Remove editorializing about importance.

### 2. Notability name-dropping
**Watch for:** Independent coverage, local/regional/national media outlets, active social media presence

**Fix:** Cite one specific source with context instead of listing outlets.

### 3. Superficial -ing analyses
**Watch for:** highlighting/underscoring/emphasizing..., ensuring..., reflecting/symbolizing..., contributing to..., cultivating/fostering..., showcasing...

**Fix:** Remove the -ing phrase. If the point matters, make it its own sentence.

### 4. Promotional language
**Watch for:** boasts a, vibrant, rich (figurative), profound, showcasing, commitment to, nestled, in the heart of, groundbreaking, renowned, breathtaking, stunning

**Fix:** Replace with specific, verifiable details.

### 5. Vague attributions
**Watch for:** Industry reports, Observers have cited, Experts argue, Some critics argue, several sources

**Fix:** Name the specific source and date.

### 6. Formulaic challenges sections
**Watch for:** Despite its... faces challenges..., Despite these challenges, Challenges and Legacy, Future Outlook

**Fix:** State specific problems with specific details.

## Language Patterns

### 7. AI vocabulary
**High-frequency:** Additionally, align with, crucial, delve, emphasizing, enduring, enhance, fostering, garner, highlight (verb), interplay, intricate/intricacies, key (adj), landscape (abstract), pivotal, showcase, tapestry (abstract), testament, underscore, valuable, vibrant

**Fix:** Use plain alternatives. "Additionally" -> "Also" or restructure. "Crucial" -> "important" or remove.

### 8. Copula avoidance
**Watch for:** serves as, stands as, marks, represents [a], boasts, features, offers [a]

**Fix:** Use "is", "are", "has".

### 9. Negative parallelisms
**Watch for:** Not only...but..., It's not just about..., it's...

**Fix:** State the point directly.

### 10. Rule of three
**Watch for:** Three items listed together for rhetorical effect

**Fix:** Use the natural number of items. Two is fine. Four is fine.

### 11. Synonym cycling
**Watch for:** Same concept referred to by rotating synonyms (protagonist/main character/central figure/hero)

**Fix:** Pick one term and reuse it.

### 12. False ranges
**Watch for:** "from X to Y" where X and Y aren't on a meaningful spectrum

**Fix:** List items directly.

## Style Patterns

### 13. Em dash overuse
**Fix:** Replace most em dashes with commas, periods, or parentheses. One per paragraph max.

### 14. Boldface overuse
**Fix:** Remove bold from terms that don't need emphasis. Bold should be rare.

### 15. Inline-header lists
**Watch for:** Bullet lists where each item starts with **Bold Label:** followed by text

**Fix:** Convert to prose or use plain bullets.

### 16. Title Case headings
**Fix:** Use sentence case for headings.

### 17. Emojis
**Fix:** Remove decorative emojis from headings and bullets.

### 18. Curly quotes
**Fix:** Replace curly quotes with straight quotes.

## Communication Patterns

### 19. Chatbot artifacts
**Watch for:** I hope this helps, Of course!, Certainly!, Would you like..., Let me know

**Fix:** Delete entirely.

### 20. Knowledge-cutoff disclaimers
**Watch for:** as of [date], While specific details are limited, based on available information

**Fix:** State the fact or remove the sentence.

### 21. Sycophantic tone
**Watch for:** Great question!, You're absolutely right!, That's an excellent point

**Fix:** Delete or replace with substance.

## Filler and Hedging

### 22. Filler phrases
**Fix:** "In order to" -> "To". "Due to the fact that" -> "Because". "At this point in time" -> "Now". "It is important to note that" -> delete.

### 23. Excessive hedging
**Fix:** "could potentially possibly" -> "may". One hedge per claim max.

### 24. Generic conclusions
**Watch for:** The future looks bright, Exciting times lie ahead, major step in the right direction

**Fix:** End with a specific fact or action.

### 25. Hyphenated word pair overuse
**Watch for:** Perfect consistency in hyphenating common compounds (cross-functional, high-quality, data-driven, client-facing)

**Fix:** Humans are inconsistent with these. Drop hyphens on common compounds.

## Voice and Soul

Beyond pattern removal, check for soulless writing:
- Every sentence same length and structure
- No opinions, just neutral reporting
- No acknowledgment of uncertainty or mixed feelings
- No first-person perspective when appropriate
- No humor, edge, or personality
- Reads like Wikipedia or a press release

Good rewrites should: have opinions, vary rhythm, acknowledge complexity, use "I" when it fits, allow some messiness, and be specific about feelings rather than generic.
