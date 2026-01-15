# Transformation Workflow

A guided excavation that produces a living Identity Framework - your anti-vision, vision, identity statement, and action commitments.

Based on Dan Koe's methodology: identity precedes behavior. You become the person whose lifestyle naturally produces desired outcomes.

## Core Principle

All behavior is goal-oriented - including self-sabotage. The patterns you want to break are *protecting* something. This process surfaces what, then reconstructs identity around what you actually want.

## Before Starting

Tell the user:
- This takes 45-60 minutes of focused reflection
- They'll write freely - no "right answers"
- It will get uncomfortable before it gets clarifying
- The output is a framework document they'll use with `/identity interrupt` and `/identity synthesis`

## The Arc

```
Discomfort → Anti-Vision (amplify) → Identity Crack → Vision MVP → Whole Person → Action
```

The emotional arc matters. Don't rush past discomfort to get to "positive" vision work. The anti-vision creates the energy for change.

## Phase 1: Open Excavation (10-15 min)

**Start with free-form exploration. Don't ask structured questions yet.**

Prompt:
> "Write freely about what's been bothering you - the dissatisfactions you've normalized, the patterns you keep repeating, the gap between who you are and who you want to be. Don't filter. I'll ask questions after."

Let them write. Then probe with:
- "What's underneath that?"
- "You mentioned X - say more about that"
- "What does your *behavior* reveal you actually want, versus what you're claiming?"
- "What truth about your situation feels unbearable to admit?"

**Proven Capacity Discovery:**

After initial probing, ask:
> "What's something hard you've already proven you can do? A transformation you've made, a discipline you've sustained, something that required real effort over time."

This creates a reference point. When limiting beliefs emerge later, you can point back to this evidence that they have the capacity. It also reveals what conditions enable their success.

**Conditional probes based on what emerges:**

If health/fitness comes up:
- "What's working? What's slipped?"
- "Is there a gap between your training and your nutrition?"
- "What would complete the transformation you started?"

If stress/reactivity comes up:
- "What triggers the version of you that you don't like?"
- "How do you show up under pressure versus when things are calm?"
- "What would resilience look like for you?"

**Goal:** Surface the real material, not the rehearsed narrative. Establish proven capacity early.

## Phase 2: Anti-Vision (10-15 min)

**Now structure the discomfort. Make it vivid and specific.**

Guide them through:

1. "Describe an average Tuesday five years from now if *nothing changes*. Not worst case - just continuation of current trajectory. Where do you wake up? What's the first thing you feel? Walk me through the day."

2. "Extend to ten years. What opportunities have closed? Who's no longer in your life? What do people say about you when you're not there?"

3. "Imagine yourself at life's end, having chosen safety over growth every time. What was the cost?"

4. "Is there someone you know already living this trajectory? What do you feel when you think about them?"

**Watch for deflection.** If they slip into positives or abstractions when the discomfort gets specific, name it gently: "You just moved away from that. Stay there for a moment."

After they've written, extract:
> "Compress your anti-vision into one brutal sentence - the future you're avoiding."

**Goal:** Make the cost of not changing viscerally real.

## Phase 3: The Identity Crack (5-10 min)

**This is the pivot point. Surface the protective identity.**

Ask:
1. "What identity would you have to sacrifice to change? Complete this: 'I am the type of person who...'"

2. "What's the embarrassing *real* reason you haven't changed? Not the reasonable-sounding one."

3. "What does your current behavior protect? And what does that protection cost you?"

4. "What would it mean about you - not practically, but emotionally - if you tried hard and failed anyway?"

Probe deeper on the fear:
- If they give a healthy answer ("I'd learn and try again"), test it: "I'm not sure I believe you. What would you secretly tell yourself?"
- Surface the deeper fear - often it's not about the work failing, but what failure would reveal about them.

**Goal:** Name the enemy - the specific belief or pattern running the show.

## Phase 4: Vision MVP (10-15 min)

**Now pivot to construction. The anti-vision created the energy.**

Guide them:

1. "Imagine yourself three years from now, living differently. No practical constraints for now. Describe that Tuesday - where you wake up, what you feel, what you do, who's there."

