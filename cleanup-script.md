# Vault Cleanup Scripts & Queries

This document contains practical Dataview queries and regex patterns to help you migrate your vault to the new property schema.

---

## Part 1: Discovery Queries

Run these queries to understand what you're working with before making changes.

### Query 1.1: Find All Files with Old `categories` Property

```dataview
TABLE categories, type as "old_type", file.ctime as created
WHERE categories
SORT file.folder, file.name
```

**Purpose**: See all 585 files that need `categories` → `type` migration.

### Query 1.2: Find Files with Multiple Categorization Properties

```dataview
TABLE categories, type, content-type
WHERE categories AND type
SORT file.folder
```

**Purpose**: Identify the 336 files with redundant categorization (have both `categories` AND `type`).

### Query 1.3: Find All Date Properties (The Chaos)

```dataview
TABLE date, created, published, last
WHERE date OR created OR published OR last
SORT file.name
```

**Purpose**: See all files with date properties to understand the scope of date standardization.

### Query 1.4: Find Empty Properties

```dataview
TABLE topics, created, rating, status, source
WHERE (topics = "" OR topics = null OR topics = []) OR
      (created = "" OR created = null) OR
      (rating = "" OR rating = null) OR
      (status = "" OR status = null) OR
      (source = "" OR source = null)
```

**Purpose**: Find files with empty properties that should be deleted.

### Query 1.5: Find Files Missing Frontmatter

```dataview
TABLE file.ctime
WHERE !categories AND !type AND !date AND !created
SORT file.folder, file.name
```

**Purpose**: Identify the 72 files that need frontmatter added.

### Query 1.6: Find Content Type Files

```dataview
TABLE content-type as "content_type", categories
WHERE content-type
SORT file.folder
```

**Purpose**: See the 206 files where `content type` needs to be split into 3 properties.

---

## Part 2: Validation Queries (After Migration)

Run these AFTER migration to verify everything worked.

### Query 2.1: Verify All Files Have New `type`

```dataview
TABLE type, date, status
WHERE type
SORT type, file.name
```

**Expected**: Should return all 744 files (after adding frontmatter to the 72 missing).

### Query 2.2: Find Files Still Using Old Properties

```dataview
TABLE categories as "OLD_categories", file.path
WHERE categories
```

**Expected**: Should return ZERO files after migration.

```dataview
TABLE content-type as "OLD_content_type", file.path
WHERE content-type
```

**Expected**: Should return ZERO files after migration.

### Query 2.3: Verify Date Format Consistency

```dataview
TABLE date, file.path
WHERE date AND !regexmatch("^\d{4}-\d{2}-\d{2}$", string(date))
SORT file.name
```

**Expected**: Should return ZERO files (all dates should be YYYY-MM-DD).

### Query 2.4: Find Files with Empty Properties (Should Be None)

```dataview
TABLE file.path
WHERE topics = "" OR topics = null OR
      status = "" OR status = null OR
      source = "" OR source = null
```

**Expected**: Should return ZERO files after cleanup.

---

## Part 3: Migration Helper Queries

These queries help you actually DO the migration work.

### Query 3.1: Group Files by Category for Migration

```dataview
TABLE rows.file.link as "Files", length(rows) as "Count"
WHERE categories
GROUP BY categories
SORT length(rows) DESC
```

**Purpose**: Understand which category values appear most (helps with bulk migration).

**Output example**:
- `[[People]]`: 32 files
- `[[Ghostwriting Business]]`: 5 files
- `[[Newsletter]]`: 4 files

### Query 3.2: Find Newsletter Files for Content Type Split

```dataview
TABLE content-type, categories, type
FROM "Newsletters"
WHERE content-type
SORT file.name
```

**Purpose**: The 206 files in Newsletters/Templates need special handling for content type split.

### Query 3.3: Find Fiction Files (Terminal Space Project)

```dataview
TABLE type, campaign, genre
FROM "Longform Writing Projects"
SORT file.folder, file.name
```

**Purpose**: Your existing Terminal Space project needs fiction template migration.

### Query 3.4: Find People Notes Across Folders

