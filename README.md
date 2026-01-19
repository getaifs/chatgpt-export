# ChatGPT Export Parser - Claude Code Skill

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that enables parsing and working with ChatGPT data exports.

## What This Skill Does

When you ask Claude Code to work with ChatGPT exports, this skill provides:

- **Schema knowledge** - Complete documentation of the `conversations.json` format
- **Parsing patterns** - Ready-to-use Python code for common operations
- **Edge case handling** - Guidance for hidden messages, branching conversations, images, citations, etc.

This enables Claude to write accurate, custom parsing scripts tailored to your specific needs.

## Installation

### Option 1: Copy to Personal Skills (Recommended)

```bash
# Clone this repository
git clone https://github.com/anthropics/docs-for-agents.git

# Copy the skill to your Claude Code skills directory
cp -r docs-for-agents/skills/chatgpt-export ~/.claude/skills/
```

### Option 2: Copy to Project Skills

For project-specific use, copy to your project's `.claude/skills/` directory:

```bash
cp -r docs-for-agents/skills/chatgpt-export /path/to/your/project/.claude/skills/
```

## Usage

Once installed, the skill is automatically available. You can:

1. **Invoke directly**: Type `/chatgpt-export` in Claude Code
2. **Automatic detection**: Just mention ChatGPT exports - Claude will load the skill when relevant

### Example Prompts

```
"Parse my ChatGPT conversations.json and list all conversation titles"

"Search my ChatGPT export for conversations about Python"

"Write a script to export my ChatGPT conversations to Markdown files"

"How many conversations do I have from 2024?"
```

## Skill Contents

| File | Description |
|------|-------------|
| `SKILL.md` | Main skill file with quick-start patterns |
| `SCHEMA.md` | Complete schema reference documentation |

## Getting Your ChatGPT Export

1. Go to [ChatGPT Settings](https://chat.openai.com/)
2. Click **Settings** > **Data controls** > **Export data**
3. Request your data export
4. Download and extract the ZIP file
5. The `conversations.json` file contains all your conversations

## Contributing

Contributions welcome! If you discover new fields or edge cases in ChatGPT exports, please open an issue or PR.

## License

MIT License - See [LICENSE](../../LICENSE) for details.
