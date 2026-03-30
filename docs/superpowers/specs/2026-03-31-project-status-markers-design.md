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

The new system intentionally collapses the old two-tier time distinction (1 year vs 2 years) into a single `Inactive` marker. The key insight is that the meaningful distinction for users is **intent** (deprecated vs. inactive), not duration of inactivity. A project inactive for 1 year and one inactive for 10 years are both worth flagging — what matters more is whether the project has been explicitly abandoned or is simply not receiving updates.

### Marker Placement

Markers go at the end of the description text, before any star badges (in the Chinese README) or at the end of the line (in the English README). If a Chinese README entry has no star badge, the marker goes at the end of the line, same as the English README. This matches the existing position of `:sleeping:` and `:sleepy:` emojis.

**English README example:**
```markdown
- [Pell](https://github.com/jaredreich/pell) - The simplest and smallest (1kB) WYSIWYG text editor for web, with no dependencies. `💤 Inactive`
```

**Chinese README example:**
```markdown
- [Pell](https://github.com/jaredreich/pell) - 最简单和最小的（1kB）Web WYSIWYG 文本编辑器，没有任何依赖性。 `💤 Inactive` ![github star](...)
```

**Scribe special case:** Scribe currently has both a `` `Deprecated` `` text badge in its description AND a `:sleeping:` emoji at the end. Both must be removed and replaced with a single `` `⛔ Deprecated` `` marker at the end of the description text:
```markdown
- [Scribe](https://github.com/guardian/scribe) - A rich text editor framework for the web platform, with patches for browser inconsistencies and sensible defaults. `⛔ Deprecated`
```

### Icons/Legend Section

Replace the current legend in `README.md`. The section heading stays as `## Icons` to preserve the existing Table of Contents anchor.

```markdown
## Icons

⛔ Deprecated — project is archived or explicitly unmaintained
💤 Inactive — no updates for at least 1 year
```

Note: `$ Non-Free` and `⊘ Proprietary` markers are not currently documented in the legend and documenting them is out of scope for this change.

For `readme-zh-CN.md`, an Icons section must be **created** (it does not currently have one), inserted before the contribution section:

```markdown
## 图标

⛔ Deprecated — 项目已归档或明确不再维护
💤 Inactive — 至少1年没有更新
```

### Marker Language

Marker text remains in English (`` `⛔ Deprecated` `` and `` `💤 Inactive` ``) in both READMEs. This is an intentional choice — these are effectively labels/badges, and keeping them in English ensures consistency and recognizability across both versions. The Chinese Icons section provides translated descriptions.

### URL Validation

As part of implementation, every URL in both READMEs must be checked for:
- HTTP response status (200, 301/302 redirects, 404, etc.)
- Whether GitHub repos are archived (indicates `⛔ Deprecated`)
- Broken or dead links

**Handling results:**
- **Redirects (301/302):** Update the URL to the new location
- **404 / dead links:** Flag to the user for a decision (remove entry, find new URL, or add a note)
- **Archived repos:** Classify as `⛔ Deprecated`
- **Non-GitHub URLs** (e.g., Tui Editor at `ui.toast.com`, FlyonUI at `flyonui.com`, Flowbite at `flowbite.com`): Check for HTTP accessibility only. These cannot be checked for archived status — classify based on the linked content's own messaging if it indicates deprecation, otherwise leave unmarked or classify based on associated GitHub repo if one exists

**Rate limiting:** When checking GitHub URLs, respect API rate limits. Use sequential requests with reasonable delays, or use the GitHub API with authentication if available.

### Scope of Changes

**Files modified:**
1. `README.md` — replace all `:sleeping:` and `:sleepy:` emojis with appropriate new markers; update the Icons section
2. `readme-zh-CN.md` — add markers to all projects; create an Icons section

**Chinese README divergence:** The Chinese README contains a different set of projects than the English one (some projects appear in only one file). Markers should be applied to **all projects in each file independently** based on the classification criteria. Synchronizing the project lists between the two READMEs is out of scope for this change.

**Chinese README Table of Contents:** The ToC in `readme-zh-CN.md` must be updated to include the new `## 图标` section.

