# Readability Audit

## Current problematic values:
- `--muted-foreground: oklch(0.58 0.015 250)` — this is the LOW CONTRAST gray (approx #6B7280)
- `--secondary-foreground: oklch(0.80 0.01 250)` — decent but could be brighter
- `--foreground: oklch(0.93 0.005 250)` — good, near-white
- `.verdict-explanation color: oklch(0.70 0.01 250)` — too dark for explanations

## Target values (from user spec):
- Primary text: #E6EDF3 → oklch(0.93 0.01 250) (already close to --foreground)
- Secondary text: #B8C1CC → oklch(0.78 0.01 250)
- Muted text: #8A94A6 → oklch(0.63 0.015 250)

## Key changes needed:
1. --muted-foreground: 0.58 → 0.65 (from #6B7280 to ~#8A94A6)
2. --secondary-foreground: 0.80 → 0.80 (already ok, maps to ~#B8C1CC)
3. verdict-explanation: 0.70 → 0.78 (make readable)
4. constraint explanations in components: check for hardcoded low-contrast colors
5. Progress bars: thicken from 6px to 8px
