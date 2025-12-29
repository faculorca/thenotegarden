# Optimized Obsidian Property Schema

## Philosophy

This schema follows these principles:
1. **One property, one purpose** - No redundant properties
2. **Queryable by default** - Every property enables filtering or linking
3. **Standardized values** - Enum-like properties use consistent values
4. **Graph-first** - Properties create network relationships
5. **Minimal but complete** - Only properties that add real value

---

## Core Property System

### Universal Properties (ALL content types)

These properties appear in every note:

```yaml
type: [note-type]
date: YYYY-MM-DD
status: [standardized-value]
```

#### `type` (Required)
- **Purpose**: Single source of truth for note categorization
- **Format**: Single wikilink `[[Type]]`
- **Replaces**: Old `categories`, `type`, and `content type` properties
- **Standard values**:
  - Content: `[[Character]]`, `[[Location]]`, `[[Scene]]`, `[[Plot]]`, `[[Story]]`
  - Reference: `[[Article]]`, `[[Book]]`, `[[Podcast]]`, `[[Video]]`
  - Meta: `[[Person]]`, `[[Company]]`, `[[Product]]`
  - Personal: `[[Note]]`, `[[Journal]]`, `[[Idea]]`
  - Output: `[[Newsletter]]`, `[[Post]]`, `[[Thread]]`

#### `date` (Required)
- **Purpose**: Creation or publication date
- **Format**: `YYYY-MM-DD` (ISO 8601)
- **Replaces**: Old `date`, `created`, `published`, `last`
- **Rule**: Use creation date for personal notes, publication date for external content

#### `status` (Optional but recommended)
- **Purpose**: Track progress and filter active work
- **Standard values**:
  - Planning: `draft`, `outline`, `research`
  - Active: `in-progress`, `active`
  - Done: `complete`, `published`, `archived`

---

## Fiction Writing Properties

### Character Notes (`type: [[Character]]`)

```yaml
---
type: [[Character]]
date: 2025-12-29
status: active
role: protagonist
first_appearance: "[[Chapter 1]]"
last_appearance:
relationships: []
locations: []
story: "[[Story Name]]"
aliases: []
---
```

**Property Definitions:**

- **`role`** (Required): Character's narrative function
  - Values: `protagonist`, `antagonist`, `supporting`, `minor`

- **`first_appearance`** (Optional): First scene/chapter where character appears
  - Format: Wikilink to scene/chapter note

- **`last_appearance`** (Optional): Last scene/chapter (useful for tracking character arcs)
  - Format: Wikilink to scene/chapter note

- **`relationships`** (Optional): Array of character relationships
  - Format: List of wikilinks to other character notes
  - Example: `["[[Jane Doe]]", "[[Bob Smith]]"]`

- **`locations`** (Optional): Key locations associated with character
  - Format: List of wikilinks to location notes

- **`story`** (Required): Parent story/project
  - Format: Wikilink to story note

- **`aliases`** (Optional): Alternative names for the character
  - Format: List of strings

---

### Location Notes (`type: [[Location]]`)

```yaml
---
type: [[Location]]
date: 2025-12-29
status: active
parent_location: "[[Larger Location]]"
location_type: city
first_appearance: "[[Chapter 1]]"
characters: []
significance: major
story: "[[Story Name]]"
---
```

**Property Definitions:**

- **`parent_location`** (Optional): Larger location containing this one
  - Format: Wikilink to parent location note
  - Example: A building would link to a city

- **`location_type`** (Required): What kind of location
  - Values: `planet`, `region`, `city`, `building`, `room`, `vehicle`, `virtual`

- **`first_appearance`** (Optional): First mention in narrative
  - Format: Wikilink to scene/chapter

- **`characters`** (Optional): Characters present or associated
  - Format: List of wikilinks to character notes

- **`significance`** (Optional): Narrative importance
  - Values: `major`, `minor`, `background`

- **`story`** (Required): Parent story/project
  - Format: Wikilink to story note

---

### Scene Notes (`type: [[Scene]]`)