```dataview
TABLE categories, type, file.folder
WHERE categories = "[[People]]" OR type = "[[Authors]]" OR file.folder = "Categories/People"
SORT file.folder, file.name
```

**Purpose**: Find all person notes (in multiple places) to standardize as `type: [[Person]]`.

---

## Part 4: Bulk Find & Replace Patterns

Use Obsidian's built-in search and replace or a tool like VSCode for these.

### Pattern 4.1: Migrate `categories: [[People]]` to `type: [[Person]]`

**Find (Regex)**:
```regex
^categories:\s*\[\[People\]\]
```

**Replace**:
```
type: "[[Person]]"
```

**Test on**: Categories/People folder (35 files)

### Pattern 4.2: Migrate `categories: [[Newsletter]]` to `type: [[Newsletter]]`

**Find (Regex)**:
```regex
^categories:\s*\[?'?\[\[Newsletter\]\]'?\]?
```

**Replace**:
```
type: "[[Newsletter]]"
```

**Note**: Handles both `[[Newsletter]]` and `['[[Newsletter]]']` formats.

### Pattern 4.3: Fix Date Format - Remove Time

**Find (Regex)**:
```regex
^(date|created|published):\s*(\d{4}-\d{2}-\d{2}),\s*\d{2}:\d{2}
```

**Replace**:
```
$1: $2
```

**Example**:
- Before: `date: 2025-09-26, 13:46`
- After: `date: 2025-09-26`

### Pattern 4.4: Fix Date Format - Extract from datetime.date()

**Find (Regex)**:
```regex
^(date|created|published):\s*\[*datetime\.date\((\d{4}),\s*(\d{1,2}),\s*(\d{1,2})\)\]*
```

**Replace**:
```
$1: $2-$3-$4
```

**Example**:
- Before: `date: [datetime.date(2025, 11, 18)]`
- After: `date: 2025-11-18`

**Note**: You may need to pad single-digit months/days manually (11 → 11, but 6 → 06).

### Pattern 4.5: Standardize Status Values

**Find**: `status: \[\[Published\]\]`
**Replace**: `status: published`

**Find**: `status: \[\[Ready\]\]`
**Replace**: `status: draft`

### Pattern 4.6: Remove Old `created` Property (After Migrating to `date`)

**Find (Regex)**:
```regex
^created:.*\n
```

**Replace**: (leave empty)

**WARNING**: Only do this AFTER you've copied `created` values to `date`.

### Pattern 4.7: Consolidate `link` to `url`

**Find**: `^link:`
**Replace**: `url:`

---

## Part 5: Dataview Queries for Content Type Migration

The `content type` property (206 files) needs special handling.

### Query 5.1: Map Content Type Values

```dataview
TABLE content-type[0] as "Format", content-type[1] as "Length", content-type[2] as "Purpose"
WHERE content-type
SORT content-type[0], content-type[1]
```

**Purpose**: See what values exist so you can build correct mapping.

### Query 5.2: Find Content Type Patterns

```dataview
TABLE content-type, rows.file.link
WHERE content-type
GROUP BY content-type
SORT length(rows) DESC
```

**Output**: Shows most common patterns (e.g., "Tip + LinkedIn + Educate Me" appears 13 times).

### Manual Migration Template for Content Type

For each file with `content type`, replace:

```yaml
# OLD
content type:
  - "[[Tip]]"
  - "[[LinkedIn]]"
  - "[[Educate Me]]"

# NEW
type: "[[Newsletter]]"
content_format: tip
content_length: medium
content_purpose: educate
```

**Mapping cheat sheet**:

| content_type[0] | → content_format |
|-----------------|------------------|
| Tip | tip |
| Observations | observation |
| Engagement | question |
| One-liners | tip |

| content_type[1] | → content_length |
|-----------------|------------------|
| LinkedIn | medium |
| LongForm | long |
| Tweet | short |
| Thread | medium |

| content_type[2] | → content_purpose |
|-----------------|------------------|
| Educate Me | educate |
| Challenge Me | challenge |
| Entertain Me | entertain |

---

## Part 6: Property Deletion Scripts

### Query 6.1: Find Files with Single-Use Properties to Delete

```dataview
TABLE servings, icon, color, director, cast, imdbId, system
WHERE servings OR icon OR color OR director OR cast OR imdbId OR system
```

