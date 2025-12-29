# Property Migration Plan

## Overview

This document provides step-by-step instructions to migrate your 744-file Obsidian vault from its current chaotic property system to the new optimized schema.

**Total files to process**: 744
**Files with frontmatter**: 672 (90.3%)
**Files without frontmatter**: 72 (9.7%)

---

## Phase 1: Critical Foundation (Do This First)

### Priority 1.1: Create Unified `type` Property

**Problem**: You have THREE overlapping categorization systems:
- `categories` (585 files, 78.6%)
- `type` (336 files, 45.2%)
- `content type` (206 files, 27.7%)

**Solution**: Consolidate into single `type` property.

#### Property Mapping: OLD → NEW

| Old Property | Old Value | New Property | New Value |
|--------------|-----------|--------------|-----------|
| `categories` | `[[People]]` | `type` | `[[Person]]` |
| `categories` | `[[Books]]` or `[[books]]` | `type` | `[[Book]]` |
| `categories` | `[[Newsletter]]` | `type` | `[[Newsletter]]` |
| `categories` | `[[Zettelkasten]]` | `type` | `[[Note]]` |
| `categories` | `[[Ghostwriting Business]]` | `type` | `[[Project]]` |
| `categories` | `[[Posts]]` | `type` | `[[Post]]` |
| `categories` | `[[Prompt]]` | `type` | `[[Prompt]]` |
| `categories` | `[[Companies]]` | `type` | `[[Company]]` |
| `categories` | `[[Swipefile]]` | `type` | `[[Reference]]` |
| `categories` | `[[Podcast episodes]]` | `type` | `[[Podcast]]` |
| `type` (old) | `[[Authors]]` | `type` | `[[Person]]` |
| `type` (old) | `[[Main Note]]` | `type` | `[[Note]]` |
| `type` (old) | `[[Reference Note]]` | `type` | `[[Note]]` |
| `type` (old) | `[[Transcription]]` | `type` | `[[Transcript]]` |
| `type` (old) | `[[Short-Form]]` | `type` | `[[Post]]` |
| `content type` | `[["Tip", "LinkedIn", "Educate Me"]]` | See below | Split into 3 properties |

#### Special Case: `content type` Property (206 files)

The `content type` property is actually well-designed but needs to be split:

```yaml
# OLD
content type:
  - "[[Tip]]"
  - "[[LinkedIn]]"
  - "[[Educate Me]]"

# NEW
type: "[[Post]]"  # or [[Newsletter]]
content_format: tip
content_length: medium
content_purpose: educate
```

**Mapping Table**:

| Old content type[0] | New content_format |
|---------------------|-------------------|
| `[[Tip]]` | `tip` |
| `[[Observations]]` | `observation` |
| `[[Engagement]]` | `question` |
| `[[One-liners]]` | `tip` |
| `[[Twitter Threads]]` | `story` |

| Old content type[1] | New content_length |
|---------------------|-------------------|
| `[[LinkedIn]]` | `medium` |
| `[[LongForm]]` | `long` |
| `[[Tweet]]` | `short` |
| `[[Thread]]` | `medium` |
| `[[CTA]]` | `short` |

| Old content type[2] | New content_purpose |
|---------------------|---------------------|
| `[[Educate Me]]` | `educate` |
| `[[Challenge Me]]` | `challenge` |
| `[[Entertain Me]]` | `entertain` |

#### Implementation Steps

1. **Backup your vault** (critical!)
2. Use the cleanup script queries (see `cleanup-script.md`)
3. Manually review files with multiple categories
4. Delete old `categories` and `type` properties after migration

---

### Priority 1.2: Standardize Date Properties

**Problem**: You have FOUR date properties with inconsistent formats:
- `date` (196 files)
- `created` (277 files, 64.3% empty!)
- `published` (26 files)
- `last` (37 files)

**Date Format Chaos**:
- `2025-11-05` ✅ (correct)
- `2025-09-26, 13:46` ❌ (has time)
- `[datetime.date(2025, 11, 18)]` ❌ (Python object)
- `[[[datetime.date(2025, 11, 24)]]]` ❌ (triple-nested!)

#### Property Mapping: OLD → NEW

| Old Property | Purpose | New Property | New Value |
|--------------|---------|--------------|-----------|
| `created` | When note was created | `date` | `YYYY-MM-DD` |
| `date` | Varies by note type | `date` | `YYYY-MM-DD` |
| `published` | When content published | `date` | `YYYY-MM-DD` |
| `last` | Last modified | DELETE | (Obsidian tracks this) |

#### Decision Rules

**For personal notes** (Slip-Box, Journal, Ideas):
- Use `created` date if it exists and isn't empty
- Format: `YYYY-MM-DD`

**For external content** (Articles, Books, Podcasts):
- Use `published` date if available
- Otherwise use `date` property
- Format: `YYYY-MM-DD`

