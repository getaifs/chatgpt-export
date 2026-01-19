# ChatGPT Export Schema Reference

Complete reference for parsing ChatGPT conversation exports (`conversations.json`).

---

## File Structure

### Top Level

```json
[
  { conversation_object_1 },
  { conversation_object_2 },
  ...
]
```

The root is an array containing all conversations. Exports typically contain hundreds to thousands of conversations.

---

## Conversation Object Schema

### Core Fields (Always Present)

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `id` | string | Unique identifier | "67a678d3-a274-8191-94ee-7ca817b25b36" |
| `title` | string | User-visible title (may be auto-generated) | "Python Data Processing" |
| `create_time` | number | Unix timestamp (seconds with decimal) | 1700000000.123456 |
| `update_time` | number | Unix timestamp of last update | 1700001000.654321 |
| `mapping` | object | Tree structure containing all messages | {...} |
| `current_node` | string | ID of the currently active message node | "5afdd785-e333-42ce-8453-1ada4380626e" |
| `conversation_id` | string | Alternative conversation identifier | "67a678d3-a274-8191-94ee-7ca817b25b36" |
| `is_archived` | boolean | Whether conversation is archived | false |

### Optional Fields

| Field | Type | Description | Presence |
|-------|------|-------------|----------|
| `default_model_slug` | string | Model used (e.g., "gpt-4", "gpt-4o", "o1-preview") | ~98% |
| `conversation_template_id` | string | GPT/project template ID ("g-p-" prefix for projects) | ~25% |
| `gizmo_id` | string | Custom GPT identifier | ~25% |
| `safe_urls` | array | Whitelisted URLs for browsing | ~95% |
| `is_do_not_remember` | boolean | Privacy setting for memory | ~95% |
| `voice` | string | Voice setting if using voice mode | ~4% |

---

## Message Mapping Structure

The `mapping` field contains the conversation tree. Each key is a UUID for a message node.

### Node Structure

```json
{
  "node-uuid-here": {
    "id": "node-uuid-here",
    "message": { message_object } | null,
    "parent": "parent-node-uuid" | null,
    "children": ["child-uuid-1", "child-uuid-2", ...]
  }
}
```

### Message Object

When `message` is not null:

```json
{
  "id": "message-uuid",
  "author": {
    "role": "user" | "assistant" | "system" | "tool",
    "name": string | null,
    "metadata": {}
  },
  "create_time": number | null,
  "update_time": number | null,
  "content": {
    "content_type": "text" | "multimodal_text" | "code" | ...,
    "parts": [...]
  },
  "status": "finished_successfully" | "in_progress" | ...,
  "end_turn": boolean | null,
  "weight": number,
  "metadata": {
    "is_visually_hidden_from_conversation": boolean,
    "attachments": [...],
    "model_slug": string,
    ...
  },
  "recipient": "all" | string
}
```

---

## Content Types

The `content.content_type` field determines how to interpret `parts`.

| Content Type | Description | Parts Format |
|--------------|-------------|--------------|
| `text` | Plain text message | `["message string"]` |
| `multimodal_text` | Mixed text and media | Array of strings and/or image objects |
| `code` | Code blocks | `["code string"]` |
| `execution_output` | Code execution results | `["output string"]` |
| `computer_output` | Terminal output | `["output string"]` |
| `reasoning_recap` | Summary of reasoning | `["summary string"]` |
| `thoughts` | Internal AI reasoning (o1/o3 models) | `["thought string"]` |
| `tether_browsing_display` | Browser tool output | Complex object |

### Content Examples

**Simple Text:**
```json
{
  "content_type": "text",
  "parts": ["Hello, how can I help you today?"]
}
```

**Multimodal (Text + Images):**
```json
{
  "content_type": "multimodal_text",
  "parts": [
    {
      "content_type": "image_asset_pointer",
      "asset_pointer": "sediment://file_0000000000000000000000000000001a",
      "size_bytes": 102400,
      "width": 1024,
      "height": 768,
      "metadata": { "sanitized": true }
    },
    "What do you see in this image?"
  ]
}
```

---

## Author Roles

| Role | Description | Visibility |
|------|-------------|------------|
| `user` | Human user messages | Always visible |
| `assistant` | AI assistant responses | Always visible |
| `system` | System messages/prompts | Usually hidden |
| `tool` | Tool execution results | Collapsible |

**Common Tool Names:**
- `file_search` - File/document search
- `browser` - Web browsing
- `python` - Code execution
- `dalle` - Image generation

---

## Tree Navigation

