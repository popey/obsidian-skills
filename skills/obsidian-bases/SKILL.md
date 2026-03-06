---
name: obsidian-bases
description: Create and edit Obsidian Bases (.base files) with views, filters, formulas, and summaries. Use when working with .base files, creating database-like views of notes, or when the user mentions Bases, table views, card views, filters, or formulas in Obsidian.
---

# Obsidian Bases Skill

## Quick Start Workflow

1. **Create the `.base` file** — Give it a descriptive name (e.g., `Tasks.base`) and open it in Obsidian.
2. **Add minimal global filters** — Scope to the right notes (e.g., `file.hasTag("task")` or `file.inFolder("Projects")`).
3. **Define at least one view** — Start with a `table` view and list the properties you want in `order`.
4. **Test in Obsidian** — Open the file; if no results appear, see Validation below.
5. **Iterate** — Add formulas, extra views, groupBy, summaries, and per-view filters incrementally.

### Validation Checklist

If a Base shows no results or errors, check:
1. **Filter syntax errors** — Strings with operators must be quoted: `'status == "done"'`.
2. **Property name mismatch** — Names must exactly match the frontmatter keys in target notes.
3. **Missing file extension filter** — Add `'file.ext == "md"'` if non-note files are being picked up.
4. **YAML quoting** — Use single quotes around formula strings that contain double quotes.

## File Format

Base files use the `.base` extension and contain valid YAML. They can also be embedded in Markdown code blocks.

## Complete Schema

```yaml
# Global filters apply to ALL views in the base
filters:
  # Can be a single filter string
  # OR a recursive filter object with and/or/not
  and: []
  or: []
  not: []

# Define formula properties that can be used across all views
formulas:
  formula_name: 'expression'

# Configure display names and settings for properties
properties:
  property_name:
    displayName: "Display Name"
  formula.formula_name:
    displayName: "Formula Display Name"
  file.ext:
    displayName: "Extension"

# Define custom summary formulas
summaries:
  custom_summary_name: 'values.mean().round(3)'

# Define one or more views
views:
  - type: table | cards | list | map
    name: "View Name"
    limit: 10                    # Optional: limit results
    groupBy:                     # Optional: group results
      property: property_name
      direction: ASC | DESC
    filters:                     # View-specific filters
      and: []
    order:                       # Properties to display in order
      - file.name
      - property_name
      - formula.formula_name
    summaries:                   # Map properties to summary formulas
      property_name: Average
```

## Filter Syntax

Filters narrow down results. They can be applied globally or per-view.

### Filter Structure

```yaml
# Single filter
filters: 'status == "done"'

# AND - all conditions must be true
filters:
  and:
    - 'status == "done"'
    - 'priority > 3'

# OR - any condition can be true
filters:
  or:
    - 'file.hasTag("book")'
    - 'file.hasTag("article")'

# NOT - exclude matching items
filters:
  not:
    - 'file.hasTag("archived")'

# Nested filters
filters:
  or:
    - file.hasTag("tag")
    - and:
        - file.hasTag("book")
        - file.hasLink("Textbook")
    - not:
        - file.hasTag("book")
        - file.inFolder("Required Reading")
```

### Filter Operators

| Operator | Description |
|----------|-------------|
| `==` | equals |
| `!=` | not equal |
| `>` | greater than |
| `<` | less than |
| `>=` | greater than or equal |
| `<=` | less than or equal |
| `&&` | logical and |
| `\|\|` | logical or |
| <code>!</code> | logical not |

## Properties

### Three Types of Properties

1. **Note properties** - From frontmatter: `note.author` or just `author`
2. **File properties** - File metadata: `file.name`, `file.mtime`, etc.
3. **Formula properties** - Computed values: `formula.my_formula`

### File Properties Reference

| Property | Type | Description |
|----------|------|-------------|
| `file.name` | String | File name |
| `file.basename` | String | File name without extension |
| `file.path` | String | Full path to file |
| `file.folder` | String | Parent folder path |
| `file.ext` | String | File extension |
| `file.size` | Number | File size in bytes |
| `file.ctime` | Date | Created time |
| `file.mtime` | Date | Modified time |
| `file.tags` | List | All tags in file |
| `file.links` | List | Internal links in file |
| `file.backlinks` | List | Files linking to this file |
| `file.embeds` | List | Embeds in the note |
| `file.properties` | Object | All frontmatter properties |

### The `this` Keyword

- In main content area: refers to the base file itself
- When embedded: refers to the embedding file
- In sidebar: refers to the active file in main content

## Formula Syntax

Formulas compute values from properties. Defined in the `formulas` section.

```yaml
formulas:
  # Simple arithmetic
  total: "price * quantity"

  # Conditional logic
  status_icon: 'if(done, "✅", "⏳")'

  # String formatting
  formatted_price: 'if(price, price.toFixed(2) + " dollars")'

  # Date formatting
  created: 'file.ctime.format("YYYY-MM-DD")'

  # Calculate days since created (use .days for Duration)
  days_old: '(now() - file.ctime).days'

  # Calculate days until due date
  days_until_due: 'if(due_date, (date(due_date) - today()).days, "")'
```

