# 📝 Notion Skill for Clawdbot

A comprehensive Notion API skill that enables Claude/Clawdbot to manage your Notion workspace—create pages, query databases, add content, and automate workflows.

## What It Does

This skill provides Claude with the knowledge to:

- **Search** pages and databases in your workspace
- **Create** new pages, database entries, and content blocks
- **Read** page content, database schemas, and query results
- **Update** page properties and block content
- **Delete/Archive** pages and blocks
- **Query** databases with filters, sorts, and pagination

### Common Use Cases

| Use Case | Example |
|----------|---------|
| **Task Management** | "Create a task called 'Review docs' due Friday" |
| **Knowledge Base** | "Add an article about deployment to the wiki" |
| **Meeting Notes** | "Create meeting notes for today's standup" |
| **CRM** | "Add Jane Smith from Acme Corp as a new lead" |
| **Research** | "Query all articles tagged 'AI' from last month" |

---

## Setup Instructions

### 1. Create a Notion Integration

1. Go to [notion.so/my-integrations](https://www.notion.so/my-integrations)
2. Click **"+ New integration"**
3. Give it a name (e.g., "Clawdbot")
4. Select the workspace it should access
5. Click **Submit**
6. Copy the **Internal Integration Secret** (starts with `ntn_` or `secret_`)

### 2. Store Your API Key

```bash
# Create config directory
mkdir -p ~/.config/notion

# Save your API key
echo "ntn_your_key_here" > ~/.config/notion/api_key

# Secure it (optional but recommended)
chmod 600 ~/.config/notion/api_key
```

### 3. Share Pages with Your Integration

**This step is required!** The API can only access pages explicitly shared with your integration.

1. Open any Notion page or database you want to access
2. Click the **"..."** menu in the top right
3. Click **"Connect to"** (or "Add connections")
4. Select your integration name
5. Confirm the connection

> **Tip:** Share a parent page to give access to all its children. Share your workspace's root pages for broad access.

### 4. Test Your Setup

```bash
NOTION_KEY=$(cat ~/.config/notion/api_key)
curl -s "https://api.notion.com/v1/users/me" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" | jq .
```

You should see your integration's details. If you get an error, double-check your API key.

---

## Usage Examples

### Basic Operations

**Search for a page:**
```
"Find my Project Tracker database"
"Search for pages about marketing"
```

**Create a page:**
```
"Create a new page called 'Q4 Planning' under my Projects page"
"Add a task called 'Review PR' with high priority, due tomorrow"
```

**Query a database:**
```
"Show me all open tasks due this week"
"Find contacts tagged 'enterprise' who haven't been contacted"
```

**Update content:**
```
"Mark the 'Fix bug' task as done"
"Add a note to yesterday's meeting notes"
```

### Workflow Examples

**Daily standup:**
```
"Create today's standup notes with sections for Yesterday, Today, and Blockers"
```

**Research collection:**
```
"Save this article to my Reading List with tags 'AI' and 'productivity'"
```

**Project kickoff:**
```
"Create a new project called 'Website Redesign' with a tasks database and meeting notes section"
```

---

## Common Workflows

### Task Tracking Database

Ideal schema for a task tracker:

| Property | Type | Purpose |
|----------|------|---------|
| Name | Title | Task name |
| Status | Select | To Do / In Progress / Done |
| Priority | Select | Low / Medium / High |
| Due Date | Date | Deadline |
| Assignee | People or Text | Who's responsible |
| Project | Relation | Link to project |
| Tags | Multi-select | Categories |

### Knowledge Base

Great for documentation and wikis:

| Property | Type | Purpose |
|----------|------|---------|
| Title | Title | Article name |
| Category | Select | Topic area |
| Tags | Multi-select | Keywords |
| Status | Select | Draft / Review / Published |
| Last Updated | Last edited time | Auto-tracked |
| Author | People | Creator |

### CRM / Contacts

Simple contact management:

| Property | Type | Purpose |
|----------|------|---------|
| Name | Title | Contact name |
| Company | Text | Organization |
| Email | Email | Email address |
| Phone | Phone | Phone number |
| Status | Select | Lead / Active / Closed |
| Last Contact | Date | When you last spoke |
| Notes | Text | Context |

---

## Troubleshooting

### "403 Forbidden" Error

**Cause:** The page isn't shared with your integration.

**Fix:** Open the page in Notion → Click "..." → "Connect to" → Select your integration.

### "401 Unauthorized" Error

**Cause:** Invalid or expired API key.

**Fix:** 
1. Go to [notion.so/my-integrations](https://www.notion.so/my-integrations)
2. Click your integration
3. Regenerate the secret
4. Update `~/.config/notion/api_key`

### "400 Bad Request" Error

**Cause:** Property name mismatch or malformed data.

**Fix:** Property names are **case-sensitive**. Check the exact names:
```bash
curl -s "https://api.notion.com/v1/databases/{id}" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" | jq '.properties | keys'
```

### "429 Rate Limited" Error

**Cause:** Too many requests (limit ~3/second average).

**Fix:** Wait a moment and retry. The API allows bursts but enforces sustained limits.

### Can't Find a Page

**Possible causes:**
1. Page not shared with integration (most common)
2. Page was deleted or archived
3. Search query too specific

**Fix:** Try a broader search, or share the parent page with your integration.

---

## Skill File Location

After installation, the skill is available at:
```
skills/notion-public/SKILL.md
```

Reference it in your Clawdbot configuration to enable Notion capabilities.

---

## API Reference

For complete API documentation, see:
- [Notion API Docs](https://developers.notion.com/)
- [API Reference](https://developers.notion.com/reference)
- [Property Value Docs](https://developers.notion.com/reference/property-value-object)

---

## Contributing

Found an issue or want to improve this skill? Contributions welcome!

1. Fork the repository
2. Make your changes
3. Submit a pull request

---

## License

MIT License - Use freely in your own projects.

---

## Changelog

### v1.0.0
- Initial release
- Complete CRUD operations for pages, databases, blocks
- Property type reference
- Filter operators documentation
- Common workflow patterns
- Troubleshooting guide
