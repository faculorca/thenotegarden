# Science Fiction Extension

Add these properties to your templates for science fiction stories:

## For Character Template

```yaml
homeworld:
tech_specialization:
species:
```

## For Location Template

```yaml
planet:
star_system:
tech_level:
```

## For Story Template

```yaml
tech_level:
faster_than_light: yes
ai_presence: limited
```

## Property Definitions

### Character Properties

**`homeworld`** (Optional)
- Planet or station where character is from
- Format: String or wikilink
- Example: `"[[Mars Colony]]"`, `"Earth"`

**`tech_specialization`** (Optional)
- Character's technical expertise
- Format: String
- Example: `"Quantum Mechanics"`, `"Starship Engineering"`

**`species`** (Optional, if multi-species setting)
- Alien race or human variant
- Format: String or wikilink
- Example: `"Human"`, `"[[Zynthari]]"`

### Location Properties

**`planet`** (Optional)
- Parent planet for locations
- Format: String or wikilink

**`star_system`** (Optional)
- Which star system
- Format: String or wikilink

**`tech_level`** (Optional)
- Technological advancement of location
- Format: String
- Values: `pre-industrial`, `industrial`, `atomic`, `information`, `space-age`, `post-singularity`

### Story Properties

**`tech_level`** (Required for sci-fi)
- Overall technological state of the setting
- Format: String
- Example: `"Near-future (2080s)"`, `"Hard sci-fi, realistic physics"`

**`faster_than_light`** (Optional)
- Whether FTL travel exists
- Format: Boolean
- Values: `yes`, `no`, `limited`

**`ai_presence`** (Optional)
- Level of AI in the setting
- Format: String
- Values: `none`, `limited`, `common`, `sentient`, `dominant`

## Additional Sections to Add

### For Story Template:

#### Technology Rules
What's possible and what isn't in this universe.

#### Scientific Basis
Real science you're building from.

### For Location Template:

#### Technological Infrastructure
What tech is available here.

#### Planetary Characteristics
Gravity, atmosphere, climate (if applicable).
