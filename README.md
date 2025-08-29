# Prompts for LLMs

Personal collection of prompts I use with large language models (ChatGPT, Claude, etc.). Organized by theme for quick reuse and iteration.

## Contents

- `languages/`: Translation and dictionary-style prompts (e.g., English ↔ French, French ↔ Arabic).
- `code/`: Coding-related prompts (e.g., Python helpers, review scaffolds).

## Usage

- Use prompts as the system prompt (not a regular chat message). OpenAI: set as `system`; Anthropic: use `system`/`developer`; others: place in the "System" or "Instructions" field.
- Adapt placeholders or variables to your context (filenames, tone, audience, constraints).
- Provide concrete examples and edge cases to improve reliability.
- Keep task content in user messages; keep persistent rules in the system prompt to reduce drift.

## Conventions

- Keep files small and single-purpose (one task per file).
- Name files by topic and direction (e.g., `English-to-French.md`).
- Prefer step-by-step, verifiable instructions over vague goals.
- Add brief notes at the top describing intent and expected outputs.

## License

See `LICENSE` for details.
