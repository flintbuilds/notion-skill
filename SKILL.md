---
name: notion
description: Complete Notion API skill for managing pages, databases, and content with practical workflows
homepage: https://developers.notion.com
metadata: {"openclaw":{"emoji":"📝","category":"productivity","tags":["productivity","databases","notes","task-management","knowledge-base"]}}
---

# Notion API Skill

Complete guide for using the Notion API to create, read, update, and manage pages, databases (data sources), and blocks.

## Quick Setup

```bash
# 1. Store your API key
mkdir -p ~/.config/notion
echo "ntn_your_key_here" > ~/.config/notion/api_key
chmod 600 ~/.config/notion/api_key

# 2. Test connection
NOTION_KEY=$(cat ~/.config/notion/api_key)
curl -s "https://api.notion.com/v1/users/me" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" | jq .
```

> **Important:** Share your Notion pages/databases with the integration before using the API. Click "..." → "Connect to" → your integration name.

---

## API Basics

All requests require these headers:
```bash
NOTION_KEY=$(cat ~/.config/notion/api_key)
curl -X GET "https://api.notion.com/v1/..." \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json"
```

---

## Core Operations

### Search
```bash
# Search everything
curl -X POST "https://api.notion.com/v1/search" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{"query": "meeting notes"}'

# Search only databases
curl -X POST "https://api.notion.com/v1/search" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{"query": "tasks", "filter": {"property": "object", "value": "database"}}'

# Search only pages
curl -X POST "https://api.notion.com/v1/search" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{"query": "project", "filter": {"property": "object", "value": "page"}}'
```

### Pages

**Get a page:**
```bash
curl "https://api.notion.com/v1/pages/{page_id}" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28"
```

**Get page content (blocks):**
```bash
curl "https://api.notion.com/v1/blocks/{page_id}/children" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28"
```

**Create a page under another page:**
```bash
curl -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {"page_id": "PARENT_PAGE_ID"},
    "properties": {
      "title": {"title": [{"text": {"content": "New Page Title"}}]}
    },
    "children": [
      {"object": "block", "type": "heading_2", "heading_2": {"rich_text": [{"text": {"content": "Introduction"}}]}},
      {"object": "block", "type": "paragraph", "paragraph": {"rich_text": [{"text": {"content": "Your content here."}}]}}
    ]
  }'
```

**Create a page in a database:**
```bash
curl -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {"database_id": "DATABASE_ID"},
    "properties": {
      "Name": {"title": [{"text": {"content": "Task Name"}}]},
      "Status": {"select": {"name": "To Do"}},
      "Priority": {"select": {"name": "High"}},
      "Due Date": {"date": {"start": "2024-12-31"}}
    }
  }'
```

**Update page properties:**
```bash
curl -X PATCH "https://api.notion.com/v1/pages/{page_id}" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{"properties": {"Status": {"select": {"name": "Done"}}}}'
```

**Archive (soft delete) a page:**
```bash
curl -X PATCH "https://api.notion.com/v1/pages/{page_id}" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{"archived": true}'
```

### Databases

**Get database schema:**
```bash
curl "https://api.notion.com/v1/databases/{database_id}" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28"
```

**Query database (all items):**
```bash
curl -X POST "https://api.notion.com/v1/databases/{database_id}/query" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{}'
```

**Query with filters and sorting:**
```bash
curl -X POST "https://api.notion.com/v1/databases/{database_id}/query" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "filter": {
      "and": [
        {"property": "Status", "select": {"equals": "In Progress"}},
        {"property": "Priority", "select": {"equals": "High"}}
      ]
    },
    "sorts": [
      {"property": "Due Date", "direction": "ascending"}
    ],
    "page_size": 100
  }'
```

**Create a database:**
```bash
curl -X POST "https://api.notion.com/v1/databases" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {"page_id": "PARENT_PAGE_ID"},
    "title": [{"text": {"content": "Project Tasks"}}],
    "is_inline": true,
    "properties": {
      "Name": {"title": {}},
      "Status": {"select": {"options": [
        {"name": "To Do", "color": "gray"},
        {"name": "In Progress", "color": "blue"},
        {"name": "Done", "color": "green"}
      ]}},
      "Priority": {"select": {"options": [
        {"name": "Low", "color": "gray"},
        {"name": "Medium", "color": "yellow"},
        {"name": "High", "color": "red"}
      ]}},
      "Due Date": {"date": {}},
      "Assignee": {"rich_text": {}},
      "Notes": {"rich_text": {}}
    }
  }'
```

### Blocks