## Functions Reference

### Global Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `date()` | `date(string): date` | Parse string to date. Format: `YYYY-MM-DD HH:mm:ss` |
| `duration()` | `duration(string): duration` | Parse duration string |
| `now()` | `now(): date` | Current date and time |
| `today()` | `today(): date` | Current date (time = 00:00:00) |
| `if()` | `if(condition, trueResult, falseResult?)` | Conditional |
| `min()` | `min(n1, n2, ...): number` | Smallest number |
| `max()` | `max(n1, n2, ...): number` | Largest number |
| `number()` | `number(any): number` | Convert to number |
| `link()` | `link(path, display?): Link` | Create a link |
| `list()` | `list(element): List` | Wrap in list if not already |
| `file()` | `file(path): file` | Get file object |
| `image()` | `image(path): image` | Create image for rendering |
| `icon()` | `icon(name): icon` | Lucide icon by name |
| `html()` | `html(string): html` | Render as HTML |
| `escapeHTML()` | `escapeHTML(string): string` | Escape HTML characters |

### Type Functions

| Type | Function | Signature | Description |
|------|----------|-----------|-------------|
| Any | `isTruthy()` | `any.isTruthy(): boolean` | Coerce to boolean |
| Any | `isType()` | `any.isType(type): boolean` | Check type |
| Any | `toString()` | `any.toString(): string` | Convert to string |
| String | `contains()` | `string.contains(value): boolean` | Check substring |
| String | `containsAll()` | `string.containsAll(...values): boolean` | All substrings present |
| String | `containsAny()` | `string.containsAny(...values): boolean` | Any substring present |
| String | `startsWith()` | `string.startsWith(query): boolean` | Starts with query |
| String | `endsWith()` | `string.endsWith(query): boolean` | Ends with query |
| String | `isEmpty()` | `string.isEmpty(): boolean` | Empty or not present |
| String | `lower()` | `string.lower(): string` | To lowercase |
| String | `title()` | `string.title(): string` | To Title Case |
| String | `trim()` | `string.trim(): string` | Remove whitespace |
| String | `replace()` | `string.replace(pattern, replacement): string` | Replace pattern |
| String | `repeat()` | `string.repeat(count): string` | Repeat string |
| String | `reverse()` | `string.reverse(): string` | Reverse string |
| String | `slice()` | `string.slice(start, end?): string` | Substring |
| String | `split()` | `string.split(separator, n?): list` | Split to list |
| String | `length` | `string.length` | Character count (field) |
| Number | `abs()` | `number.abs(): number` | Absolute value |
| Number | `ceil()` | `number.ceil(): number` | Round up |
| Number | `floor()` | `number.floor(): number` | Round down |
| Number | `round()` | `number.round(digits?): number` | Round to digits |
| Number | `toFixed()` | `number.toFixed(precision): string` | Fixed-point notation |
| Number | `isEmpty()` | `number.isEmpty(): boolean` | Not present |
| List | `contains()` | `list.contains(value): boolean` | Element exists |
| List | `containsAll()` | `list.containsAll(...values): boolean` | All elements exist |
| List | `containsAny()` | `list.containsAny(...values): boolean` | Any element exists |
| List | `filter()` | `list.filter(expression): list` | Filter by condition (uses `value`, `index`) |
| List | `map()` | `list.map(expression): list` | Transform elements (uses `value`, `index`) |
| List | `reduce()` | `list.reduce(expression, initial): any` | Reduce to single value (uses `value`, `index`, `acc`) |
| List | `flat()` | `list.flat(): list` | Flatten nested lists |
| List | `join()` | `list.join(separator): string` | Join to string |
| List | `reverse()` | `list.reverse(): list` | Reverse order |
| List | `slice()` | `list.slice(start, end?): list` | Sublist |
| List | `sort()` | `list.sort(): list` | Sort ascending |
| List | `unique()` | `list.unique(): list` | Remove duplicates |
| List | `isEmpty()` | `list.isEmpty(): boolean` | No elements |
| List | `length` | `list.length` | Element count (field) |
| File | `asLink()` | `file.asLink(display?): Link` | Convert to link |
| File | `hasLink()` | `file.hasLink(otherFile): boolean` | Has link to file |
| File | `hasTag()` | `file.hasTag(...tags): boolean` | Has any of the tags |
| File | `hasProperty()` | `file.hasProperty(name): boolean` | Has property |
| File | `inFolder()` | `file.inFolder(folder): boolean` | In folder or subfolder |
| Link | `asFile()` | `link.asFile(): file` | Get file object |
| Link | `linksTo()` | `link.linksTo(file): boolean` | Links to file |
| Object | `isEmpty()` | `object.isEmpty(): boolean` | No properties |
| Object | `keys()` | `object.keys(): list` | List of keys |
| Object | `values()` | `object.values(): list` | List of values |
| Regexp | `matches()` | `regexp.matches(string): boolean` | Test if matches |

### Date Functions & Fields

**Fields:** `date.year`, `date.month`, `date.day`, `date.hour`, `date.minute`, `date.second`, `date.millisecond`

