# Knap standard filters (CLI, 0.4.0)

Format: h1..h6, bold, italic, strike, code, code_block, link, image, wikilink, list, table, table_pretty, blockquote, callout:"type", embed, comment, hr, hard_break, highlight, escape_md, math, math_block, footnote, yaml, yaml_property:"key"
Text: upper, lower, capitalize, title, camel, pascal, kebab, snake, uncamel, trim, indent, truncate, truncatewords, replace, safe_name, encode_uri, decode_uri, unescape
Date: date:"FMT" (moment-style tokens, local tz), date_modify, duration
Number: calc, round, number_format
Collection: length, first, last, nth, slice, join:"sep", split, reverse, sort, unique, compact, merge, map, where, sum, template, object, parse_json
HTML cleanup (string-based, CLI ok): strip_tags, strip_attr, remove_tags, remove_attr, replace_tags, strip_md
DOM-only (library API, not CLI): html_to_json, remove_html

No `default` filter. Fallback is `{{ x ?? "value" }}`.
Reference: https://knap.md/filters and https://knap.md/cli