**Add content to a page:**
```bash
curl -X PATCH "https://api.notion.com/v1/blocks/{page_id}/children" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "children": [
      {"object": "block", "type": "heading_1", "heading_1": {"rich_text": [{"text": {"content": "Main Heading"}}]}},
      {"object": "block", "type": "paragraph", "paragraph": {"rich_text": [{"text": {"content": "Regular paragraph text."}}]}},
      {"object": "block", "type": "bulleted_list_item", "bulleted_list_item": {"rich_text": [{"text": {"content": "First bullet"}}]}},
      {"object": "block", "type": "bulleted_list_item", "bulleted_list_item": {"rich_text": [{"text": {"content": "Second bullet"}}]}},
      {"object": "block", "type": "to_do", "to_do": {"rich_text": [{"text": {"content": "Task item"}}], "checked": false}},
      {"object": "block", "type": "code", "code": {"rich_text": [{"text": {"content": "console.log(\"Hello\")"}}], "language": "javascript"}},
      {"object": "block", "type": "callout", "callout": {"rich_text": [{"text": {"content": "Important note!"}}], "icon": {"emoji": "💡"}}}
    ]
  }'
```

**Update a block:**
```bash
curl -X PATCH "https://api.notion.com/v1/blocks/{block_id}" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{"paragraph": {"rich_text": [{"text": {"content": "Updated text"}}]}}'
```

**Delete a block:**
```bash
curl -X DELETE "https://api.notion.com/v1/blocks/{block_id}" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28"
```

---

## Property Types Reference

| Type | Create/Update Format | Notes |
|------|---------------------|-------|
| **Title** | `{"title": [{"text": {"content": "..."}}]}` | Required, one per database |
| **Rich Text** | `{"rich_text": [{"text": {"content": "..."}}]}` | Multi-line text |
| **Number** | `{"number": 42}` | Integers or decimals |
| **Select** | `{"select": {"name": "Option"}}` | Single choice |
| **Multi-select** | `{"multi_select": [{"name": "A"}, {"name": "B"}]}` | Multiple choices |
| **Date** | `{"date": {"start": "2024-01-15"}}` | ISO 8601 format |
| **Date Range** | `{"date": {"start": "2024-01-15", "end": "2024-01-20"}}` | With end date |
| **Checkbox** | `{"checkbox": true}` | Boolean |
| **URL** | `{"url": "https://example.com"}` | Valid URL |
| **Email** | `{"email": "user@example.com"}` | Valid email |
| **Phone** | `{"phone_number": "+1234567890"}` | Any format |
| **Relation** | `{"relation": [{"id": "page_id"}]}` | Link to other pages |
| **People** | `{"people": [{"id": "user_id"}]}` | Notion users |
| **Files** | `{"files": [{"name": "file.pdf", "external": {"url": "..."}}]}` | External URLs only via API |
| **Formula** | Read-only | Computed from other properties |
| **Rollup** | Read-only | Aggregates from relations |
| **Created time** | Read-only | Auto-set |
| **Last edited time** | Read-only | Auto-set |

---

## Filter Operators

### Text filters (title, rich_text, url, email, phone)
```json
{"property": "Name", "rich_text": {"equals": "exact match"}}
{"property": "Name", "rich_text": {"contains": "partial"}}
{"property": "Name", "rich_text": {"starts_with": "prefix"}}
{"property": "Name", "rich_text": {"ends_with": "suffix"}}
{"property": "Name", "rich_text": {"is_empty": true}}
{"property": "Name", "rich_text": {"is_not_empty": true}}
```

### Number filters
```json
{"property": "Count", "number": {"equals": 5}}
{"property": "Count", "number": {"greater_than": 10}}
{"property": "Count", "number": {"less_than_or_equal_to": 100}}
```

### Select filters
```json
{"property": "Status", "select": {"equals": "Done"}}
{"property": "Status", "select": {"does_not_equal": "Cancelled"}}
{"property": "Status", "select": {"is_empty": true}}
```

### Multi-select filters
```json
{"property": "Tags", "multi_select": {"contains": "urgent"}}
{"property": "Tags", "multi_select": {"does_not_contain": "archived"}}
```

### Date filters
```json
{"property": "Due", "date": {"equals": "2024-01-15"}}
{"property": "Due", "date": {"before": "2024-01-01"}}
{"property": "Due", "date": {"after": "2024-01-01"}}
{"property": "Due", "date": {"on_or_before": "2024-01-15"}}
{"property": "Due", "date": {"past_week": {}}}
{"property": "Due", "date": {"next_month": {}}}
```

### Checkbox filters
```json
{"property": "Complete", "checkbox": {"equals": true}}
```

