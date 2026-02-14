# Drag & Drop Item Allocation Integration Plan

**Date:** 2026-02-14
**Feature:** Replace checkbox-based item allocation with drag-and-drop UI
**Branch:** feat/dragdrop-allocation

## Context

The current paysplitter app has a per-item checkbox UI for "Who's sharing this?" — users must tap checkboxes for each item × each person. With many items and people, this is tedious.

The approved prototype (`prototype-dragdrop.html`) shows a drag-and-drop approach: items appear as chips in an "unassigned pool", users drag them into person zones, and shared items auto-split the cost.

## Architecture

**Single file:** `index.html` (4208 lines) — all HTML/CSS/JS in one file.

**Data model (unchanged):**
- `items[]` — each has `{id, name, price, people: [personName, ...]}`
- `people[]` — array of person names
- `calculate()` + `calculatePersonBreakdown()` — reads `item.people` to compute shares

**Key insight:** The drag-and-drop UI writes to the SAME `item.people[]` array. No data model changes needed. All downstream logic (calculate, breakdown, summary, Firebase sync, undo/redo) works unchanged.

## What Changes

### 1. CSS — Add drag-and-drop styles (~120 lines)

Add these new CSS classes (from prototype, adapted to match app theme):
- `.allocation-section` — container for the allocation UI
- `.items-pool`, `.items-pool.drag-over`, `.items-pool.empty-hint::after` — unassigned items pool
- `.item-chip`, `.item-chip.dragging`, `.item-chip.drag-clone` — draggable item chips
- `.dnd-person-zone`, `.dnd-person-zone.drag-over` — person drop zones (prefix with `dnd-` to avoid conflict with existing `.person-zone` if any)
- `.dnd-person-header`, `.dnd-person-name`, `.dnd-person-total` — person zone header
- `.dnd-person-items` — items within a person zone
- `.shared-badge` — shared item count badge
- `.remove-chip-btn` — remove button on assigned items
- `.mode-toggle`, `.mode-btn`, `.mode-btn.active` — drag/tap mode toggle
- `.assign-all-btn` — "Split all evenly" button

Use the app's existing color scheme: `#7B2D8E` / `#E91E8C` gradient instead of prototype's `#667eea`.

### 2. HTML — Replace Items sub-tab content

**Current Items sub-tab** (`subtab-items`, lines 1827-1899):
```
- Receipt Preview section (KEEP)
- Undo/Redo bar (KEEP)
- Quick Split buttons (KEEP — but rewire to update allocation UI)
- AI Receipt Scanner (KEEP)
- Item list with checkboxes (REPLACE)
- Add Item button (KEEP)
```

**New Items sub-tab layout:**
```
- Receipt Preview section (unchanged)
- Undo/Redo bar (unchanged)
- Quick Split buttons (unchanged — splitAllEqually/splitEachOwn/clearAllAssignments already modify item.people[])
- AI Receipt Scanner (unchanged)
- Item Entry section — simple list of item name + price + delete (NO checkboxes)
- Add Item button (unchanged)
- Mode Toggle (drag/tap)
- Allocation Section:
  - Unassigned Items Pool (chips)
  - Person Drop Zones
  - "Split all items evenly" button
```

**Specifically, replace the `renderItems()` output.** The item rows should show:
- Item name input
- Item price input
- Remove button
- NO checkboxes (remove `.item-people` div entirely)

Then add a new section BELOW the items list for the allocation UI.

### 3. JavaScript — Add drag-and-drop logic (~200 lines)

#### 3a. New functions to add:

```javascript
// Drag & Drop state
let dragState = null;
let cloneEl = null;
let currentAllocMode = 'drag'; // 'drag' or 'tap'
let selectedAllocItemId = null; // for tap mode

// Core drag functions
function startAllocDrag(e, itemId) { ... }
function onAllocDragMove(e) { ... }
function onAllocDragEnd(e) { ... }
function moveAllocClone(x, y) { ... }

// Tap mode
function handleAllocItemTap(itemId) { ... }
function handleAllocPersonTap(person) { ... }

// Mode toggle
function setAllocMode(mode) { ... }

// Render allocation UI
function renderAllocation() { ... }  // renders pool + person zones
function renderAllocPool() { ... }
function renderAllocPersonZones() { ... }

// Helper
function getUnassignedItems() { ... }
function getItemShareCount(itemId) { ... }
```

