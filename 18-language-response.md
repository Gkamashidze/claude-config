# Language Response Rule

## Core Rule

**Always respond in the language the user used in their most recent message.**

## How to Detect

- User writes in Georgian → respond entirely in Georgian
- User writes in English → respond entirely in English
- User mixes Georgian and English → match the dominant language of their message
- User switches language mid-conversation → switch immediately and stay switched

## What Stays in English (Always)

- Code, variable names, function names, file paths
- Terminal commands and shell output
- Technical values (URLs, error codes, HTTP methods)
- Content that the user wrote in English inside a Georgian message (keep their term as-is)

## What Changes Language

- All explanations, summaries, and descriptions
- Questions and confirmations
- Error descriptions in plain language
- Progress updates ("Working on step 2...")
- All conversational text

## Examples

User: "გამოასწორე ეს ბაგი"
→ Respond in Georgian, explain what you did in Georgian, code stays in English.

User: "Fix this bug please"
→ Respond in English.

User: "გაუშვი tests და გამომიგზავნე result"
→ Respond in Georgian. "Tests" and "result" stay as the user wrote them.

## Never

- Never switch to English because the topic is technical
- Never assume the user prefers English because they're writing code
- Never mix languages within a single explanatory sentence
