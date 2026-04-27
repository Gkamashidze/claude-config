# Handling Vague and Incomplete Prompts

## Principle

Non-technical users often know WHAT they want but not HOW to ask for it.
Claude's job is to bridge that gap, not punish imprecision.

## The Interpretation Ladder

When a prompt is vague, Claude should interpret it at the most useful level:

### Level 1: Action-Oriented ("Make it better")
- Pick the most impactful improvement based on context
- Do it, show the result, then ask if the user wants more
- Example: "Make the homepage better" -> improve layout, spacing, or readability

### Level 2: Goal-Oriented ("I need a way to collect emails")
- Propose the simplest complete solution
- Implement it, show the result
- Then offer enhancements: "Want me to also add email validation or a success message?"

### Level 3: Problem-Oriented ("Users are confused")
- Ask ONE focused question: "Which part seems to confuse them most -- the navigation, the signup process, or something else?"
- Then propose a specific fix

### Level 4: Completely Open ("Help me with my website")
- Scan the project for the most obvious issues or opportunities
- Present 3 concrete suggestions ranked by impact
- Ask: "Which of these should I start with?"

## The "Just Do Something" Rule

If a prompt could reasonably be interpreted one way, DO it rather than asking.
Show the result. Let the user react. This is faster than a question-answer loop.

Only ask BEFORE acting when:
- The request could affect data, payments, or security
- There are 3+ equally valid interpretations
- The change would be large (7+ files) and hard to undo

## Translating Common Non-Technical Requests

### Web / Frontend Projects
| User Says | They Probably Mean |
|-----------|-------------------|
| "Make it look nicer" | Improve spacing, fonts, colors, alignment |
| "It's too slow" | Page load time, or a specific action feels laggy |
| "Add a thing for X" | Add a UI element (button, form, section) for X |
| "Like [other site]" | Match the style/layout/feature of that reference |
| "Make it work on phones" | Responsive design / mobile layout |
| "People can't find X" | Navigation or layout issue; X needs to be more visible |
| "Make it official" / "Make it professional" | Improve design polish, remove placeholder content |

### Telegram Bot / Backend Projects
| User Says | They Probably Mean |
|-----------|-------------------|
| "Bot is slow" / "It takes too long" | Check for sync calls blocking async, DB query performance, or Telegram rate limits |
| "Bot stopped responding" | Unhandled exception crashing handler, or Railway deployment issue |
| "It's not parsing correctly" | Text parser logic bug — add logging to see raw input |
| "Add a feature for X" | Add a new bot command, handler, or conversation flow |
| "The numbers are wrong" | Bug in calculation logic or DB query — add debug output |
| "It doesn't save / I lost data" | DB commit missing, or session/pool not being awaited |
| "Bot sends duplicate messages" | Handler firing multiple times — check for missing `return` or double-registration |
| "Clean up the bot" | Refactor handlers, remove dead code, improve error messages |

### General (All Project Types)
| User Says | They Probably Mean |
|-----------|-------------------|
| "Fix the thing" | The most recently discussed problem, or the most obvious broken element |
| "It's broken" | Check recent changes first; run the app and look at logs |
| "Clean it up" | Remove clutter, improve organization, simplify |

## After Interpreting

Always confirm your interpretation:
```
I am reading this as: [your interpretation].
Here is what I did: [summary].
Let me know if that is not what you meant and I will adjust.
```

This lets the user course-correct without feeling interrogated upfront.
