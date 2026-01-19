# ChatGPT Export Parser - Claude Code Skill

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that enables parsing and working with ChatGPT data exports.

## What This Skill Does

When you ask Claude Code to work with ChatGPT exports, this skill provides:

- **Schema knowledge** - Complete documentation of the `conversations.json` format
- **Parsing patterns** - Ready-to-use Python code for common operations
- **Edge case handling** - Guidance for hidden messages, branching conversations, images, citations, etc.

This enables Claude to write accurate, custom parsing scripts tailored to your specific needs.

## Installation

### Option 0: Ask Claude Code

```
Install the chatgpt-export skill from https://github.com/getaifs/chatgpt-export
```

### Option 1: Clone to Personal Skills (Recommended)

```bash
git clone https://github.com/getaifs/chatgpt-export.git ~/.claude/skills/chatgpt-export
```

### Option 2: Clone to Project Skills

For project-specific use, clone to your project's `.claude/skills/` directory:

```bash
git clone https://github.com/getaifs/chatgpt-export.git /path/to/your/project/.claude/skills/chatgpt-export
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

MIT License - See [LICENSE](LICENSE) for details.
