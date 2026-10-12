# data/entries.yml — schema and workflow

`data/entries.yml` is the source of truth for this repository's resource lists.
Category pages (`docs/categories/*/readme.md`) and the browse site are
**generated** from it by `scripts/generate.py` (stdlib only, no dependencies).

## Workflow

1. Edit `data/entries.yml` (add / update / remove entries).
2. Run `python3 scripts/generate.py` from the repo root.
   - Regenerates `docs/categories/*/readme.md` in place.
   - Builds the static site into `site/` (gitignored; deployed by CI).
3. Commit `data/entries.yml` **and** the regenerated category pages together.

CI (`validate-generated.yml`) fails the PR if the committed pages differ from
what the script generates. The browse site is deployed to GitHub Pages from
`main` by `pages.yml`.

## Schema

```yaml
categories:
  - slug: ide-and-cli            # required, unique; matches docs/categories/<slug>/
    title: IDE and CLI Tools     # required; page heading
    description: "..."           # optional; shown under the heading
    image: ide-and-cli.webp      # optional; docs/categories/<slug>/<image>
    image_alt: ide-and-cli       # optional; alt text for the image
    layout: table                # required: table | sections | list
    sections:                    # required for layout: table
      - name: IDE                # omit `name` for a single untitled table
        format: table            # table (default) or prose
        fields: [name, summary, source]
        headers: [Tool, Main Features & Models Supported, Link]
        name_bold: false         # render names as **bold** (default false)
        subsections:             # optional ### sub-tables
          - name: ChatGPT API (Free)
            fields: [name, summary, url]
            headers: [Repository Name, Description, URL]

entries:
  - name: Cursor                 # required
    category: ide-and-cli        # required; must match a category slug
    section: IDE                 # required for layout: table (null if untitled)
    subsection: ~                # optional; must match a subsection name
    summary: "..."               # short description / notes
    url: https://...             # primary link
    url_label: Cursor            # optional; renders source cell as [label](url)
    extra:                       # optional; extra table columns (fields: extra.*)
      usage: "..."
      cli_native: "Yes"
    tags: [ide]                  # optional; used by the site's tag filter
```

### Field reference (`fields` values)

| field          | renders as                                    |
| -------------- | --------------------------------------------- |
| `name`         | entry name (`**bold**` when `name_bold: true`) |
| `summary`      | description text (`-` when empty)              |
| `source`       | `[url_label](url)`, bare url, or `-`           |
| `url`          | bare url                                      |
| `extra.<key>`  | value of `extra.<key>` (empty when missing)   |

### Layouts

- `table` — one markdown table per section (entries carry `section`).
- `sections` — each entry becomes a `## Name` block with summary + url.
- `list` — each entry becomes a `- <url>` bullet.

### Notes

- Keep `summary` on a single line; the YAML parser supports only
  single-line values (no `|` / `>` block scalars).
- Quote values containing `: ` or leading special characters.
- `tags` should be lowercase slugs; they power the site's tag filter.
- The generator validates the file and reports problems with entry numbers.