**What stays the same:**
- No projects are removed
- No reordering of entries
- `$ Non-Free` and `⊘ Proprietary` markers are untouched
- Project descriptions remain unchanged (except Scribe's inline `Deprecated` badge removal)

### Classification Approach

For each project:
1. Check if the GitHub repo URL is valid/accessible
2. If the repo is archived or description says deprecated/unmaintained -> `⛔ Deprecated`
3. If the repo's last commit is >= 1 year ago (from implementation date, 2026-03-31) and it's not deprecated -> `💤 Inactive`
4. Otherwise -> no marker (active)

### Currently Marked Projects (English README)

Projects currently carrying `:sleeping:` (2+ years inactive):

| Project | Section | Current Marker | Expected New |
|---------|---------|---------------|--------------|
| Content Tools | Standalone | `:sleeping:` | TBD via validation |
| grande.js | Standalone | `:sleeping:` | TBD via validation |
| Medium Editor | Standalone | `:sleeping:` | TBD via validation |
| Medium.js | Standalone | `:sleeping:` | TBD via validation |
| Mobiledoc Kit | Standalone | `:sleeping:` | TBD via validation |
| Pell | Standalone | `:sleeping:` | TBD via validation |
| Pen Editor | Standalone | `:sleeping:` | TBD via validation |
| Scribe | Standalone | `:sleeping:` + `Deprecated` | `⛔ Deprecated` |
| UEditor | Standalone | `:sleeping:` | `⛔ Deprecated` (archived) |
| wangEditor | Standalone | `:sleeping:` | TBD via validation |
| wysihtml | Standalone | `:sleeping:` | TBD via validation |
| bootstrap-wysiwyg | jQuery | `:sleeping:` | TBD via validation |
| Easyeditor | jQuery | `:sleeping:` | TBD via validation |
| jQuery-Notebook | jQuery | `:sleeping:` | TBD via validation |
| popline | jQuery | `:sleeping:` | TBD via validation |
| simditor | jQuery | `:sleeping:` | TBD via validation |
| textAngular | Angular | `:sleeping:` | TBD via validation |
| Dante II | React | `:sleeping:` | TBD via validation |
| Draft.js | React | `:sleeping:` | TBD via validation |
| react-mobiledoc-editor | React | `:sleeping:` | TBD via validation |
| react-quill | React | `:sleeping:` | TBD via validation |
| react-rte | React | `:sleeping:` | TBD via validation |
| react-summernote | React | `:sleeping:` | TBD via validation |
| vue-html5-editor | Vue | `:sleeping:` | TBD via validation |
| vue-mobiledoc-editor | Vue | `:sleeping:` | TBD via validation |
| vue-wysiwyg | Vue | `:sleeping:` | TBD via validation |
| vue-ckeditor5 | Vue | `:sleeping:` | TBD via validation |
| Vue2Editor | Vue | `:sleeping:` | TBD via validation |
| bootstrap-wysihtml5-rails | Ruby | `:sleeping:` | TBD via validation |
| bootsy | Ruby | `:sleeping:` | TBD via validation |
| ckeditor (rails) | Ruby | `:sleeping:` | TBD via validation |
| Mercury Editor | Ruby | `:sleeping:` | TBD via validation |
| EmojiOne Area | WYSIWYG-alike | `:sleeping:` | TBD via validation |
| last-draft | WYSIWYG-alike | `:sleeping:` | TBD via validation |
| Ory editor | WYSIWYG-alike | `:sleeping:` | TBD via validation |
| Sir Trevor | WYSIWYG-alike | `:sleeping:` | TBD via validation |
| woofmark | WYSIWYG-alike | `:sleeping:` | TBD via validation |
| ngx-wall | WYSIWYG-alike | `:sleeping:` | TBD via validation |

Projects currently carrying `:sleepy:` (1 year inactive):

| Project | Section | Current Marker | Expected New |
|---------|---------|---------------|--------------|
| Tui Editor | Standalone | `:sleepy:` | TBD via validation |
| Toast UI Editor | jQuery | `:sleepy:` | TBD via validation |

### Maintenance

This spec covers a one-time reclassification. Ongoing maintenance of markers (e.g., re-checking repos periodically) is out of scope but could be addressed in a future GitHub Action or scheduled review process.
