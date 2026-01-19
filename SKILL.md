---
name: chatgpt-export
description: Parse and work with ChatGPT data exports (conversations.json). Use when users want to read, search, analyze, or write scripts to process their ChatGPT conversation history.
---

# ChatGPT Export Parser

This skill provides knowledge to parse ChatGPT data exports. Use this when users ask you to work with their ChatGPT conversation history.

## Export Structure Overview

ChatGPT exports contain a `conversations.json` file:
- **Format**: Single-line minified JSON array
- **Size**: Can exceed 200MB for active users
- **Structure**: Array of conversation objects, each using a tree structure for messages

## Quick Start: Loading Conversations

```python
import json

def load_conversations(path):
    """Load ChatGPT conversations.json file."""
    with open(path, 'r', encoding='utf-8') as f:
        return json.load(f)

conversations = load_conversations("conversations.json")
print(f"Found {len(conversations)} conversations")
```

## Key Parsing Patterns

### Extract Visible Messages from a Conversation

```python
def get_visible_messages(conversation):
    """Extract user/assistant messages, filtering hidden system messages."""
    messages = []
    for node in conversation['mapping'].values():
        msg = node.get('message')
        if not msg:
            continue

        # Skip system messages and hidden content
        if msg['author']['role'] == 'system':
            continue
        if msg.get('weight', 1.0) == 0.0:
            continue
        if msg.get('metadata', {}).get('is_visually_hidden_from_conversation'):
            continue

        messages.append(msg)

    # Sort by creation time
    messages.sort(key=lambda m: m.get('create_time') or 0)
    return messages
```

### Extract Text Content from a Message

```python
def get_text_content(message):
    """Extract text from a message, handling multimodal content."""
    parts = message['content'].get('parts', [])
    text_parts = []

    for part in parts:
        if isinstance(part, str):
            text_parts.append(part)
        elif isinstance(part, dict):
            content_type = part.get('content_type')
            if content_type == 'image_asset_pointer':
                text_parts.append(f"[Image: {part['asset_pointer']}]")

    return '\n'.join(text_parts)
```

### Follow Conversation Thread (Tree Navigation)

```python
def get_conversation_thread(mapping, current_node_id):
    """Follow the tree from current_node back to root."""
    thread = []
    node_id = current_node_id

    while node_id:
        node = mapping[node_id]
        if node.get('message'):
            thread.append(node)
        node_id = node.get('parent')

    return list(reversed(thread))
```

## Essential Fields Reference

### Conversation Object
| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique conversation ID |
| `title` | string | User-visible title |
| `create_time` | number | Unix timestamp (seconds) |
| `mapping` | object | Tree of message nodes |
| `current_node` | string | ID of active message |

### Message Object
| Field | Type | Description |
|-------|------|-------------|
| `author.role` | string | "user", "assistant", "system", or "tool" |
| `content.content_type` | string | "text", "multimodal_text", "code", etc. |
| `content.parts` | array | Content parts (strings or objects) |
| `create_time` | number/null | Message timestamp |

### Content Types
- `text` - Plain text, parts is `["message text"]`
- `multimodal_text` - Mixed content, parts can include image objects
- `code` - Code blocks
- `execution_output` - Code execution results
- `thoughts` - Internal reasoning (o1/o3 models)

## Image Handling

Images use asset pointers:
```json
{
  "content_type": "image_asset_pointer",
  "asset_pointer": "sediment://file_XXXXXXXX",
  "width": 1024,
  "height": 768
}
```

Image files in export: `file_XXXXXXXX-sanitized.jpg`

## Common Use Cases

1. **List conversations**: Iterate `conversations` array, access `title` and `create_time`
2. **Search by content**: Extract text from each message, search with regex or string matching
3. **Export to Markdown**: Format messages with headers for user/assistant roles
4. **Date filtering**: Compare `create_time` with target timestamps

For complete schema documentation, see [SCHEMA.md](SCHEMA.md).