#### 3b. Modify existing functions:

1. **`renderItems()`** — Remove the checkbox UI. Keep item name/price/delete. Call `renderAllocation()` at the end.

2. **`togglePerson()`** — Keep for backward compatibility but it will be called less. The allocation UI uses `assignAllocItem()` and `unassignAllocItem()` instead.

3. **`addItem()`** — After adding, also call `renderAllocation()`.

4. **`removeItem()`** — After removing, also call `renderAllocation()`.

5. **`splitAllEqually()`** — Already modifies `item.people[]` correctly. Just need to call `renderAllocation()` after.

6. **`splitEachOwn()`** — Same, already clears `item.people[]`. Call `renderAllocation()`.

7. **`clearAllAssignments()`** — Same pattern.

8. **`restoreState()`** — Call `renderAllocation()` after restoring.

9. **`loadSessionFromFirebase()`** — Call `renderAllocation()` after loading.

#### 3c. Assignment functions (write to item.people[]):

```javascript
function assignAllocItem(person, itemId) {
    const item = items.find(i => i.id === itemId);
    if (item && !item.people.includes(person)) {
        saveStateForUndo();
        item.people.push(person);
        renderItems(); // re-renders items + allocation
        calculate();
        syncSessionToFirebase();
    }
}

function unassignAllocItem(person, itemId) {
    const item = items.find(i => i.id === itemId);
    if (item) {
        saveStateForUndo();
        item.people = item.people.filter(p => p !== person);
        renderItems();
        calculate();
        syncSessionToFirebase();
    }
}
```

### 4. Integration with existing systems

| System | Impact | Action |
|--------|--------|--------|
| Firebase sync | None | `item.people[]` is synced via `syncSessionToFirebase()` — works unchanged |
| Undo/Redo | None | `saveStateForUndo()` captures `items[]` which includes `people[]` |
| URL sharing | None | Encodes `item.people[]` — works unchanged |
| History | None | Saves `item.people[]` — works unchanged |
| Calculate | None | Reads `item.people[]` — works unchanged |
| Summary | None | `calculatePersonBreakdown()` reads `item.people[]` |
| Receipt scan | Minor | After `confirmPreview()`, call `renderAllocation()` |

## Implementation Tasks

### Task 1: Add CSS styles
- Add all drag-and-drop CSS classes after line ~1740 (before `</style>`)
- Use app colors: `#7B2D8E` primary, `#E91E8C` accent
- Prefix allocation-specific classes with `dnd-` where they might conflict

### Task 2: Add allocation HTML section
- In `subtab-items`, AFTER the `<button class="add-item-btn">` line (~1896), add:
  - Mode toggle (drag/tap)
  - Allocation container div (`<div id="allocationSection">`)
    - Unassigned pool div (`<div id="allocPool">`)
    - Person zones div (`<div id="allocPersonZones">`)
    - Split evenly button

### Task 3: Modify renderItems() to remove checkboxes
- Remove the `.item-people` div from the template
- Remove the "Who's sharing this?" label
- Keep item name input, price input, remove button

### Task 4: Add JavaScript drag-and-drop logic
- Add all new functions listed in 3a
- Modify existing functions listed in 3b
- Wire up the assignment functions (3c)

### Task 5: Test integration
- Verify items can be dragged to people
- Verify shared items show correct split
- Verify calculate() and breakdown work
- Verify undo/redo works with allocations
- Verify Firebase sync works
- Verify receipt scan → allocation flow works

## Files Modified
- `index.html` — CSS + HTML + JS changes (single file)

## Testing
Since this is a single HTML file with no build system, testing is manual:
1. Open in browser
2. Add people
3. Add items (manually or via AI scan)
4. Drag items to people
5. Check summary shows correct amounts
6. Check undo/redo works