### Compound filters
```json
{
  "and": [
    {"property": "Status", "select": {"equals": "Active"}},
    {"or": [
      {"property": "Priority", "select": {"equals": "High"}},
      {"property": "Due", "date": {"before": "2024-02-01"}}
    ]}
  ]
}
```

---

## Opinionated Workflow Patterns

These are battle-tested patterns for AI assistants managing tasks in Notion.

### Task Lifecycle (Recommended)

When given a task:
1. **Create task** in your Tasks database with status "In Progress"
2. **Do the work**
3. **On completion:**
   - Mark original task → "Done"
   - Create a **Review task**: "Review: [Original Task Name]"
   - Include a **clickable link** to the deliverable in Notes
   - Set Review task status to "Not Started"

This creates an audit trail and ensures deliverables get reviewed.

**Example — Creating a review task with clickable link:**
```bash
curl -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {"database_id": "YOUR_TASKS_DB_ID"},
    "properties": {
      "Name": {"title": [{"text": {"content": "Review: API Integration Complete"}}]},
      "Status": {"select": {"name": "Not Started"}},
      "Priority": {"select": {"name": "Medium"}},
      "Notes": {"rich_text": [{"text": {"content": "Deliverable: ", "link": null}}, {"text": {"content": "API Documentation", "link": {"url": "https://notion.so/your-page-id"}}}]}
    }
  }'
```

### Side Quests Pattern

For quick research or tangential tasks:
1. Create a subpage under a "Side Quests" parent page
2. Still create a tracker task (for visibility)
3. Still create a review task when done
4. Keeps main task list clean while tracking everything

### Formatting Rules

For clean, consistent Notion content:

| Rule | Why |
|------|-----|
| **Bullets over tables** | Better mobile rendering, easier to edit |
| **No emojis in body content** | Cleaner look, emojis only for page icons |
| **Clickable links always** | Use `{"text": {"content": "Link Text", "link": {"url": "..."}}}` |
| **Collapsible sections for long content** | Use toggle blocks for details |
| **Consistent property naming** | Title case: "Due Date" not "due_date" |

**Making links clickable in rich_text:**
```json
{
  "rich_text": [
    {"text": {"content": "See the "}},
    {"text": {"content": "full report", "link": {"url": "https://example.com/report"}}},
    {"text": {"content": " for details."}}
  ]
}
```

### Preference/Config Tracking

If your agent learns user preferences:
1. Store in a "System Config" database
2. Properties: Setting Name, Value, Category, Last Updated
3. Query on startup to load preferences
4. Update when user states new preferences

---

## Common Workflows

### Task Tracking

**Get open tasks due this week:**
```bash
curl -X POST "https://api.notion.com/v1/databases/{database_id}/query" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "filter": {
      "and": [
        {"property": "Status", "select": {"does_not_equal": "Done"}},
        {"property": "Due Date", "date": {"this_week": {}}}
      ]
    },
    "sorts": [{"property": "Due Date", "direction": "ascending"}]
  }'
```

**Create a quick task:**
```bash
curl -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {"database_id": "YOUR_TASKS_DB_ID"},
    "properties": {
      "Name": {"title": [{"text": {"content": "Review PR #123"}}]},
      "Status": {"select": {"name": "To Do"}},
      "Priority": {"select": {"name": "High"}},
      "Due Date": {"date": {"start": "'$(date -v+1d +%Y-%m-%d)'"}}
    }
  }'
```

### Knowledge Base

**Add a new article:**
```bash
curl -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {"database_id": "YOUR_KB_DB_ID"},
    "properties": {
      "Title": {"title": [{"text": {"content": "How to Configure X"}}]},
      "Category": {"select": {"name": "Technical"}},
      "Tags": {"multi_select": [{"name": "setup"}, {"name": "configuration"}]},
      "Status": {"select": {"name": "Draft"}}
    },
    "children": [
      {"object": "block", "type": "heading_2", "heading_2": {"rich_text": [{"text": {"content": "Overview"}}]}},
      {"object": "block", "type": "paragraph", "paragraph": {"rich_text": [{"text": {"content": "This guide explains..."}}]}},
      {"object": "block", "type": "heading_2", "heading_2": {"rich_text": [{"text": {"content": "Steps"}}]}},
      {"object": "block", "type": "numbered_list_item", "numbered_list_item": {"rich_text": [{"text": {"content": "First, do this"}}]}},
      {"object": "block", "type": "numbered_list_item", "numbered_list_item": {"rich_text": [{"text": {"content": "Then, do that"}}]}}
    ]
  }'
```

### Meeting Notes