**For your output** (Newsletters, Posts):
- Use `published` date if available
- Otherwise use `date` property
- Format: `YYYY-MM-DD`

#### Date Format Fixes

**Cleanup patterns** (use regex):

```regex
# Pattern 1: Remove time from date
date: 2025-09-26, 13:46
→ date: 2025-09-26

# Pattern 2: Extract from Python datetime
date: [datetime.date(2025, 11, 18)]
→ date: 2025-11-18

# Pattern 3: Remove nested lists
date: [[[datetime.date(2025, 11, 24)]]]
→ date: 2025-11-24
```

---

### Priority 1.3: Add `status` Property to Active Work

**Current state**: Only 22 files have `status` (59% empty!)

**New requirement**: All active projects need `status`.

#### Apply `status` to:

1. **Newsletter/Post files** (Newsletters folder):
   - `draft` - Not written yet
   - `in-progress` - Currently writing
   - `published` - Live

2. **Fiction files** (when you start fiction):
   - `outline` - Planning stage
   - `draft` - First draft
   - `revised` - Editing
   - `complete` - Finished

3. **Reference files**:
   - `to-read` - Haven't read yet
   - `active` - Currently reading
   - `archived` - Read and processed

4. **Project files**:
   - `active` - Current projects
   - `archived` - Past projects

#### Status Standardization

**OLD status values found**:
- `[[Published]]` → `published`
- `[[Ready]]` → `draft`

**NEW standardized values** (lowercase, no brackets):
- `draft`, `outline`, `in-progress`, `active`, `complete`, `published`, `archived`, `to-read`

---

## Phase 2: Fiction Setup (If Using for Fiction)

### Priority 2.1: Set Up Fiction Folder Structure

Create this structure:

```
Fiction/
├── Characters/
├── Locations/
├── Scenes/
├── Plots/
└── Stories/
```

### Priority 2.2: Create Your First Story

1. Use `Story Template.md` from `obsidian-templates/`
2. Fill in basic information
3. This becomes the parent for all other fiction notes

### Priority 2.3: Link Fiction Notes

**Critical rule**: Every Character, Location, Scene, and Plot note MUST link to a Story.

```yaml
# All fiction notes need this property
story: "[[Your Story Name]]"
```

### Priority 2.4: Apply Genre Extensions

Only if needed:
1. Copy properties from `obsidian-templates/genre-extensions/`
2. Add to relevant Character/Story templates
3. Don't add properties you won't use

---

## Phase 3: Cleanup (After Core Migration)

### Priority 3.1: Remove Empty Properties

**Properties with >60% empty values** (delete from files where empty):

| Property | Empty Rate | Action |
|----------|-----------|--------|
| `via` | 100% | DELETE entirely |
| `end` | 100% | DELETE entirely |
| `people` | 83.3% | Delete where empty |
| `start` | 75.0% | Delete where empty |
| `org` | 72.7% | Delete where empty |
| `source` | 65.2% | Delete where empty |
| `created` | 64.3% | Migrate to `date`, then delete |
| `rating` | 62.5% | Delete where empty |
| `topics` | 61.4% | Delete where empty |

### Priority 3.2: Delete Single-Use Properties

**These properties appear in ≤4 files** (template artifacts):

```yaml
# Delete these entirely:
servings, icon, color, director, cast, imdbId, system,
userID, profile, longform, clipped, channel, duration,
reviewed, role (old), season, phone, producer, country,
variety, process, attribution, series, cuisine, ingredients,
coordinates, artist, twitter, maker, episode, birthday,
cover, isbn, isbn13, pages
```

**Exception**: Keep if you plan to use them. But most are from templates you're not using (recipe, movie, video game).

### Priority 3.3: Standardize Property Value Casing

**Problem**: Same value with different casing breaks queries.

**Found issues**:
- `[[Books]]` vs `[[books]]`
- `[[Prompt]]` vs `Prompt` (missing brackets)
- `[[Authors]]` vs `[[People]]` (inconsistent terminology)

**Fix**:
1. Choose ONE canonical form (recommend: `[[Title Case]]`)
2. Find and replace all variations
3. Update all backlinks

### Priority 3.4: Standardize Property Formats

**Problem**: Same property using both string and list format.

```yaml
# WRONG - Inconsistent formats
categories: [[People]]  # String
categories: ['[[People]]']  # List

# RIGHT - Choose one
type: "[[Person]]"  # Use string for single values
topics: ["[[Writing]]", "[[Fiction]]"]  # Use list for multiple
```

**Fix patterns**:

```yaml
# Single value properties (use string)
type: "[[Type]]"
pov_character: "[[Character]]"
location: "[[Location]]"
story: "[[Story]]"

# Multiple value properties (use list)
relationships: ["[[Char1]]", "[[Char2]]"]
characters: []
locations: []
topics: []
genres: []
```

