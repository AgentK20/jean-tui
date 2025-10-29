# Testing Guide for gcool Modal Refactoring

## Automated Tests

### Running Unit Tests

```bash
go test ./tui -v
```

**Expected Output:** All 14 tests should pass
```
=== RUN   TestSearchBasedModalInput_NavigateUp
--- PASS: TestSearchBasedModalInput_NavigateUp (0.00s)
=== RUN   TestSearchBasedModalInput_NavigateDown
--- PASS: TestSearchBasedModalInput_NavigateDown (0.00s)
[... 12 more tests ...]
PASS
ok  	github.com/coollabsio/gcool/tui	0.422s
```

**Tests Cover:**
- Navigation (up/down) in search and list modals
- Focus transitions between search input and list
- Type-to-search functionality
- Enter key confirmation
- Cancel button behavior
- Escape key handling
- Boundary checking (preventing overflow)
- Custom key handling (e.g., 'd' for delete in sessions)

---

## Manual Testing Guide

### Prerequisites

1. Navigate to a git repository:
   ```bash
   cd /path/to/your/git/repo
   ```

2. Build gcool:
   ```bash
   go build -o gcool
   ```

3. Run gcool:
   ```bash
   ./gcool
   ```

---

### Test Scenario 1: Branch Select Modal (Type 'a')

**Purpose:** Test search-based modal with type-to-search

**Test Steps:**

1. Press `a` to open "Create worktree from existing branch" modal
2. You should see a search input and branch list below it

**Test Cases:**

#### 1a. Navigation in list
- Press `↓` (down arrow) → should select next branch
- Press `↑` (up arrow) → should select previous branch
- Press `j` → should select next branch (vim keybinding)
- Press `k` → should select previous branch (vim keybinding)

**Expected:** List selection moves smoothly, showing `›` marker on selected branch

#### 1b. Focus transition (search to list)
- Press `↓` from search input → focus moves to branch list
- First down press should NOT execute anything, just move focus to list

**Expected:** Visual feedback shows list is now focused (selected branch shows `›`)

#### 1c. Type-to-search from list
- Highlight any branch with arrow keys to get into list
- Start typing (e.g., `m` for "main") → search input should receive the character
- Continue typing (e.g., `ai`) → filter results in real-time
- Press `backspace` → character should be deleted from search

**Expected:**
- List filters as you type
- Selected branch resets to first match
- Search input shows typed text
- Original list is restored when search is cleared

#### 1d. Tab navigation
- From search input: Press `tab` → moves to branch list
- From list: Press `tab` → moves to OK button (highlighted differently)
- From OK button: Press `tab` → moves to Cancel button
- From Cancel: Press `tab` → wraps back to search input

**Expected:** Visual focus indicator moves to each element (different highlight color)