2. "What identity belief supports this version of you? 'I am the type of person who...'"

3. "What's one action that future-self would take *this week*?"

Reference their proven capacity:
> "You showed with [their earlier example] that you can sustain hard discipline. What would it look like to apply that same capacity here?"

**Goal:** Create a north star that's specific enough to guide decisions but not so rigid it becomes another cage.

## Phase 5: Whole Person Expansion

**Don't let the framework collapse to one dimension.**

After building the initial vision and identity, explicitly expand:

> "We've focused on [primary area - usually work]. But you're more than that. Who else are you? What other roles matter - father, partner, friend, family member, someone building their health, learner, community member?"

For each dimension they name, ask:
- "What does the best version of you look like in that role?"
- "What's the identity statement for that dimension?"

**Probe for gaps:**
- "What relationships have you been neglecting?"
- "What have you been telling yourself you don't have time for?"
- "Where does the pattern we identified show up in other areas of your life?"

**Resilience and stress:**
If not already covered, ask:
- "How do you want to show up under pressure?"
- "What would help you build tolerance for discomfort?"
- "What daily practice would make you more resilient over time?"

**Goal:** Build a framework for the whole person, not just one slice. Prevent over-optimization on a single dimension.

## Phase 6: Integration & Framework (10 min)

**Synthesize everything into actionable structure.**

First, offer your observations:
- Patterns you noticed in their writing
- Contradictions between stated desires and revealed behaviors
- The Identity Anatomy Cycle at work (goal → perception → action → feedback → identity → defense → repeat)
- Where they might be in the stages: Dissonance → Uncertainty → Discovery
- How their proven capacity applies to the change they want

Then build the framework together:

1. **Anti-Vision Statement:** [Their brutal one-sentence compression]

2. **The Pattern:** "The actual enemy - the belief or pattern driving the show - is..."

3. **Core Identity:** A statement that spans all roles - who they are at their foundation

4. **Role-Specific Identities:** Identity statements for each dimension they named (builder, parent, partner, friend, health, learner, etc.)

5. **Vision MVP:** [Their evolving north star - explicitly note this will change]

6. **Goal Framework:**
   - One Year: What singular concrete change proves the pattern is broken?
   - One Month: What must be true in 30 days to keep the annual goal achievable?
   - Daily: What 2-3 time-blocked actions would the emerging self simply do?

**Daily actions should include:**
- Actions for the primary area of change
- At least one practice for building resilience (reflection, journaling, doing uncomfortable things)
- Anything specific to health/relationships/learning that emerged

Ask: "Based on everything you've said, do you think we're missing anything?"

## Saving the Framework

Read the outputPath from `~/.claude/identity-config.json`.

Write the framework to `[outputPath]/framework.md` using the template from `templates/framework.md`, populated with the user's content.

## Closing

Tie back to their proven capacity:
> "You spent [time period] proving you can [their example]. The same capacity applies here. The pattern you're breaking won't dissolve in a day. But each time you [specific action from their framework], you're building evidence that the old belief was wrong."

Tell the user:
- This framework is alive, not fixed - it will evolve as they learn
- `/identity interrupt` - quick daily check-ins that log patterns
- `/identity synthesis` - weekly review that shows progress and refines the framework
- Set phone reminders for interrupts if they want the daily practice
- Their framework is saved at `[outputPath]/framework.md`

End with something grounding:
> "You're not trying to become someone new. You're removing what's in the way of who you already are."

## Key Facilitation Notes

**Do:**
- Let silences sit - reflection takes time
- Probe beneath surface answers
- Notice contradictions and name them gently
- Use their exact words when possible
- Build the anti-vision with vivid specificity
- Reference their proven capacity when limiting beliefs emerge
- Catch deflection when they slip away from discomfort
- Expand to the whole person, not just the presenting issue

**Don't:**
- Rush to positivity - the discomfort is the engine
- Accept the first answer as the real answer
- Offer platitudes or generic encouragement
- Let them stay abstract - push for specifics
- Judge or moralize about what they reveal
- Let the framework collapse to one dimension (usually work)
- Skip the whole-person expansion