**Create meeting notes template:**
```bash
curl -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {"database_id": "YOUR_MEETINGS_DB_ID"},
    "properties": {
      "Title": {"title": [{"text": {"content": "Team Sync - '"$(date +%Y-%m-%d)"'"}}]},
      "Date": {"date": {"start": "'"$(date +%Y-%m-%d)"'"}},
      "Attendees": {"multi_select": [{"name": "Team"}]},
      "Type": {"select": {"name": "Recurring"}}
    },
    "children": [
      {"object": "block", "type": "heading_2", "heading_2": {"rich_text": [{"text": {"content": "Attendees"}}]}},
      {"object": "block", "type": "bulleted_list_item", "bulleted_list_item": {"rich_text": [{"text": {"content": ""}}]}},
      {"object": "block", "type": "heading_2", "heading_2": {"rich_text": [{"text": {"content": "Agenda"}}]}},
      {"object": "block", "type": "numbered_list_item", "numbered_list_item": {"rich_text": [{"text": {"content": ""}}]}},
      {"object": "block", "type": "heading_2", "heading_2": {"rich_text": [{"text": {"content": "Notes"}}]}},
      {"object": "block", "type": "paragraph", "paragraph": {"rich_text": [{"text": {"content": ""}}]}},
      {"object": "block", "type": "heading_2", "heading_2": {"rich_text": [{"text": {"content": "Action Items"}}]}},
      {"object": "block", "type": "to_do", "to_do": {"rich_text": [{"text": {"content": ""}}], "checked": false}}
    ]
  }'
```

### CRM / Contact Management

**Add a new contact:**
```bash
curl -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {"database_id": "YOUR_CONTACTS_DB_ID"},
    "properties": {
      "Name": {"title": [{"text": {"content": "Jane Smith"}}]},
      "Company": {"rich_text": [{"text": {"content": "Acme Corp"}}]},
      "Email": {"email": "jane@acme.com"},
      "Phone": {"phone_number": "+1-555-0123"},
      "Status": {"select": {"name": "Lead"}},
      "Source": {"select": {"name": "Website"}},
      "Tags": {"multi_select": [{"name": "enterprise"}, {"name": "priority"}]}
    }
  }'
```

---

## Pagination

Results are paginated (max 100 items). Handle it like this:

```bash
# First request
response=$(curl -s -X POST "https://api.notion.com/v1/databases/{database_id}/query" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{"page_size": 100}')

# Check for more pages
has_more=$(echo "$response" | jq -r '.has_more')
next_cursor=$(echo "$response" | jq -r '.next_cursor')

# Get next page if exists
if [ "$has_more" = "true" ]; then
  curl -X POST "https://api.notion.com/v1/databases/{database_id}/query" \
    -H "Authorization: Bearer $NOTION_KEY" \
    -H "Notion-Version: 2022-06-28" \
    -H "Content-Type: application/json" \
    -d '{"page_size": 100, "start_cursor": "'"$next_cursor"'"}'
fi
```

---

## Troubleshooting

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `401 Unauthorized` | Invalid or expired API key | Regenerate key at notion.so/my-integrations |
| `403 Forbidden` | Page not shared with integration | Click "..." → "Connect to" on the page |
| `404 Not Found` | Invalid ID or page deleted | Verify the ID exists and is accessible |
| `400 Bad Request` | Malformed JSON or invalid property | Check property names match schema exactly |
| `409 Conflict` | Concurrent edit collision | Retry the request |
| `429 Rate Limited` | Too many requests | Wait and retry with exponential backoff |

### Debugging Tips

**Test API key:**
```bash
curl -s "https://api.notion.com/v1/users/me" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" | jq .
```

**Check database schema (property names are case-sensitive!):**
```bash
curl -s "https://api.notion.com/v1/databases/{database_id}" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" | jq '.properties | keys'
```

**Validate page exists and is accessible:**
```bash
curl -s "https://api.notion.com/v1/pages/{page_id}" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" | jq '{id, archived, parent}'
```

### Rate Limits
- Average: ~3 requests/second
- Burst: Higher for short periods
- When rate limited, wait 1 second and retry with exponential backoff

---

## Notes & Limitations

- **Property names are case-sensitive** — must match exactly
- **Page/database IDs** — UUIDs work with or without dashes
- **View filters can't be set via API** — that's Notion UI only
- **Files** — API can only add external URL files, not upload
- **Comments** — Supported via separate `/comments` endpoint
- **Formulas/Rollups** — Read-only, computed automatically
- **Rich text** — Supports formatting, links, mentions in the `annotations` field
- **Block children** — Some blocks (toggle, callout) can have nested children
