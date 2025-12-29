# Historical Fiction Extension

Add these properties to your Character or Story Template for historical fiction:

## For Story Template

```yaml
historical_period:
historical_accuracy_notes:
```

## For Character Template

```yaml
historical_period:
real_person: no
historical_accuracy_notes:
```

## Property Definitions

**`historical_period`** (Required)
- When the story takes place
- Format: String
- Example: `"Victorian England (1837-1901)"`, `"American Civil War (1861-1865)"`

**`real_person`** (For characters only, Required if historical fiction)
- Whether this character is based on a real historical figure
- Format: Boolean
- Values: `yes` or `no`

**`historical_accuracy_notes`** (Optional)
- Track where you deviate from history and why
- Format: Free text
- Use for: Recording research, noting intentional changes, tracking anachronisms

## Additional Sections to Add

### For Story Template:

#### Historical Context
Real events and timeline that frame the story.

#### Historical Liberties
Where you've changed history for narrative purposes.

#### Research Sources
Books, documents, and references consulted.

### For Character Template:

#### Historical Accuracy
If based on a real person, what's accurate vs fictionalized.

#### Period-Appropriate Details
Clothing, speech patterns, social constraints of the era.

#### Historical Context
How historical events affect this character.