**Action**: Delete these properties from the files found (template artifacts).

### Query 6.2: Find Files with 100% Empty Properties

```dataview
TABLE via, end, role as "old_role"
WHERE via != null OR end != null
```

**Expected**: Should find 11 files with `via`, 4 with `end`. Delete these properties.

### Pattern 6.1: Delete Empty `topics` Property

**Find (Regex)**:
```regex
^topics:\s*(\[\]|""|null|\n)$
```

**Replace**: (leave empty)

### Pattern 6.2: Delete Empty `created` Property

**Find (Regex)**:
```regex
^created:\s*(\[\]|""|null)?\n
```

**Replace**: (leave empty)

---

## Part 7: Obsidian Plugins to Help

These Obsidian plugins make migration easier:

### 7.1 MetaEdit
- GUI for editing frontmatter
- Bulk property renaming
- Install: Community plugins → search "MetaEdit"

### 7.2 Dataview
- All queries in this document require Dataview
- Already popular, likely installed
- Install: Community plugins → search "Dataview"

### 7.3 Find and Replace
- Core plugin (already enabled)
- Use for regex patterns above
- Settings → Core plugins → Find and replace

### 7.4 Text Expand (Optional)
- Auto-expand templates
- Useful for applying new schema to old notes
- Install: Community plugins → search "Text Expand"

---

## Part 8: Migration Workflow (Step-by-Step)

### Recommended Process:

1. **Backup vault** (copy entire folder)

2. **Test on one folder**:
   - Pick a small folder (e.g., `References/Books` - 11 files)
   - Apply migration patterns
   - Verify with validation queries
   - If it works, proceed to next folder

3. **Migrate by content type**:
   - **Phase A**: People notes (Categories/People + root People files)
   - **Phase B**: Reference notes (References folder)
   - **Phase C**: Newsletters (Newsletters folder)
   - **Phase D**: Slip-Box notes (Slip-Box folder)
   - **Phase E**: Templates (decide: migrate or delete?)
   - **Phase F**: Everything else

4. **Run validation queries** after each phase

5. **Delete old properties** only after confirming new ones work

6. **Add missing frontmatter** to 72 files (see Part 9)

---

## Part 9: Add Frontmatter to Missing Files

### Query 9.1: List Files Without Any Properties

```dataview
LIST
WHERE !categories AND !type AND !date AND !created AND !tags
SORT file.folder, file.name
```

**Found**: 72 files

**Action**: Add minimum frontmatter:

```yaml
---
type: "[[Type]]"
date: {{file.ctime}}
---
```

**Files to review** (from your vault):
- `Digital Minimalism AI Research.md` → type: `[[Note]]`
- `Prompt.md` → type: `[[Prompt]]`
- `Fiction.md` → type: `[[Note]]` or category page
- `Newsletter.md` → type: `[[Note]]` or category page
- Template files → decide: keep as-is or delete

---

## Part 10: Verification Checklist

After running all migrations, verify:

### ✅ Core Properties
```dataview
TABLE type, date, status
WHERE type
LIMIT 10
```
- [ ] All files have `type`
- [ ] All files have `date` in YYYY-MM-DD format
- [ ] Active files have `status`

### ✅ No Old Properties
```dataview
LIST
WHERE categories OR content-type
```
- [ ] Returns ZERO results

### ✅ No Empty Properties
```dataview
LIST
WHERE topics = "" OR created = "" OR rating = ""
```
- [ ] Returns ZERO results (or files you intentionally left)

### ✅ Consistent Formatting
```dataview
TABLE type, author, topics
WHERE type
LIMIT 20
```
- [ ] Wikilinks use consistent casing (`[[Title]]` not `[[title]]`)
- [ ] Lists use array format: `[]`
- [ ] Single values don't use arrays

### ✅ Fiction Notes Linked (If Applicable)
```dataview
TABLE story, type
WHERE type = [[Character]] OR type = [[Location]] OR type = [[Scene]] OR type = [[Plot]]
```
- [ ] All fiction notes link to a `story`

---

## Part 11: Emergency Rollback

If something breaks:

