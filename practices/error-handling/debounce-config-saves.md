---
concern: error-handling
tech: [react, typescript, sqlite]
priority: recommended
source-repo: RepoTracker
applies-to: [react, typescript]
---
# Debounce Database Writes on User Input

## PATTERN
When user input (text fields, number inputs, sliders) triggers a database or API write, debounce the write so it fires only after input settles -- not on every keystroke.

## WHY
Typing "365" in a number input fires 3 onChange events ("3", "36", "365"), producing 3 database writes. On SQLite, this causes unnecessary WAL churn. On remote databases or APIs, it wastes bandwidth and can trigger rate limits. On slow storage, intermediate writes can queue up and cause visible lag.

## EXAMPLE
From RepoTracker (`src/components/settings/GeneralSettings.tsx`):

Wrong -- writes on every keystroke:
```tsx
function handleChange(value: string) {
  setLocalValue(value);
  const num = parseInt(value, 10);
  if (num > 0) setConfig("threshold", String(num)); // fires on every key
}
```

Right -- debounce the persistence:
```tsx
const saveConfig = useMemo(
  () => debounce((key: string, value: string) => setConfig(key, value), 500),
  [],
);

function handleChange(value: string) {
  setLocalValue(value);
  const num = parseInt(value, 10);
  if (num > 0) saveConfig("threshold", String(num));
}

// Clean up on unmount
useEffect(() => () => saveConfig.cancel(), [saveConfig]);
```

Or save on blur instead of on change:
```tsx
<input
  value={localValue}
  onChange={(e) => setLocalValue(e.target.value)}
  onBlur={() => setConfig("threshold", localValue)}
/>
```

## CHECK
- [ ] Search for `setConfig`, `saveSettings`, or database write calls inside `onChange` handlers
- [ ] Check number/text inputs that persist to storage for debouncing
- [ ] Verify no `onBlur` or debounce wrapper exists around the save call

## IMPLEMENT
1. Add a debounce utility (lodash `debounce`, or a 10-line custom implementation)
2. Wrap the storage write in the debounce, keeping local state updates instant
3. Add cleanup in `useEffect` return to cancel pending debounced calls on unmount
4. Alternative: switch to `onBlur`-based saving for simple settings forms

## NOTES
- 300-500ms is a good debounce interval for text input; longer (1000ms) for search-as-you-type.
- Save-on-blur is simpler and avoids the need for a debounce library entirely.
- Discovered during RepoTracker code review.
