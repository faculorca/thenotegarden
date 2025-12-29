# Cyberpunk Character Extension

Add these properties to your Character Template for cyberpunk stories:

```yaml
augmentations: []
corporate_affiliation:
net_runner_level:
chrome_percentage:
```

## Property Definitions

**`augmentations`** (Optional)
- Cybernetic enhancements the character has
- Format: List of strings
- Example: `["Neural Interface", "Arm Replacement", "Optical Enhancement"]`

**`corporate_affiliation`** (Optional)
- Which mega-corporation they work for or against
- Format: String or wikilink
- Example: `"[[Arasaka Corp]]"` or `"Freelance"`

**`net_runner_level`** (Optional)
- Skill level in hacking/cyberspace
- Format: String or number
- Values: `novice`, `competent`, `expert`, `legendary` OR `1-10`

**`chrome_percentage`** (Optional)
- How much of their body is cybernetic
- Format: Integer (0-100)
- Useful for tracking humanity loss themes

## Additional Sections to Add

Add these sections to the Character Template body:

### Augmentations Detail
List and describe each cybernetic enhancement.

### Corporate History
Past and present relationships with corporations.

### Net Running Skills
Hacking abilities and signature moves.

### Humanity & Chrome
How augmentations affect their identity and relationships.
