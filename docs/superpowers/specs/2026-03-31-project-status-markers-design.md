# Project Status Markers Redesign

## Problem

The current status system uses two emoji markers (`:sleepy:` for 1 year inactive, `:sleeping:` for 2+ years inactive) and one text badge (`Deprecated`) applied to a single project (Scribe). This system is insufficient:

- Projects inactive for 2 years and 10 years look identical (both get `:sleeping:`)
- Only one project is marked `Deprecated` despite others being archived (e.g., UEditor)
- The distinction between "sleepy" and "sleeping" is subtle and requires checking the legend

## Design

### New Status Markers

Replace the existing emoji-only system with two combined badge + emoji markers:

| Marker | Rendered | Criteria |
|--------|----------|----------|
| `` `⛔ Deprecated` `` | `⛔ Deprecated` | Repo archived on GitHub, OR README/description explicitly says deprecated/unmaintained |
| `` `💤 Inactive` `` | `💤 Inactive` | No commits for at least 1 year, repo still open, not explicitly abandoned |

Projects without a marker are considered **active**.

### Marker Placement

Markers go at the end of the description text, before any star badges (in the Chinese README) or at the end of the line (in the English README). This matches the existing position of `:sleeping:` and `:sleepy:` emojis.

**English README example:**
```markdown
- [Pell](https://github.com/jaredreich/pell) - The simplest and smallest (1kB) WYSIWYG text editor for web, with no dependencies. `💤 Inactive`
```

**Chinese README example:**
```markdown
- [Pell](https://github.com/jaredreich/pell) - 最简单和最小的（1kB）Web WYSIWYG 文本编辑器，没有任何依赖性。 `💤 Inactive` ![github star](...)
```

### Icons/Legend Section

Replace the current legend:

```markdown
## Icons

⛔ Deprecated — project is archived or explicitly unmaintained
💤 Inactive — no updates for at least 1 year
```

### URL Validation

As part of implementation, every URL in both READMEs must be checked for:
- HTTP response status (200, 301/302 redirects, 404, etc.)
- Whether GitHub repos are archived (indicates `⛔ Deprecated`)
- Broken or dead links that need updating or flagging

### Scope of Changes

**Files modified:**
1. `README.md` — replace all `:sleeping:` and `:sleepy:` emojis with appropriate new markers; update the Icons section
2. `readme-zh-CN.md` — apply same markers (note: Chinese README currently has no status markers, they should be added for consistency)

**What stays the same:**
- No projects are removed
- No reordering of entries
- `$ Non-Free` and `⊘ Proprietary` markers are untouched
- Project descriptions remain unchanged

### Classification Approach

For each project:
1. Check if the GitHub repo URL is valid/accessible
2. If the repo is archived or description says deprecated/unmaintained -> `⛔ Deprecated`
3. If the repo's last commit is >= 1 year ago and it's not deprecated -> `💤 Inactive`
4. Otherwise -> no marker (active)

### Known Deprecated Projects

Based on current knowledge:
- **Scribe** — already marked Deprecated, repo archived
- **UEditor** — repo archived on GitHub

Additional deprecated projects will be identified during URL validation.

### Known Inactive Projects (currently marked with emojis)

All projects currently marked `:sleeping:` or `:sleepy:` will be verified and reclassified. The current `:sleeping:` projects span the English README across all sections. The Chinese README has no markers but should receive them.
