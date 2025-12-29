# Fantasy Character Extension

Add these properties to your Character Template for fantasy stories:

```yaml
magic_type: []
species:
faction_loyalty:
magic_level:
```

## Property Definitions

**`magic_type`** (Optional)
- What kind(s) of magic they can use
- Format: List of strings or wikilinks
- Example: `["Fire Magic", "Divination"]` or `["[[Arcane]]", "[[Divine]]"]`

**`species`** (Required for fantasy)
- Character's race or species
- Format: String or wikilink
- Example: `"Human"`, `"[[Elf]]"`, `"Half-Orc"`

**`faction_loyalty`** (Optional)
- Which group, guild, or kingdom they serve
- Format: String or wikilink
- Example: `"[[Mage Guild]]"`, `"[[Kingdom of Eldoria]]"`

**`magic_level`** (Optional)
- Magical ability or power tier
- Format: String or number
- Values: `none`, `apprentice`, `adept`, `master`, `archmage` OR `1-10`

## Additional Sections to Add

Add these sections to the Character Template body:

### Magical Abilities
Detail their spells, powers, and limitations.

### Species Traits
Racial abilities and cultural background.

### Faction Relationships
Connections to guilds, kingdoms, or organizations.

### Magical Items
Artifacts, weapons, or tools they possess.