1. **You backed up the vault, right?** (See Part 8, Step 1)

2. **Restore from backup**:
   ```bash
   # Delete current vault
   rm -rf /path/to/vault

   # Restore backup
   cp -r /path/to/backup /path/to/vault
   ```

3. **Try again** on a test folder first

---

## Part 12: Common Issues & Solutions

### Issue: "Dataview query returns nothing!"

**Solution**: Check syntax, property names match new schema.

### Issue: "My dates are still broken after regex!"

**Solution**: Some dates need manual fixing (especially datetime.date format with single-digit months).

### Issue: "I accidentally deleted all my frontmatter!"

**Solution**: Restore from backup. Use Obsidian's local backup feature (Settings → Files & Links → Deleted files).

### Issue: "Content type split is too tedious!"

**Solution**: Use MetaEdit plugin for bulk editing, or write a Python script to automate (example below).

---

## Part 13: Advanced - Python Migration Script

If you're comfortable with Python, here's a script to automate content type migration:

```python
import os
import re
import yaml
from pathlib import Path

# Configuration
VAULT_PATH = "/path/to/your/vault"
BACKUP = True

# Mapping dictionaries
FORMAT_MAP = {
    "[[Tip]]": "tip",
    "[[Observations]]": "observation",
    "[[Engagement]]": "question",
    "[[One-liners]]": "tip",
}

LENGTH_MAP = {
    "[[LinkedIn]]": "medium",
    "[[LongForm]]": "long",
    "[[Tweet]]": "short",
    "[[Thread]]": "medium",
}

PURPOSE_MAP = {
    "[[Educate Me]]": "educate",
    "[[Challenge Me]]": "challenge",
    "[[Entertain Me]]": "entertain",
}

def migrate_file(file_path):
    """Migrate a single file's content type property."""
    with open(file_path, 'r', encoding='utf-8') as f:
        content = f.read()

    # Extract frontmatter
    match = re.match(r'^---\n(.*?)\n---\n(.*)', content, re.DOTALL)
    if not match:
        return False

    fm_text, body = match.groups()
    frontmatter = yaml.safe_load(fm_text)

    # Check if has content type
    if 'content type' not in frontmatter:
        return False

    ct = frontmatter['content type']
    if not isinstance(ct, list) or len(ct) < 3:
        return False

    # Map to new properties
    frontmatter['type'] = "[[Newsletter]]"  # or determine dynamically
    frontmatter['content_format'] = FORMAT_MAP.get(ct[0], "unknown")
    frontmatter['content_length'] = LENGTH_MAP.get(ct[1], "unknown")
    frontmatter['content_purpose'] = PURPOSE_MAP.get(ct[2], "unknown")

    # Remove old property
    del frontmatter['content type']

    # Write back
    new_fm = yaml.dump(frontmatter, default_flow_style=False, allow_unicode=True)
    new_content = f"---\n{new_fm}---\n{body}"

    if BACKUP:
        backup_path = file_path + ".backup"
        with open(backup_path, 'w', encoding='utf-8') as f:
            f.write(content)

    with open(file_path, 'w', encoding='utf-8') as f:
        f.write(new_content)

    return True

# Run migration
def main():
    vault = Path(VAULT_PATH)
    migrated = 0

    for md_file in vault.rglob("*.md"):
        if migrate_file(md_file):
            print(f"Migrated: {md_file}")
            migrated += 1

    print(f"\nTotal migrated: {migrated} files")

if __name__ == "__main__":
    main()
```

**Usage**:
1. Update `VAULT_PATH`
2. Test on a backup first!
3. Run: `python migrate.py`

---

## Summary

This cleanup script provides:

✅ **Discovery queries** to understand current state
✅ **Validation queries** to verify migration success
✅ **Find & replace patterns** for bulk changes
✅ **Content type migration** helpers
✅ **Property deletion** queries
✅ **Step-by-step workflow** for safe migration
✅ **Emergency rollback** procedures
✅ **Python automation** for advanced users

**Remember**:
- Backup before starting
- Migrate one folder at a time
- Validate after each phase
- Don't delete old properties until new ones work

---

*This is tedious work, but the result is a vault that doesn't fight you every time you want to find something.*
