# knap-skill

A [Claude Code](https://claude.com/claude-code) skill for [Knap](https://knap.md), Kepano's template language that turns data into Markdown.

Not written from the docs. Every command in the skill was run against the Knap CLI (0.4.0), and it ships the gotchas that cost failed renders the first time round.

## Install

```sh
git clone https://github.com/remigius-labs/knap-skill
cp -r knap-skill/knap ~/.claude/skills/knap
```

Claude Code picks it up on the next session. Knap itself: `npm i -g knap` (Node 20+).

## What it covers

- `render` one note from a JSON object, `batch` one file per CSV row / JSON array item / JSON file in a folder
- Filename templates, `--dry-run`, `--overwrite`, stdin piping, Defuddle → Knap for URL-to-note
- Obsidian frontmatter, tags, wikilinks, callouts, and tables from data
- Every standard filter name (`knap/filters.md`), since unknown filter names are hard errors

## The gotchas

| Symptom | Fix |
|---|---|
| `UNKNOWN_FILTER: "default"` | Use `{{ x ?? "fallback" }}` |
| `tags: - "a"` broken YAML | `{{ tags \| list }}` on its own line under `tags:` |
| `["[[A]]","[[B]]"]` in output | `wikilink` on an array returns JSON, add `\| join:", "` |
| `PARSE_ERROR: Unexpected character '-'` | `{{-` / `-}}` trim is not supported in the CLI |
| Stray blank lines around loops | Put each `{% %}` tag on its own line, Knap removes that line |
| Time shifted by hours | `date` renders in local timezone, feed date-only strings |

## Example

```liquid
---
type: lead
date: {{ date | date:"YYYY-MM-DD" }}
status: {{ status | lower }}
tags:
{{ tags | list }}
---
# {{ name }}

Channel: {{ channel ?? "unknown" }}
{% for n in notes %}
- {{ n }}
{% endfor %}
Related: {{ people | wikilink | join:", " }}
```

```sh
knap batch lead.md --data leads.json --output-dir notes \
  --filename '{{ date | date:"YYYY-MM-DD" }}-{{ name | safe_name | kebab }}.md' --dry-run
```

Output, `notes/2026-09-10-ada.md`:

```markdown
---
type: lead
date: 2026-09-10
status: open
tags:
- lead
- friend
---
# Ada

Channel: unknown
- First call done
- Sent the deck
Related: [[Ada]], [[Lovelace Labs]]
```

## How it was built

Test-driven: a fresh agent with no skill could not produce a single Knap command. The skill was written from real runs, then a fresh agent with the skill rendered a batch correctly first try. Audited with a NASA Power of Ten pass on the executable content: 10/10.

MIT.
