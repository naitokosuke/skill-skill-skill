# skill-skill-skill

My personal collection of Claude Code Skills.

## What are Claude Code Skills?

Claude Code Skills are reusable prompts and workflows for Claude Code. They can be invoked using slash commands like `/skill-name`.

## Directory Structure

```
.claude/
└── skills/
    └── <skill-name>.md    # Skill definition files
```

## Creating a Skill

1. Create a Markdown file in the `.claude/skills/` directory
2. The filename becomes the skill name (e.g., `review.md` → `/review`)
3. Write the skill instructions in the file

### Example Skill File

```markdown
---
description: Perform a code review
---

Please review the code with the following criteria:

1. Code quality and readability
2. Performance issues
3. Security concerns
4. Test coverage
```

## Usage

Invoke skills in Claude Code like this:

```
/skill-name
```

## License

MIT