```yaml
---
type: [[Scene]]
date: 2025-12-29
status: draft
chapter: "[[Chapter 3]]"
timeline_position:
pov_character: "[[Character Name]]"
location: "[[Location Name]]"
characters: []
plot_threads: []
word_count:
story: "[[Story Name]]"
---
```

**Property Definitions:**

- **`chapter`** (Optional): Parent chapter or section
  - Format: Wikilink to chapter note

- **`timeline_position`** (Optional): When scene occurs
  - Format: Flexible (Day 3, 2045-03-15, Act 2, etc.)

- **`pov_character`** (Required): Whose perspective
  - Format: Wikilink to character note

- **`location`** (Required): Where scene takes place
  - Format: Wikilink to location note

- **`characters`** (Required): Who appears in scene
  - Format: List of wikilinks to character notes

- **`plot_threads`** (Optional): Which plot lines advance
  - Format: List of wikilinks to plot notes

- **`word_count`** (Optional): Scene length
  - Format: Integer

- **`story`** (Required): Parent story/project
  - Format: Wikilink to story note

---

### Plot Notes (`type: [[Plot]]`)

```yaml
---
type: [[Plot]]
date: 2025-12-29
status: developing
plot_type: main
related_characters: []
scenes: []
story: "[[Story Name]]"
---
```

**Property Definitions:**

- **`plot_type`** (Required): Plot classification
  - Values: `main`, `subplot`, `character-arc`, `theme`

- **`related_characters`** (Required): Characters involved
  - Format: List of wikilinks to character notes

- **`scenes`** (Optional): Scenes where plot advances
  - Format: List of wikilinks to scene notes

- **`story`** (Required): Parent story/project
  - Format: Wikilink to story note

---

### Story Notes (`type: [[Story]]`)

```yaml
---
type: [[Story]]
date: 2025-12-29
status: drafting
genre: []
setting:
pov_type: third-limited
tense: past
target_length:
current_word_count:
main_characters: []
main_locations: []
themes: []
---
```

**Property Definitions:**

- **`genre`** (Required): Story genres
  - Format: List of strings or wikilinks
  - Examples: `["Cyberpunk", "Thriller"]`

- **`setting`** (Optional): High-level setting description
  - Format: Free text
  - Example: "Neo-Tokyo 2087"

- **`pov_type`** (Required): Narrative perspective
  - Values: `first`, `second`, `third-limited`, `third-omniscient`, `multiple`

- **`tense`** (Required): Narrative tense
  - Values: `past`, `present`, `future`

- **`target_length`** (Optional): Goal word count
  - Format: Integer or string ("novella", "novel", etc.)

- **`current_word_count`** (Optional): Running total
  - Format: Integer

- **`main_characters`** (Required): Protagonist(s)
  - Format: List of wikilinks to character notes

- **`main_locations`** (Required): Key settings
  - Format: List of wikilinks to location notes

- **`themes`** (Optional): Central themes
  - Format: List of strings

---

## Genre-Specific Extensions

### Principle: Modular Addition

Base templates contain only universal properties. Genre-specific properties are added **only when needed** by copying from extension templates.

### Cyberpunk Extension

Add these properties to Character notes in cyberpunk stories:

```yaml
augmentations: []
corporate_affiliation:
net_runner_level:
chrome_percentage:
```

### Fantasy Extension

Add these properties to Character notes in fantasy stories:

```yaml
magic_type: []
species:
faction_loyalty:
magic_level:
```

### Historical Fiction Extension

Add these properties to Character or Story notes:

```yaml
historical_period:
real_person: no
historical_accuracy_notes:
```

---

## Reference Content Properties

### Article/Book/Podcast Notes

```yaml
---
type: [[Article]]  # or [[Book]], [[Podcast]], etc.
date: 2025-12-29
author: ["[[Author Name]]"]
url:
topics: []
---
```

**Key Properties:**

- **`author`**: List of wikilinks to person notes
- **`url`**: Direct link to content
- **`topics`**: List of wikilinks to topic notes (replaces old `topics` property which was 61% empty)

