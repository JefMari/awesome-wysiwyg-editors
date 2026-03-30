# Project Status Markers Redesign — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the existing `:sleepy:`/`:sleeping:` emoji status system with clear `⛔ Deprecated` and `💤 Inactive` badge+emoji markers across both READMEs, informed by live URL validation of every listed project.

**Architecture:** Three-phase approach: (1) validate all URLs and classify each project, (2) apply markers to both README files, (3) update legends and ToC. Each phase builds on the previous.

**Tech Stack:** GitHub API (via `gh` CLI), HTTP requests (via `curl`), manual markdown editing.

**Spec:** `docs/superpowers/specs/2026-03-31-project-status-markers-design.md`

**Guard rails (do NOT):**
- Do not remove any project entries
- Do not reorder entries
- Do not modify `$ Non-Free` or `⊘ Proprietary` markers
- Do not change project descriptions (except removing Scribe's inline `Deprecated` badge)
- Marker text must stay in English (`⛔ Deprecated`, `💤 Inactive`) in BOTH READMEs — do not translate the badge text
- Expect approximately 38 currently-marked projects in the English README (36 `:sleeping:`, 2 `:sleepy:`)
- The Chinese README has ZERO existing status markers — Task 4 is adding markers from scratch, not replacing

**Parallelization note:** Tasks 1 and 2 can run in parallel (independent URL checks). Tasks 3 and 4 can also run in parallel (modify different files).

---

### Task 1: Validate all URLs in English README and classify projects

Extract every project URL from `README.md`, check accessibility, determine archived status, and check last commit date. Produce a classification list.

**Files:**
- Read: `README.md`
- Create: `docs/superpowers/plans/url-validation-results.md` (temporary working doc)

- [ ] **Step 1: Extract all GitHub repo URLs from README.md**

Use grep to pull all GitHub URLs from the file. There are approximately 90+ project entries across all sections.

- [ ] **Step 2: Check each GitHub URL for accessibility and archived status**

For each GitHub repo URL, use the GitHub API to check. Example command:
```bash
gh api repos/{owner}/{repo} --jq '{name: .full_name, archived: .archived, pushed_at: .pushed_at, description: .description}'
```

Classification logic:
- 404 response = broken link (flag separately)
- `archived: true` = `⛔ Deprecated`
- `pushed_at` before 2025-03-31 (1 year before implementation date) = `💤 Inactive`
- Otherwise = Active (no marker)

For non-GitHub URLs (Tui Editor at `ui.toast.com`, Flowbite at `flowbite.com`, FlyonUI at `flyonui.com`), use `curl -sI` to check HTTP status. If the non-GitHub project also has an associated GitHub repo listed elsewhere in the README (e.g., Tui Editor links to `ui.toast.com` but `nhnent/tui.editor` exists on GitHub), cross-reference with that repo's GitHub API data for classification.

Run checks sequentially to respect rate limits. Use `gh api` which handles auth automatically.

- [ ] **Step 3: Record results in url-validation-results.md**

Create a markdown table with columns: Project Name, URL, HTTP Status, Archived?, Last Push Date, Classification (`⛔ Deprecated` / `💤 Inactive` / Active / Broken).

- [ ] **Step 4: Handle redirected URLs**

If any URLs return 301/302 redirects, update them to the new location. Present the list of updated URLs to the user for awareness but no action needed — redirects are straightforward fixes.

- [ ] **Step 5: Flag broken URLs to the user**

If any URLs return 404 or are otherwise unreachable, present these to the user for decision: remove the entry, find a new URL, or add a note. Do NOT proceed with marker changes for broken-link entries until the user decides.

- [ ] **Step 6: Commit validation results**

```bash
git add docs/superpowers/plans/url-validation-results.md
git commit -m "docs: add URL validation results for status markers"
```

### Task 2: Validate all URLs in Chinese README and classify projects

Same process as Task 1 but for `readme-zh-CN.md`. The Chinese README has a different project list — some entries overlap, some are unique.

**Files:**
- Read: `readme-zh-CN.md`
- Modify: `docs/superpowers/plans/url-validation-results.md` (append Chinese-only entries)

- [ ] **Step 1: Extract all GitHub repo URLs from readme-zh-CN.md**

- [ ] **Step 2: Identify URLs unique to the Chinese README**

Compare the Chinese URL list against the English URL list (already checked in Task 1). Only check URLs that are new/unique to the Chinese file. Method: extract both URL lists and diff them, or collect Chinese URLs into a set and subtract the English set.

- [ ] **Step 3: Check unique URLs using same method as Task 1**

- [ ] **Step 4: Append Chinese-only results to url-validation-results.md**

- [ ] **Step 5: Handle redirected URLs (update to new location)**

- [ ] **Step 6: Flag any broken URLs to the user**

- [ ] **Step 7: Commit updated validation results**

```bash
git add docs/superpowers/plans/url-validation-results.md
git commit -m "docs: add Chinese README URL validation results"
```

### Task 3: Apply markers to English README

Using the classification results from Tasks 1-2, replace all old markers with new ones in `README.md`.

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Handle Scribe special case**

Replace:
```markdown
- [Scribe](https://github.com/guardian/scribe) - `Deprecated` A rich text editor framework for the web platform, with patches for browser inconsistencies and sensible defaults. :sleeping:
```
With:
```markdown
- [Scribe](https://github.com/guardian/scribe) - A rich text editor framework for the web platform, with patches for browser inconsistencies and sensible defaults. `⛔ Deprecated`
```

- [ ] **Step 2: Replace all `:sleeping:` markers with appropriate new markers**

For each project currently marked `:sleeping:`, replace the emoji with the correct classification from the validation results:
- If classified as Deprecated: replace `:sleeping:` with `` `⛔ Deprecated` ``
- If classified as Inactive: replace `:sleeping:` with `` `💤 Inactive` ``
- If classified as Active (unlikely but possible if repo recently revived): remove `:sleeping:` entirely

- [ ] **Step 3: Replace all `:sleepy:` markers with appropriate new markers**

Same as Step 2 but for the two `:sleepy:` entries (Tui Editor, Toast UI Editor).

- [ ] **Step 4: Add markers to previously unmarked projects that are now Inactive or Deprecated**

Some projects may not have had `:sleeping:` or `:sleepy:` but are now classified as Inactive or Deprecated based on URL validation. Add markers to these entries.

- [ ] **Step 5: Update any redirected/broken URLs identified in Task 1**

Apply any URL fixes the user approved in Task 1 Step 4.

- [ ] **Step 6: Update the Icons section**

Replace:
```markdown
## Icons

:sleepy: no updates for 1 year
:sleeping: no updates for 2 years
```
With:
```markdown
## Icons

⛔ Deprecated — project is archived or explicitly unmaintained
💤 Inactive — no updates for at least 1 year
```

- [ ] **Step 7: Verify all old markers are removed**

Search `README.md` for any remaining `:sleeping:`, `:sleepy:`, or standalone `` `Deprecated` `` text to ensure nothing was missed.

- [ ] **Step 8: Commit English README changes**

```bash
git add README.md
git commit -m "docs: replace emoji status markers with Deprecated/Inactive badges in English README"
```

### Task 4: Apply markers to Chinese README

Using the classification results, add markers to `readme-zh-CN.md` and create the Icons section.

**Files:**
- Modify: `readme-zh-CN.md`

- [ ] **Step 1: Add markers to all classified projects (from scratch)**

The Chinese README currently has ZERO status markers — this is adding markers for the first time, not replacing existing ones. For each project entry in the Chinese README:
- If classified as Deprecated: add `` `⛔ Deprecated` `` before the `![github star]` badge (or at end of line if no badge)
- If classified as Inactive: add `` `💤 Inactive` `` before the `![github star]` badge (or at end of line if no badge)
- If Active: no change
- **Important:** Keep marker text in English — do not translate to Chinese

- [ ] **Step 2: Update any redirected/broken URLs identified in Task 2**

- [ ] **Step 3: Create the Icons section**

Insert before the `## 贡献` section:

```markdown
## 图标

⛔ Deprecated — 项目已归档或明确不再维护
💤 Inactive — 至少1年没有更新

```

- [ ] **Step 4: Update the Table of Contents**

Add `  - [图标](#图标)` to the ToC. Insert it between `- [类似 WYSIWYG](#类似-wysiwyg)` and `- [贡献](#贡献)` (around line 18 of `readme-zh-CN.md`).

- [ ] **Step 5: Verify no markers were missed**

Cross-reference the Chinese README entries against the validation results to ensure all Deprecated/Inactive projects received markers.

- [ ] **Step 6: Commit Chinese README changes**

```bash
git add readme-zh-CN.md
git commit -m "docs: add Deprecated/Inactive status markers to Chinese README"
```

### Task 5: Final verification and cleanup

**Files:**
- Read: `README.md`, `readme-zh-CN.md`
- Delete: `docs/superpowers/plans/url-validation-results.md` (optional — keep if useful as reference)

- [ ] **Step 1: Verify English README renders correctly**

Review the full `README.md` to confirm:
- All markers use the correct format (`` `⛔ Deprecated` `` or `` `💤 Inactive` ``)
- No old `:sleeping:` or `:sleepy:` emojis remain
- Scribe has no duplicate markers
- Icons section is updated
- No formatting issues

- [ ] **Step 2: Verify Chinese README renders correctly**

Review the full `readme-zh-CN.md` to confirm:
- All markers are placed correctly (before star badges where present, end of line otherwise)
- Icons section exists with Chinese descriptions
- ToC includes the new section
- No formatting issues

- [ ] **Step 3: Present summary to user**

Show the user a summary of all changes:
- How many projects marked `⛔ Deprecated`
- How many projects marked `💤 Inactive`
- How many projects remain Active (no marker)
- Any URLs that were updated
- Any issues flagged for user decision