#### 1e. Enter key behavior
- In search input: Press `enter` → moves focus to OK button (doesn't create worktree)
- In branch list: Press `enter` → moves focus to OK button (doesn't create worktree)
- On OK button: Press `enter` → creates worktree

**Expected:** Worktree only created when OK button has focus

#### 1f. Cancel button behavior
- Navigate to Cancel button using `tab`
- Press `enter` → modal closes WITHOUT creating worktree

**Expected:** Modal closes, no worktree created, back to main view

#### 1g. Escape key
- At any point in modal: Press `esc` → modal closes

**Expected:** Modal closes, no worktree created

---

### Test Scenario 2: Checkout Branch Modal (Type 'C')

**Purpose:** Test search-based modal with checkout action

**Test Steps:**

1. Press `C` (Shift+C) to open "Checkout branch" modal
2. Same behavior as branch select modal, but action is "Checkout" instead of "Create"

**Test Cases:**

#### 2a. Type-to-search
- In branch list, type `dev` → filters to branches containing "dev"
- Press `backspace` → removes character from search

**Expected:** Branch list filters in real-time

#### 2b. Enter confirmation
- Navigate to branch using arrows
- Press `tab` until on "Checkout" button
- Press `enter` → checks out selected branch
- Status bar shows: "Checking out branch: <branch-name>"

**Expected:** Branch is checked out successfully

#### 2c. Cancel without checkout
- Navigate to Cancel button
- Press `enter` → modal closes without checkout

**Expected:** Modal closes, currently checked out branch unchanged

---

### Test Scenario 3: Change Base Branch Modal (Type 'c')

**Purpose:** Test search-based modal with config save

**Test Steps:**

1. Press `c` to open "Change base branch" modal
2. Select or search for a branch

**Test Cases:**

#### 3a. Type-to-search
- In list, type branch name → filters in real-time

**Expected:** List filters as you type

#### 3b. Set base branch
- Navigate to branch
- Press `tab` to "Set" button
- Press `enter` → sets base branch and saves to config

**Expected:** Status shows "Base branch set to: <branch> (saved)"

#### 3c. Reject cancel
- Navigate to Cancel button
- Press `enter` → closes without changing base branch

**Expected:** Base branch unchanged, config not modified

---

### Test Scenario 4: Session List Modal (Type 'S')

**Purpose:** Test list selection modal with custom key handling

**Test Steps:**

1. Create a tmux session:
   ```bash
   tmux new-session -d -s test-session
   ```

2. In gcool main view, press `S` to open session list modal

**Test Cases:**

#### 4a. Navigation in session list
- Press `↓` → selects next session
- Press `↑` → selects previous session
- Boundary check: Can't go below 0 or beyond last session

**Expected:** Selected session shows with marker

#### 4b. Attach to session
- Select a session
- Press `enter` → attaches to session in tmux

**Expected:** Gcool exits, you're now in tmux session

#### 4c. Delete session
- Select a session
- Press `d` → kills the session
- Status shows: "Session killed"
- Session list refreshes

**Expected:** Session is removed from list

#### 4d. Escape closes
- Press `esc` → closes session modal

**Expected:** Back to main worktree view

---

### Test Scenario 5: Editor Selection Modal (Type 'e')

**Purpose:** Test list selection modal for editor preferences

**Test Steps:**

1. In main view, press `e` to open editor selection modal

**Test Cases:**

#### 5a. Navigation
- Press `↓` → selects next editor
- Press `↑` → selects previous editor

**Expected:** Selection moves through editor list (code, cursor, nvim, vim, etc.)

#### 5b. Select editor
- Navigate to desired editor
- Press `enter` → saves as preferred editor

**Expected:** Status shows "Editor set to: <editor>"

#### 5c. Escape cancels
- Press `esc` → closes without changing editor preference

**Expected:** Modal closes, previous editor setting preserved

---

## Test Coverage Summary

| Feature | Search Modal | List Modal |
|---------|---|---|
| Navigate up/down | ✓ | ✓ |
| Type-to-search | ✓ | ✗ |
| Tab cycling | ✓ | ✗ |
| Enter confirmation | ✓ | ✓ |
| Cancel button | ✓ | ✗ |
| Escape closes | ✓ | ✓ |
| Custom keys (d) | ✗ | ✓ |

---

## Key Changes to Verify

1. **Modal Navigation is Consistent**
   - All modals respond to arrow keys consistently
   - Tab cycling works in all modals
   - Escape closes all modals

2. **Type-to-Search Works Everywhere**
   - Can type while in branch list (search modals)
   - Search filters list in real-time
   - Backspace clears characters

3. **Button Focus Works Correctly**
   - OK/Set/Checkout buttons only execute when focused
   - Cancel button closes without action
   - No accidental operations

4. **Refactoring Didn't Break Functionality**
   - All original features still work
   - Modals behave the same from user perspective
   - Code is just cleaner internally

---

## Troubleshooting

### Modal doesn't respond to keyboard
- Make sure you're not in the main view (press `q` to exit modal if stuck)
- Check that modal is actually open (see help bar at bottom)

### Type-to-search not working in list
- Make sure you're in a search-based modal (branchSelect, checkoutBranch, changeBaseBranch)
- List modals (sessionList, editorSelect) don't have search feature

### Can't exit modal
- Press `esc` to close any modal
- If stuck, press `q` as backup (if implemented)

---

## Performance Notes

All keyboard input should be instantaneous with no lag. If you notice:
- Slow filtering → May be due to large branch list
- Slow navigation → Check system resources

Test with repositories of different sizes (small, medium, large) to ensure performance is acceptable.