---

## Personal Content Properties

### Newsletter/Post Notes

```yaml
---
type: [[Newsletter]]  # or [[Post]], [[Thread]]
date: 2025-12-29
status: published
topics: []
content_format: tip
content_length: long
content_purpose: educate
---
```

**Property Definitions:**

- **`content_format`** (Optional): What kind of content
  - Values: `tip`, `observation`, `story`, `analysis`, `question`

- **`content_length`** (Optional): Size category
  - Values: `short` (tweet), `medium` (LinkedIn), `long` (article)

- **`content_purpose`** (Optional): Reader goal
  - Values: `educate`, `entertain`, `challenge`, `inspire`

**Note**: These three properties replace the old `content type` property which was actually well-designed but overlapped with `type`.

---

## Properties to REMOVE

These properties from your current vault should be deleted:

### Completely Remove (100% empty or single use)
- `via` (100% empty)
- `end` (100% empty)
- `role` (100% empty - will be recreated in fiction templates)
- `season`, `phone`, `producer`, `country`, `variety`, `process`, `attribution`, `series` (all 100% empty)
- Template artifacts: `servings`, `icon`, `color`, `director`, `cast`, `imdbId`, `system`, `ingredients`, `cuisine`, `coordinates`, `artist`, `twitter`, `maker`, `episode`, `birthday`, `cover`, `isbn`, `isbn13`, `pages`, `rating`, `year`, `genre` (old version)

### Consolidate Into New Schema
- `categories` → becomes `type`
- `type` (old) → becomes `type` (new)
- `content type` → becomes `content_format`, `content_length`, `content_purpose`
- `created`, `published`, `last` → becomes `date`
- `topics` (61% empty) → keep but use more consistently

### Keep But Standardize
- `author` → standardize as list of wikilinks
- `url` / `link` → consolidate to just `url`
- `source` → keep for attribution
- `image` → keep for visual references
- `tags` → keep for non-hierarchical tagging

---

## Migration Priority

### Phase 1: Critical (Do First)
1. Create new `type` property from old `categories`/`type`/`content type`
2. Standardize `date` property format (YYYY-MM-DD)
3. Add `status` to active projects

### Phase 2: Fiction Setup (If using for fiction)
4. Create fiction templates
5. Set up genre extensions
6. Link fiction notes together

### Phase 3: Cleanup (After migration)
7. Remove empty properties
8. Delete single-use properties
9. Standardize property value casing
10. Add frontmatter to 72 files missing it

---

## Dataview Query Examples

### Find all active characters
```dataview
TABLE role, first_appearance, story
FROM "path/to/fiction"
WHERE type = [[Character]] AND status = "active"
SORT role
```

### Find scenes by location
```dataview
TABLE pov_character, characters, word_count
FROM "path/to/fiction"
WHERE type = [[Scene]] AND location = [[Downtown Tokyo]]
SORT timeline_position
```

### Track plot thread progress
```dataview
TABLE plot_type, related_characters, status
FROM "path/to/fiction"
WHERE type = [[Plot]]
SORT status, plot_type
```

### Content pipeline
```dataview
TABLE content_format, content_purpose, status
FROM "Newsletters"
WHERE type = [[Newsletter]] AND status != "published"
SORT date DESC
```

---

## Schema Validation Checklist

Use this to ensure note quality:

- [ ] Every note has a `type` property
- [ ] Every note has a `date` in YYYY-MM-DD format
- [ ] Active projects have a `status` property
- [ ] Fiction notes link to parent `story`
- [ ] Character notes specify `role`
- [ ] Scene notes link to `location` and `pov_character`
- [ ] No empty properties (delete if unused)
- [ ] All wikilinks use consistent casing
- [ ] List properties use array format `[]`

---

*This schema is designed to be minimal, queryable, and graph-friendly. Every property serves a purpose. If a property doesn't enable filtering, linking, or graph relationships, it shouldn't exist.*
