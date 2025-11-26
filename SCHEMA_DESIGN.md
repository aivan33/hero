# Poker Scenario Config Schema - Design Rationale

## Overview
This schema defines a comprehensive JSON structure for poker scenarios in educational content, optimized for vertical 1080x1920 format visualizations.

## Schema Structure & Reasoning

### 1. **Board State** (`board`)
```json
{
  "street": "flop",
  "cards": ["Ah", "Kd", "7s"]
}
```

**Design Decisions:**
- **Explicit street tracking**: Separate `street` field prevents ambiguity when cards array might have varying lengths
- **Card notation**: Using rank+suit format (`"As"`, `"Kd"`) is poker-standard, compact, and easily parseable
- **Suit abbreviations**: `s=spades, h=hearts, d=diamonds, c=clubs` - industry standard
- **Rank abbreviations**: `A,K,Q,J,T,9-2` where `T=10` - avoids two-character ranks

### 2. **Player Information** (`hero`, `villain`)
```json
{
  "position": "BTN",
  "holeCards": ["Ac", "Qh"],
  "stack": 85
}
```

**Design Decisions:**
- **Position enums**: Standard poker position abbreviations (UTG, BB, BTN, etc.) - recognizable and space-efficient
- **Optional hole cards**: Hero's cards may be hidden in range analysis scenarios
- **Stack sizes in BB**: Big blinds are the poker standard for normalizing stack depths across different stakes
- **Separate hero/villain objects**: Clear distinction, easily extensible to multiplayer scenarios

### 3. **Pot Information** (`pot`)
```json
{
  "size": 15,
  "currentBet": 10
}
```

**Design Decisions:**
- **Separation of pot and bet**: Critical distinction for calculating pot odds (need to call 10 to win 15+10)
- **Default currentBet to 0**: Clean handling of check scenarios
- **All amounts in BB**: Consistent with stack representation

### 4. **Range Matrix** (`rangeMatrix`)
```json
{
  "displayFor": "villain",
  "matrix": [[1.0, 0.8, ...], ...],
  "colorScheme": "heatmap"
}
```

**Design Decisions:**
- **13x13 structure**: Standard poker range matrix
  - Row 0: `AA, AKs, AQs, AJs, ATs, A9s, A8s, A7s, A6s, A5s, A4s, A3s, A2s`
  - Row 1: `AKo, KK, KQs, KJs, KTs, K9s, K8s, K7s, K6s, K5s, K4s, K3s, K2s`
  - Diagonal = pairs, upper-right = suited, lower-left = offsuit
- **Frequencies (0-1)**: Standard representation for weighted ranges
  - 1.0 = 100% of this hand in range
  - 0.5 = 50% of this hand
  - 0.0 = never in range
- **displayFor label**: Critical for educational context - explicitly shows whose perspective
- **Optional colorScheme**: Allows visual customization without changing data

### 5. **Action History** (`actionHistory`)
```json
[
  {
    "street": "preflop",
    "player": "hero",
    "action": "raise",
    "amount": 2.5
  }
]
```

**Design Decisions:**
- **Array structure**: Chronological order naturally preserved
- **Street per action**: Allows clear visualization of multi-street sequences
- **Player enum**: Includes "other" for multiway pots
- **Conditional amount**: Only required for bet/raise/allin actions
- **Action enums**: Standard poker actions (fold, check, call, bet, raise, allin)

### 6. **Metadata** (`metadata`)
```json
{
  "title": "BTN vs BB: Top Pair Decision",
  "difficulty": "intermediate",
  "tags": ["continuation-bet", "top-pair"]
}
```

**Design Decisions:**
- **Entirely optional**: Core visualization doesn't depend on it
- **Title field**: Useful for educational labeling
- **Difficulty levels**: Helps content organization
- **Tags array**: Flexible categorization for filtering/search

## Matrix Layout Reference

The 13x13 matrix follows standard poker range notation:

```
     A    K    Q    J    T    9    8    7    6    5    4    3    2
A   AA   AKs  AQs  AJs  ATs  A9s  A8s  A7s  A6s  A5s  A4s  A3s  A2s
K   AKo  KK   KQs  KJs  KTs  K9s  K8s  K7s  K6s  K5s  K4s  K3s  K2s
Q   AQo  KQo  QQ   QJs  QTs  Q9s  Q8s  Q7s  Q6s  Q5s  Q4s  Q3s  Q2s
J   AJo  KJo  QJo  JJ   JTs  J9s  J8s  J7s  J6s  J5s  J4s  J3s  J2s
T   ATo  KTo  QTo  JTo  TT   T9s  T8s  T7s  T6s  T5s  T4s  T3s  T2s
9   A9o  K9o  Q9o  J9o  T9o  99   98s  97s  96s  95s  94s  93s  92s
8   A8o  K8o  Q8o  J8o  T8o  98o  88   87s  86s  85s  84s  83s  82s
7   A7o  K7o  Q7o  J7o  T7o  97o  87o  77   76s  75s  74s  73s  72s
6   A6o  K6o  Q6o  J6o  T6o  96o  86o  76o  66   65s  64s  63s  62s
5   A5o  K5o  Q5o  J5o  T5o  95o  85o  75o  65o  55   54s  53s  52s
4   A4o  K4o  Q4o  J4o  T4o  94o  84o  74o  64o  54o  44   43s  42s
3   A3o  K3o  Q3o  J3o  T3o  93o  83o  73o  63o  53o  43o  33   32s
2   A2o  K2o  Q2o  J2o  T2o  92o  82o  72o  62o  52o  42o  32o  22
```

- **Diagonal**: Pocket pairs (AA, KK, QQ, etc.)
- **Upper-right triangle**: Suited hands (indicated by 's')
- **Lower-left triangle**: Offsuit hands (indicated by 'o')

## Extensibility Considerations

The schema is designed for future expansion:

1. **Multiway pots**: `hero`/`villain` structure can be replaced with `players` array
2. **Additional streets**: `board.cards` array naturally extends
3. **Multiple ranges**: Could add `ranges` array with multiple matrices
4. **Stack history**: Could track stack changes per action in `actionHistory`
5. **Equity calculations**: Could add `equity` field to show win percentages
6. **Hand history IDs**: Could add external reference for real hand examples

## Validation Notes

The JSON Schema includes:
- Pattern matching for card notation (e.g., `^[AKQJT98765432][shdc]$`)
- Min/max constraints on arrays (2 hole cards, 0-5 board cards)
- Enum validation for positions, actions, streets
- Range validation for frequencies (0-1)

## Usage Example

See `example-scenario.json` for a complete working example of a BTN vs BB scenario on an Ace-high flop with villain's calling range displayed.
