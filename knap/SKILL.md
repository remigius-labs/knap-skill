---
name: knap
description: Use when generating Markdown or Obsidian notes from structured data (JSON, CSV, a folder of JSON files) with a template, or when the user mentions Knap, knap.md, Kepano's template language, "render notes from data", "one note per record", frontmatter from JSON, or Obsidian Web Clipper templates.
---

# Knap CLI

Knap is Kepano's Markdown template engine (npm `knap`, MIT, powers Obsidian Web Clipper and Importer). Data in, Markdown out. Syntax is Twig/Liquid-style. `knap --help` is the flag reference; this skill is the parts that bite.

## When to use

- One note from one JSON object (`render`)
- One file per record from a CSV, a JSON array, or a folder of JSON files (`batch`)
- Frontmatter, wikilinks, callouts, tables for Obsidian from data
- Turning a saved link into a note: `npx defuddle parse URL --markdown --json | knap render tpl.md --data -`

Not for: HTML DOM filters (`html_to_json`, `remove_html`, library API only), TSV, non-object JSON.

## Setup

```sh
knap --version || npm i -g knap   # Node 20+
```

## Commands

```sh
knap render tpl.md --data one.json --output note.md          # one note
cat one.json | knap render tpl.md --data -                   # pipe data (one stdin only)
knap render -t '# {{ title }}' --set title=Hi                # inline, --set = strings only
knap batch tpl.md --data recs.json --output-dir notes \
  --filename '{{ date | date:"YYYY-MM-DD" }}-{{ name | safe_name | kebab }}.md' --dry-run
knap batch tpl.md --data ./json-folder --output-dir notes --overwrite   # keeps basenames
```

In scripts use `set -euo pipefail`: knap exits 1 on any error and the pipe examples otherwise mask a failed `cat`. Always `--dry-run` a batch first. Batch validates everything before writing; duplicate filenames fail even with `--overwrite`. Errors exit 1, print `file:line:col code`, and leave `--output` untouched.

## Template syntax (verified on 0.4.0)

```liquid
{{ title }}  {{ author.name }}  {{ rows.0.k }}  {{ rows[1].v }}
{{ title | trim | upper }}          {{ published | date:"YYYY-MM-DD" }}
{{ channel ?? "unknown" }}          {# there is NO default filter, use ?? #}
{% if status == "OPEN" %}…{% elseif x %}…{% else %}…{% endif %}
{% for p in people %}- {{ p }}
{% endfor %}
{% set h = title | upper %}{{ h }}
```

## Obsidian frontmatter and links

```liquid
---
type: lead
date: {{ date | date:"YYYY-MM-DD" }}
status: {{ status | lower }}
tags:
{{ tags | list }}
---
Related: {{ people | wikilink | join:", " }}
{{ "text" | callout:"warning" }}
{{ rows | table }}
```

## Gotchas (each one cost a failed run)

| Symptom | Fix |
|---|---|
| `UNKNOWN_FILTER: "default"` | Use `{{ x ?? "fallback" }}` |
| `tags: - "a"` broken YAML | Put `{{ tags \| list }}` on its own line under `tags:`, or `{{ tags \| yaml_property:"tags" }}` |
| `["[[A]]","[[B]]"]` in output | `wikilink` on an array returns JSON, add `\| join:", "` |
| `PARSE_ERROR: Unexpected character '-'` | `{{-`/`-}}` whitespace trim is not supported in the CLI, drop the dashes |
| Stray blank lines around loops/ifs | Put each `{% %}` tag on its own line: Knap removes that whole line. Inline tags keep surrounding text as-is |
| Time shifted by hours | `date` renders in local timezone, feed date-only strings for dates |
| `--set enabled=false` is truthy | `--set` values are strings, use JSON data for booleans/numbers |
| Batch wrote nothing | A single bad record fails the whole batch, read the `record N` in the error |

Filter names: `knap` has no list command, see `filters.md` here. Unknown filter names are errors, not passthrough.