| Function | Signature | Description |
|----------|-----------|-------------|
| `date()` | `date.date(): date` | Remove time portion |
| `format()` | `date.format(string): string` | Format with Moment.js pattern |
| `time()` | `date.time(): string` | Get time as string |
| `relative()` | `date.relative(): string` | Human-readable relative time |
| `isEmpty()` | `date.isEmpty(): boolean` | Always false for dates |

### Duration Type

When subtracting two dates, the result is a **Duration** type (not a number). Duration has its own properties and methods.

**Duration Fields:** `duration.days`, `duration.hours`, `duration.minutes`, `duration.seconds`, `duration.milliseconds` — all return total units as Number.

**IMPORTANT:** Duration does NOT support `.round()`, `.floor()`, `.ceil()` directly. Access a numeric field first, then apply number functions.

```yaml
# CORRECT
"(date(due_date) - today()).days"           # Returns number of days
"(now() - file.ctime).days.round(0)"        # Rounded days
"(now() - file.ctime).hours.round(0)"       # Rounded hours

# WRONG - will cause error:
# "((date(due) - today()) / 86400000).round(0)"
```

### Date Arithmetic

```yaml
# Duration units: y/year/years, M/month/months, d/day/days,
#                 w/week/weeks, h/hour/hours, m/minute/minutes, s/second/seconds

"date + \"1M\""           # Add 1 month
"now() + \"1 day\""       # Tomorrow
"today() + \"7d\""        # A week from today
"(now() - file.ctime).days"   # Days since created (Duration → Number)
"now() + (duration('1d') * 2)"
```

## View Types

### Table View

```yaml
views:
  - type: table
    name: "My Table"
    order:
      - file.name
      - status
      - due_date
    summaries:
      price: Sum
      count: Average
```

### Cards View

```yaml
views:
  - type: cards
    name: "Gallery"
    order:
      - file.name
      - cover_image
      - description
```

### List View

```yaml
views:
  - type: list
    name: "Simple List"
    order:
      - file.name
      - status
```

### Map View

Requires latitude/longitude properties and the Maps community plugin.

```yaml
views:
  - type: map
    name: "Locations"
```

## Default Summary Formulas

| Name | Input Type | Description |
|------|------------|-------------|
| `Average` | Number | Mathematical mean |
| `Min` | Number | Smallest number |
| `Max` | Number | Largest number |
| `Sum` | Number | Sum of all numbers |
| `Range` | Number | Max - Min |
| `Median` | Number | Mathematical median |
| `Stddev` | Number | Standard deviation |
| `Earliest` | Date | Earliest date |
| `Latest` | Date | Latest date |
| `Range` | Date | Latest - Earliest |
| `Checked` | Boolean | Count of true values |
| `Unchecked` | Boolean | Count of false values |
| `Empty` | Any | Count of empty values |
| `Filled` | Any | Count of non-empty values |
| `Unique` | Any | Count of unique values |

## Complete Example — Task Tracker Base

```yaml
filters:
  and:
    - file.hasTag("task")
    - 'file.ext == "md"'

formulas:
  days_until_due: 'if(due, (date(due) - today()).days, "")'
  is_overdue: 'if(due, date(due) < today() && status != "done", false)'
  priority_label: 'if(priority == 1, "🔴 High", if(priority == 2, "🟡 Medium", "🟢 Low"))'

properties:
  status:
    displayName: Status
  formula.days_until_due:
    displayName: "Days Until Due"
  formula.priority_label:
    displayName: Priority

views:
  - type: table
    name: "Active Tasks"
    filters:
      and:
        - 'status != "done"'
    order:
      - file.name
      - status
      - formula.priority_label
      - due
      - formula.days_until_due
    groupBy:
      property: status
      direction: ASC
    summaries:
      formula.days_until_due: Average

  - type: table
    name: "Completed"
    filters:
      and:
        - 'status == "done"'
    order:
      - file.name
      - completed_date
```

## Common Patterns

### Filter by Tag
```yaml
filters:
  and:
    - file.hasTag("project")
```

### Filter by Folder
```yaml
filters:
  and:
    - file.inFolder("Notes")
```

### Filter by Date Range
```yaml
filters:
  and:
    - 'file.mtime > now() - "7d"'
```

### Filter by Property Value
```yaml
filters:
  and:
    - 'status == "active"'
    - 'priority >= 3'
```

### Combine Multiple Conditions
```yaml
filters:
  or:
    - and:
        - file.hasTag("important")
        - 'status != "done"'
    - and:
        - 'priority == 1'
        - 'due != ""'
```

## Embedding Bases

```markdown
![[MyBase.base]]

<!-- Specific view -->
![[MyBase.base#View Name]]
```

## YAML Quoting Rules

- Use single quotes for formulas containing double quotes: `'if(done, "Yes", "No")'`
- Use double quotes for simple strings: `"My View Name"`
- Escape nested quotes properly in complex expressions

## References

- [Bases Syntax](https://help.obsidian.md/bases/syntax)
- [Functions](https://help.obsidian.md/bases/functions)
- [Views](https://help.obsidian.md/bases/views)
- [Formulas](https://help.obsidian.md/formulas)