### Root Node
- Has `parent: null`
- Represents conversation start
- Usually has `message: null`

### Linear Path
Most conversations follow a single parent->child chain.

### Branching
Multiple children indicate:
1. User edited their message
2. User regenerated assistant response
3. Conversation split into paths

### Get Conversation Thread

```python
def get_conversation_thread(mapping, current_node_id):
    """Follow from current_node back to root."""
    thread = []
    node_id = current_node_id

    while node_id:
        node = mapping[node_id]
        if node.get('message'):
            thread.append(node)
        node_id = node.get('parent')

    return list(reversed(thread))
```

---

## Image References

### Asset Pointer Format
```
sediment://file_XXXXXXXXXXXXXXXXXXXXXXXX
```

### Image Object Structure
```json
{
  "content_type": "image_asset_pointer",
  "asset_pointer": "sediment://file_0000000000000000000000000000001a",
  "size_bytes": 102400,
  "width": 1024,
  "height": 768,
  "fovea": null,
  "metadata": { "sanitized": true }
}
```

### Resolving Image Files
Export includes sanitized images:
- Format: `file_XXXXXXXX-sanitized.jpg` (or `.jpeg`, `.png`)
- Extract file ID from `asset_pointer`, append `-sanitized.{ext}`

---

## Edge Cases

### Empty Messages
Some messages have empty content:
```json
{ "content": { "content_type": "text", "parts": [""] } }
```
**Handling:** Skip when rendering.

### Missing create_time
Some system messages have `create_time: null`.
**Handling:** Use parent's time or conversation's `create_time`.

### Hidden Messages
Filter messages where:
- `weight == 0.0`
- `metadata.is_visually_hidden_from_conversation == true`

### Citations
ChatGPT includes citations in formats:
- Chinese brackets: `【cite】【turn1view3】`
- Unicode markers: `\ue200cite\ue202turn1view3\ue201`

**Regex patterns:**
```python
chinese_brackets = r'【cite】【[^】]+】'
unicode_markers = r'\ue200(?:file)?cite\ue202[^\ue201]+\ue201'
```

### Reasoning Models (o1/o3)
May include:
- `content_type: "thoughts"` - Internal reasoning
- `content_type: "reasoning_recap"` - Reasoning summary

---

## Complete Parsing Examples

### Extract All Visible Messages

```python
def extract_visible_messages(conversation):
    messages = []
    for node in conversation['mapping'].values():
        msg = node.get('message')
        if not msg:
            continue

        # Filter system/hidden messages
        if msg['author']['role'] == 'system':
            continue
        if msg.get('weight', 1.0) == 0.0:
            continue
        if msg.get('metadata', {}).get('is_visually_hidden_from_conversation'):
            continue

        messages.append(msg)

    # Sort by create_time
    messages.sort(key=lambda m: m.get('create_time') or 0)
    return messages
```

### Extract Text Content

```python
def extract_text_content(message):
    parts = message['content'].get('parts', [])
    text_parts = []

    for part in parts:
        if isinstance(part, str):
            text_parts.append(part)
        elif isinstance(part, dict) and part.get('content_type') == 'image_asset_pointer':
            text_parts.append(f"[Image: {part['asset_pointer']}]")

    return '\n'.join(text_parts)
```

### Search Conversations by Title

```python
def search_by_title(conversations, query):
    query_lower = query.lower()
    return [c for c in conversations if query_lower in c.get('title', '').lower()]
```

### Search Conversations by Content

```python
def search_by_content(conversations, query):
    query_lower = query.lower()
    results = []

    for conv in conversations:
        for msg in extract_visible_messages(conv):
            text = extract_text_content(msg)
            if query_lower in text.lower():
                results.append(conv)
                break

    return results
```

### Export Conversation to Markdown

```python
def export_to_markdown(conversation):
    lines = [f"# {conversation['title']}\n"]

    for msg in extract_visible_messages(conversation):
        role = msg['author']['role'].capitalize()
        text = extract_text_content(msg)
        lines.append(f"## {role}\n\n{text}\n")

    return '\n'.join(lines)
```

---

## Best Practices

1. **Check field existence** - Many fields are optional, use `.get()` with defaults
2. **Handle large files** - Use streaming for 200MB+ files if needed
3. **Preserve tree structure** - Contains branching/edit history information
4. **Handle encoding** - Ensure UTF-8 throughout (emojis, unicode, code blocks)
5. **Filter appropriately**:
   - User-facing: Hide system messages, filter by weight
   - Analysis: Keep all messages
   - Search/indexing: Extract text from visible messages only