### Priority 3.5: Add Frontmatter to Missing Files

**72 files have NO frontmatter** (9.7% of vault).

Key files missing frontmatter:
- `Readme.md` (intentional?)
- `Digital Minimalism AI Research.md`
- `Prompt.md`
- `Fiction.md`
- `Newsletter.md`
- Multiple template files
- Multiple files in `Categories/` folder

**Action**:
1. Review the list in `vault-analysis-raw.md` (Section 1.3)
2. Add minimum frontmatter:
   ```yaml
   ---
   type: [[Type]]
   date: YYYY-MM-DD
   ---
   ```

### Priority 3.6: Consolidate URL Properties

**Current**: Both `url` and `link` exist
**New**: Use only `url`

```yaml
# OLD
link: https://example.com

# NEW
url: https://example.com
```

---

## Migration Execution Order

### Week 1: Foundation
- [ ] Backup entire vault
- [ ] Run Phase 1.1: Create unified `type` property (585 files)
- [ ] Run Phase 1.2: Standardize dates (277 files)
- [ ] Run Phase 1.3: Add `status` to active work

### Week 2: Cleanup
- [ ] Phase 3.1: Remove empty properties
- [ ] Phase 3.2: Delete single-use properties
- [ ] Phase 3.3: Fix casing issues
- [ ] Phase 3.4: Standardize formats
- [ ] Phase 3.5: Add frontmatter to 72 files

### Week 3+: Fiction (If Applicable)
- [ ] Phase 2.1: Create Fiction folder structure
- [ ] Phase 2.2: Create first story
- [ ] Phase 2.3: Create characters, locations, etc.
- [ ] Phase 2.4: Add genre extensions if needed

---

## Verification Checklist

After migration, verify:

- [ ] Every note has a `type` property
- [ ] Every note has a `date` in `YYYY-MM-DD` format
- [ ] No notes have old `categories` property
- [ ] No notes have old `type` property (replaced with new one)
- [ ] No notes have `content type` property (split into 3)
- [ ] Active projects have `status` property
- [ ] No empty properties remain (except intentionally)
- [ ] All wikilinks use consistent casing
- [ ] Fiction notes link to parent `story` (if applicable)

---

## Risk Areas & Manual Review Needed

### High Risk: Categories with Multiple Values

Some files have multiple categories that need manual decisions:

```yaml
# Example: What should this become?
categories:
  - "[[Zettelkasten]]"
  - "[[Podcast episodes]]"

# Options:
type: "[[Podcast]]"  # Primary type
topics: ["[[Zettelkasten]]"]  # Secondary classification
```

**Action**: Review the 50+ files with multiple categories (see cleanup script).

### Medium Risk: Content Type Split

206 files with `content type` need to be split into 3 properties. The cleanup script handles common patterns, but review edge cases:

```yaml
# Edge case examples from your vault:
content type:
  - "[[One-liners]]"
  - "[[EducateMe]]"  # Note: typo in original
  - "[[Educate Me]]"

content type:
  - "[[Tip]]"
  - "[[prompt]]"  # Note: lowercase
  - "[[Educate Me]]"
```

### Low Risk: Template Files

Your `Templates/` folder (63 files) may have old property schemas. Don't migrate these—either delete them or leave them as-is since they're templates.

---

## Post-Migration Benefits

After completing this migration, you'll be able to:

1. **Find things instantly**:
   ```dataview
   LIST FROM "path" WHERE type = [[Character]] AND status = "active"
   ```

2. **Build knowledge graphs**:
   - Characters → Locations they visit
   - Scenes → Characters present
   - Plots → Related characters

3. **Track progress**:
   ```dataview
   TABLE status, word_count
   FROM "Fiction"
   WHERE type = [[Scene]] AND status != "complete"
   ```

4. **Reduce cognitive load**:
   - One property for categorization (`type`)
   - One property for dates (`date`)
   - Clear, consistent values

5. **Scale confidently**:
   - Add new fiction notes without chaos
   - Templates work across all content
   - No more wondering "which property do I use?"

---

## Emergency Rollback

If migration goes wrong:

1. **You did backup, right?** (See Week 1, Step 1)
2. Restore from backup
3. Try cleanup script on a test folder first
4. Migrate in batches (folder by folder)

---

## Getting Help

**Common issues**:

1. **"I broke all my queries!"**
   - Check property names (old vs new)
   - Check value format (string vs list)

2. **"I have files with no `type`!"**
   - See Section 3.5: Add Frontmatter

3. **"My dates are still broken!"**
   - Use cleanup script regex patterns
   - See Section 1.2: Date Format Fixes

4. **"I want to keep both `categories` and `type`!"**
   - Bad idea. You'll regret it.
   - But if you insist, at least make them consistent.

---

*This migration will take time but the result is a vault that scales, queries efficiently, and doesn't make you question your life choices every time you create a note.*
