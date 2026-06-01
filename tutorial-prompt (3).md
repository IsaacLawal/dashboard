Build a single self-contained HTML file called index.html. It must work 
when opened via VS Code Live Server or dragged directly into a browser — 
no build step, no npm, no external CSS or JS files. All styles inline in 
one <style> tag, all JavaScript inline in one <script> tag at the bottom. 
External CDN links are allowed only for Supabase.

The app is called TimeBlox — a dark-mode time-blocking and productivity 
web app that works on desktop and mobile.

---

Visual style (whole file)

Dark theme. Page background #0D0D0F. Body font: -apple-system, 
BlinkMacSystemFont, "Inter", "Segoe UI", Roboto, Helvetica, Arial, 
sans-serif. Mono font (numbers, times, durations): ui-monospace, 
"SF Mono", Menlo, Consolas, monospace.

CSS variables:
  --bg:           #0D0D0F
  --bg-card:      rgba(255,255,255,0.04)
  --bg-hover:     rgba(255,255,255,0.07)
  --border:       rgba(255,255,255,0.08)
  --text-primary: #FAFAFA
  --text-secondary: #B8B6B0
  --text-tertiary:  #76746E
  --accent-green:  #6BE3A4
  --accent-blue:   #5B9CF6
  --accent-purple: #A78BFA
  --accent-yellow: #F2C063
  --accent-pink:   #F472B6
  --accent-red:    #FF6B6B
  --success:       #6BE3A4
  --danger:        #FF6B6B

Category colors:
  Fun Block        → --accent-green   #6BE3A4
  Focus Block      → --accent-blue    #5B9CF6
  Night Routine    → --accent-purple  #A78BFA
  Morning Routine  → --accent-yellow  #F2C063
  Custom Block     → --accent-pink    #F472B6

Card chassis: background var(--bg-card), border 1px solid var(--border), 
border-radius 16px, padding 18px 20px, backdrop-filter blur(24px) 
saturate(1.2), box-shadow 0 12px 40px rgba(0,0,0,0.45).

Body: centered, max-width 900px, padding 20px 16px, safe-area-aware. 
On screens ≤ 480px: padding 12px 10px.

---

App shell

At the very top render the app title:
  <h1 class="app-title">TimeBlox</h1>
Styled: font-size 26px, font-weight 800, letter-spacing -0.03em, 
background linear-gradient(180deg, #FFFFFF 0%, #C7C4BC 120%), 
-webkit-background-clip text, -webkit-text-fill-color transparent, 
margin-bottom 16px.

Below the title, a sticky tab bar with five tabs:
  <nav class="tab-bar">
    <button class="tab-btn active" data-tab="calendar">📅 Calendar</button>
    <button class="tab-btn" data-tab="stats">📊 Stats</button>
    <button class="tab-btn" data-tab="templates">⚡ Templates</button>
    <button class="tab-btn" data-tab="todo">✅ To-Do</button>
    <button class="tab-btn" data-tab="quotes">💬 Quotes</button>
  </nav>

Tab bar styling: flex row, gap 4px, background rgba(255,255,255,0.03), 
border 1px solid var(--border), border-radius 14px, padding 5px, 
margin-bottom 20px, position sticky, top 0, z-index 100, 
backdrop-filter blur(20px). Each .tab-btn: flex 1, padding 8px 4px, 
border-radius 10px, font-size 12px, font-weight 600, color 
var(--text-secondary), background transparent, border none, cursor pointer, 
transition all 0.2s. Active tab: background rgba(255,255,255,0.10), 
color var(--text-primary). On screens ≤ 480px: font-size 10px, 
padding 7px 2px.

Each tab content is a <div class="tab-content" id="tab-{name}"> that is 
shown (display block) or hidden (display none) based on active tab. 
Switching tabs is instant with no animation.

---

Tab 1 — Calendar

Structure inside #tab-calendar:

TOP: A calendar navigation row.
  <div class="cal-nav">
    <button id="calPrev">‹</button>
    <div class="cal-view-toggle">
      <button class="cal-view-btn active" data-view="day">Day</button>
      <button class="cal-view-btn" data-view="week">Week</button>
      <button class="cal-view-btn" data-view="month">Month</button>
    </div>
    <button id="calNext">›</button>
  </div>
  <div class="cal-label" id="calLabel"></div>

calLabel shows the current view range:
  Day view: "Monday, June 2" (full weekday, full month, day).
  Week view: "Jun 2 – Jun 8" or "May 26 – Jun 1" for cross-month weeks.
  Month view: "June 2026".

calPrev / calNext move by one day / one week / one month depending on 
current view. Styled as ghost buttons, 32px square, border-radius 8px, 
font-size 18px, color var(--text-secondary).

FOCUS PROGRESS BAR (shown in Day view only):
  <div class="focus-progress" id="focusProgress">
    <div class="focus-progress-label">
      <span id="focusCountText">0 / 0 Focus Blocks</span>
      <span id="focusPct">0%</span>
    </div>
    <div class="focus-bar-track">
      <div class="focus-bar-fill" id="focusBarFill"></div>
    </div>
  </div>

focus-bar-track: height 6px, background rgba(255,255,255,0.08), 
border-radius 99px. focus-bar-fill: height 100%, background 
var(--accent-blue), border-radius 99px, transition width 0.4s 
cubic-bezier(0.22,1,0.36,1). Recalculates instantly on any block 
add / delete / complete / incomplete. Deleted blocks excluded.

BLOCKS AREA:
  <div id="blocksArea"></div>

In Day view: renders all blocks for the selected day sorted by creation 
order. Each block is a card with drag handle for reordering.

In Week view: renders a 7-column week grid (Mon–Sun). Each column has 
a day header (short weekday + date number) and lists that day's block 
count. Clicking a day header switches to Day view for that day.

In Month view: renders a 7-column calendar grid (Mon–Sun headers). 
Each cell shows the date number and a colored dot for each block that day 
(max 3 dots, then +N more). Clicking a cell switches to Day view for 
that date. Cross-month weeks are fully supported — no isolated 
single-day weeks. Cells outside the current month are dimmed 
(opacity 0.35).

ADD BLOCK BUTTON (Day view only):
  <button class="add-block-btn" id="addBlockBtn">+ Add Time Block</button>
Styled: full width, dashed border 1.5px var(--border), border-radius 12px, 
padding 12px, color var(--text-secondary), background transparent, 
font-size 13px, font-weight 600, cursor pointer. Hover: border-color 
var(--accent-blue), color var(--accent-blue).

---

Block card structure

Each block renders as a card inside blocksArea:

  <div class="block-card" data-id="{id}" data-category="{category}">
    <div class="block-header">
      <span class="drag-handle">⋮⋮</span>
      <span class="block-color-dot"></span>
      <span class="block-title-text">{title}</span>
      <span class="block-total-time">{X}m</span>
      <button class="block-toggle">▾</button>
      <button class="block-complete-btn">{done ? ✓ : ○}</button>
      <button class="block-edit-btn">✎</button>
      <button class="block-delete-btn">×</button>
    </div>
    <div class="block-body" style="display:{collapsed?none:block}">
      <div class="block-mood" id="mood-{id}"></div>
      <div class="block-notes" id="notes-{id}"></div>
      <div class="block-checklist" id="checklist-{id}"></div>
      <div class="block-segments" id="segments-{id}"></div>
      <button class="add-segment-btn" data-block="{id}">+ Add Segment</button>
    </div>
  </div>

Block card styling:
border-left: 3.5px solid {category color}, border-radius 14px, 
background var(--bg-card), border 1px solid var(--border), 
margin-bottom 10px, overflow hidden. Completed blocks: opacity 0.55, 
background rgba(107,227,164,0.04).

block-header: flex row, align-items center, gap 8px, padding 12px 14px, 
cursor pointer on toggle.

block-color-dot: 10px circle filled with category color, flex-shrink 0.

block-title-text: flex 1, font-size 14px, font-weight 700, 
color var(--text-primary), click to inline-edit (same pattern as 
dashboard — contentEditable, Enter commits, Escape cancels).

block-total-time: 11px mono, color var(--text-tertiary), auto-calculated 
as sum of all segment durations in minutes. Updates live.

block-toggle: rotates ▾ (expanded) / ▸ (collapsed), 0.2s transition.

block-complete-btn: 28px circle, border 1.5px solid var(--border), 
font-size 13px. Completed: background var(--accent-green), 
border-color var(--accent-green), color #0D0D0F.

block-edit-btn, block-delete-btn: 24px, color var(--text-tertiary), 
hover color var(--text-primary) / var(--accent-red).

drag-handle: ⋮⋮ 14px wide, opacity 0 on idle, opacity 1 on row hover, 
grab cursor, letter-spacing -2px.

Drag-and-drop reordering of blocks within the day using HTML5 
drag/drop — same pattern as dashboard goal list.

---

Block body sections

MOOD REFLECTION (shown after block is marked complete):
  Five emoji buttons in a row: 😁 🙂 😐 😞 💩
  Selected emoji gets background rgba(255,255,255,0.12) and scale 1.15.
  Stored as block.mood. If not yet completed, hide this section.

NOTES:
  A <textarea> or contentEditable div, placeholder "Add notes…", 
  11px, color var(--text-secondary), background transparent, 
  border 1px solid var(--border), border-radius 8px, padding 8px 10px, 
  saves on blur. Hidden if empty and block is not being edited.

CHECKLIST:
  List of { text, done } items attached to the block.
  Each row: checkbox + text (inline-editable) + × delete.
  Checked items: opacity 0.5, line-through.
  Below list: small input + "Add item" button.
  Users can edit and delete checklist items at any time.

SEGMENTS:
  List of timed segments inside the block.
  Each segment row:
    <div class="segment-row">
      <span class="seg-type-badge">{Work|Break|Custom}</span>
      <span class="seg-title">{title or placeholder}</span>
      <span class="seg-duration">{N}m</span>
      <button class="seg-delete">×</button>
    </div>
  seg-type-badge: 9px uppercase mono, padding 2px 6px, border-radius 4px.
    Work → background rgba(91,156,246,0.15), color var(--accent-blue)
    Break → background rgba(107,227,164,0.15), color var(--accent-green)
    Custom → background rgba(244,114,182,0.15), color var(--accent-pink)
  Each segment title is inline-editable. Each segment duration is 
  inline-editable (number input, minutes). On any duration change, 
  block-total-time recalculates immediately.

ADD SEGMENT BUTTON:
  Opens a small inline form directly below the segment list:
    Type selector (Work / Break / Custom)
    Title input (optional)
    Duration input (minutes, required)
    Confirm + Cancel buttons
  On confirm: push segment to block.segments, recalculate total, save, 
  re-render.

---

Block edit modal

Clicking block-edit-btn opens a full modal overlay:
  <div class="modal-overlay" id="blockModal">
    <div class="modal-card">
      <h2 class="modal-title">Edit Block</h2>
      <label>Title <input id="modalTitle"></label>
      <label>Category
        <select id="modalCategory">
          <option>Fun Block</option>
          <option>Focus Block</option>
          <option>Night Routine</option>
          <option>Morning Routine</option>
          <option>Custom Block</option>
        </select>
      </label>
      <label>Color <input type="color" id="modalColor"></label>
      <div class="modal-actions">
        <button id="modalSave">Save</button>
        <button id="modalCancel">Cancel</button>
        <button id="modalSaveAsTemplate">Save as Template</button>
      </div>
    </div>
  </div>

Modal overlay: position fixed, inset 0, background rgba(0,0,0,0.72), 
backdrop-filter blur(8px), display flex, align-items center, 
justify-content center, z-index 1000.

Modal card: var(--bg-card) background, border 1px solid var(--border), 
border-radius 20px, padding 24px, width min(480px, calc(100vw - 32px)), 
max-height 80vh, overflow-y auto.

For Morning Routine and Night Routine blocks, show an extra button 
"Save as Default Routine" in the modal. Clicking it saves the block's 
full structure (segments, checklist, notes) as the master template for 
that routine type under localStorage key routine_template_morning or 
routine_template_night.

When creating a new Morning Routine or Night Routine block, if a master 
template exists for that type, auto-populate segments, checklist, and 
notes from it. The new block is fully independently editable.

---

Tab 2 — Stats

Structure inside #tab-stats:

DAILY FOCUS PROGRESS (today only):
  Same focus progress bar as in Calendar tab — kept in sync with same 
  data. Re-renders whenever blocks change.

TWO STAT CARDS side by side (stack on mobile):

  WEEKLY card:
    Header: "This Week" + date range (Mon–Sun of current week)
    Big number: total Focus Blocks completed this week
    Sub-label: "Focus Blocks completed"
    Below: a simple 7-bar chart (Mon–Sun), each bar height proportional 
    to Focus Blocks completed that day. Bars colored var(--accent-blue), 
    height max 60px, width 100%, gap 4px. Day labels below each bar: 
    3-letter weekday, 9px mono tertiary.

  MONTHLY card:
    Header: "This Month" + month name + year
    Big number: total Focus Blocks completed this month
    Sub-label: "Focus Blocks completed"
    Below: a bar per week in the month (W1–W4/W5), height proportional 
    to completions. Same styling as weekly bars.

Stats rules:
  Weeks run Mon→Sun.
  Cross-month weeks are supported — a week belongs to the month in which 
  Thursday falls (ISO 8601 convention).
  Deleted blocks are never counted.
  Numbers update automatically whenever blocks change.
  No duplicate counting.
  Handles leap years and year transitions.

---

Tab 3 — Templates

Structure inside #tab-templates:

SEARCH + FILTER ROW:
  <div class="tmpl-controls">
    <input id="tmplSearch" placeholder="Search templates…">
    <select id="tmplFilter">
      <option value="">All categories</option>
      <option>Fun Block</option>
      <option>Focus Block</option>
      <option>Night Routine</option>
      <option>Morning Routine</option>
      <option>Custom Block</option>
    </select>
  </div>

TEMPLATE LIST:
  <div id="tmplList"></div>

Each template renders as a card:
  <div class="tmpl-card" data-id="{id}">
    <div class="tmpl-color-bar"></div>  <!-- 4px top border in category color -->
    <div class="tmpl-info">
      <span class="tmpl-name">{name}</span>
      <span class="tmpl-meta">{totalDuration}m · {segCount} segments · {category}</span>
    </div>
    <div class="tmpl-actions">
      <button class="tmpl-import-btn">Import</button>
      <button class="tmpl-rename-btn">✎</button>
      <button class="tmpl-delete-btn">×</button>
    </div>
  </div>

tmpl-card: flex row, align-items center, gap 12px, padding 12px 14px, 
background var(--bg-card), border 1px solid var(--border), 
border-radius 12px, margin-bottom 8px.

tmpl-name: font-size 14px, font-weight 700, color var(--text-primary).
tmpl-meta: font-size 11px, mono, color var(--text-tertiary).

Import button: on click, prompt user to select a date (default today), 
then create a new block on that date using the template's title, 
category, color, segments, notes. The new block is fully independently 
editable. Import feels instant.

Rename: inline edit of tmpl-name (same contentEditable pattern).

Templates support drag-and-drop reordering within the list.

Empty state: "No templates yet — save a Focus Block as a template from 
the Calendar tab." 12px tertiary italic, centered, 24px vertical padding.

---

Tab 4 — To-Do

Structure inside #tab-todo:

TWO SECTIONS stacked:

SECTION A — Brain Dump:
  <div class="section-header">🧠 Brain Dump</div>
  <p class="section-sub">Quick capture — no rules, just write.</p>
  <ul id="dumpList" class="todo-list"></ul>
  <div class="todo-input-row">
    <input id="dumpInput" placeholder="Capture a thought…">
    <button id="dumpAdd">Add</button>
  </div>

Each Brain Dump item:
  <li class="todo-item" data-id="{id}">
    <input type="checkbox" class="todo-check">
    <span class="todo-text">{text}</span>
    <span class="push-count">{pushCount > 0 ? 'pushed '+pushCount+'×' : ''}</span>
    <button class="todo-push" title="Push to tomorrow">→</button>
    <button class="todo-delete">×</button>
  </li>

todo-item: flex row, align-items center, gap 8px, padding 10px 12px, 
background var(--bg-card), border 1px solid var(--border), 
border-radius 10px, margin-bottom 6px.

push-count: 10px mono, color var(--accent-yellow), opacity 0.8. 
Hidden when pushCount = 0.

Carried-over items from previous days render in a "↩ Carried Over" 
section at the very top of the Brain Dump list, with background 
rgba(242,192,99,0.07) and a left border 3px solid var(--accent-yellow).

todo-text: inline-editable (same contentEditable pattern).

Checked items: opacity 0.5, line-through.

Push to tomorrow: move item to tomorrow's Brain Dump list, 
increment item.pushCount, mark item.pushedFrom = today's date. 
Remove from today's list. On the next day the item appears under 
"↩ Carried Over".

SECTION B — Eisenhower Matrix:
  <div class="section-header">🎯 Eisenhower Matrix</div>
  <div class="matrix-grid" id="matrixGrid">
    <div class="matrix-cell" data-quadrant="do-first">
      <div class="matrix-label urgent important">✅ Urgent + Important</div>
      <ul class="matrix-list" id="q-do-first"></ul>
      <div class="matrix-input-row">
        <input placeholder="Add task…">
        <button>Add</button>
      </div>
    </div>
    <div class="matrix-cell" data-quadrant="schedule">
      <div class="matrix-label not-urgent important">📅 Not Urgent + Important</div>
      <ul class="matrix-list" id="q-schedule"></ul>
      <div class="matrix-input-row">…</div>
    </div>
    <div class="matrix-cell" data-quadrant="delegate">
      <div class="matrix-label urgent not-important">📤 Urgent + Not Important</div>
      <ul class="matrix-list" id="q-delegate"></ul>
      <div class="matrix-input-row">…</div>
    </div>
    <div class="matrix-cell" data-quadrant="eliminate">
      <div class="matrix-label not-urgent not-important">🗑️ Not Urgent + Not Important</div>
      <ul class="matrix-list" id="q-eliminate"></ul>
      <div class="matrix-input-row">…</div>
    </div>
  </div>

matrix-grid: display grid, grid-template-columns 1fr 1fr, gap 10px. 
On screens ≤ 600px: grid-template-columns 1fr.

matrix-cell: background var(--bg-card), border 1px solid var(--border), 
border-radius 14px, padding 14px.

matrix-label colors:
  urgent + important:     color var(--accent-red),    font-weight 700
  not-urgent + important: color var(--accent-blue),   font-weight 700
  urgent + not-important: color var(--accent-yellow), font-weight 700
  not-urgent + not-important: color var(--text-tertiary), font-weight 700

Each matrix task item has same structure as Brain Dump item — checkbox, 
text, push-count, push button, delete button.

Tasks can be dragged between quadrants and dragged from Brain Dump into 
any quadrant. When dragged from Brain Dump into a quadrant, item is 
removed from Brain Dump and added to that quadrant, retaining pushCount.

Pushed matrix tasks carry over to the same quadrant the next day under 
a "↩ Carried Over" section in that quadrant.

---

Tab 5 — Quotes

Structure inside #tab-quotes:

RANDOM QUOTE DISPLAY:
  <div class="quote-display" id="quoteDisplay">
    <div class="quote-text" id="quoteText">Add a quote to get started.</div>
    <div class="quote-author" id="quoteAuthor"></div>
  </div>
  <button class="random-btn" id="randomQuoteBtn">✨ Random Quote</button>

quote-display: background var(--bg-card), border 1px solid var(--border), 
border-radius 16px, padding 24px, text-align center, margin-bottom 16px.
quote-text: font-size 17px, font-style italic, color var(--text-primary), 
line-height 1.6, margin-bottom 8px.
quote-author: font-size 12px, mono, color var(--text-tertiary).
random-btn: full-width, padding 10px, border-radius 10px, 
background rgba(255,255,255,0.06), border 1px solid var(--border), 
color var(--text-primary), font-size 13px, font-weight 600, 
cursor pointer, margin-bottom 20px. 
Hover: background rgba(255,255,255,0.10).

Random button picks one quote at random from saved quotes and animates 
it in (opacity 0 → 1, translateY 8px → 0, 0.3s ease). If no quotes 
saved, show "No quotes yet — add one below."

ADD QUOTE FORM:
  <div class="add-quote-form">
    <input id="quoteInput" placeholder="Type a quote…">
    <input id="authorInput" placeholder="Author (optional)">
    <button id="addQuoteBtn">Save Quote</button>
  </div>

SAVED QUOTES LIST:
  <div id="quotesList"></div>

Each saved quote:
  <div class="quote-item">
    <div class="qi-text">"{text}"</div>
    <div class="qi-author">{author or ''}</div>
    <button class="qi-delete">×</button>
  </div>

quote-item: background var(--bg-card), border 1px solid var(--border), 
border-radius 12px, padding 12px 14px, margin-bottom 8px, flex row, 
gap 10px, align-items flex-start.
qi-text: flex 1, font-size 13px, italic, color var(--text-primary).
qi-author: font-size 11px, mono, color var(--text-tertiary).
qi-delete: color var(--text-tertiary), hover color var(--accent-red).

---

Data & sync

At the very top of the <script> block declare:

  const SUPABASE_URL = '';
  const SUPABASE_ANON_KEY = '';
  const SUPABASE_TABLE = 'timeblox_state';
  const SHARED_KEY = 'shared-timeblox-v1';

If both SUPABASE_URL and SUPABASE_ANON_KEY are non-empty, load the 
Supabase JS client from CDN:
  https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2/dist/umd/supabase.min.js

On app load:
  1. Paint the UI instantly from localStorage (key 'timeblox_local_cache').
  2. If Supabase is configured, fetch the row where key = SHARED_KEY 
     from SUPABASE_TABLE. On success, merge remote data into app state, 
     update localStorage cache, re-render.
  3. If Supabase fetch fails, silently stay on localStorage data.

On every save (any block add/edit/delete, any to-do change, any quote 
change):
  1. Update localStorage immediately.
  2. If Supabase is configured, upsert { key: SHARED_KEY, data: fullState, 
     updated_at: new Date().toISOString() } to SUPABASE_TABLE.
  3. If upsert fails, stay silent — do not block or error the UI.

App state shape (one object, serialized to JSON):
  {
    blocks: { 'YYYY-MM-DD': [ ...blockObjects ] },
    templates: [ ...templateObjects ],
    routineTemplates: { morning: {...}, night: {...} },
    todos: { 'YYYY-MM-DD': { dump: [...], matrix: { 'do-first': [...], 
             'schedule': [...], 'delegate': [...], 'eliminate': [...] } } },
    quotes: [ { id, text, author } ]
  }

Also generate a separate file called supabase-setup.sql containing:
  CREATE TABLE IF NOT EXISTS timeblox_state (
    key TEXT PRIMARY KEY,
    data JSONB NOT NULL,
    updated_at TIMESTAMPTZ DEFAULT NOW()
  );
  ALTER TABLE timeblox_state DISABLE ROW LEVEL SECURITY;

---

Logic & state

Helper functions:
  getDateStr(date) → 'YYYY-MM-DD' string.
  getTodayStr() → today's date string.
  getTomorrowStr() → tomorrow's date string.
  getWeekDays(dateStr) → array of 7 date strings Mon–Sun for 
    the week containing dateStr.
  getMonthWeeks(year, month) → array of weeks (each an array of 
    7 date strings) for the month grid, including leading/trailing days 
    to complete Mon–Sun rows. No isolated single-day weeks.

Active state:
  let activeTab = 'calendar'
  let calView = 'day'
  let calDate = getTodayStr()  // currently selected date

All renders read from a central state object. Every mutation goes 
through a save(newState) function that:
  1. Updates the in-memory state object.
  2. Writes to localStorage.
  3. Fires a Supabase upsert (non-blocking).
  4. Calls the relevant render functions.

renderCalendar() — re-renders blocksArea and calLabel based on 
calView and calDate.
renderFocusProgress(dateStr) — counts Focus Blocks for dateStr, 
updates bar and text.
renderStats() — rebuilds the Stats tab cards and charts.
renderTemplates() — rebuilds the Templates tab list.
renderTodo(dateStr) — rebuilds To-Do tab for dateStr.
renderQuotes() — rebuilds Quotes tab.

Animations:
  Expand/collapse block body: max-height 0 → max-height 1000px, 
  overflow hidden, transition 0.3s cubic-bezier(0.22,1,0.36,1).
  Checklist item completion: opacity 1 → 0.5, transition 0.2s.
  Quote random display: opacity 0, translateY 8px → opacity 1, 
  translateY 0, transition 0.3s ease.
  Drag-and-drop: opacity 0.5 on dragged item.

---

Acceptance checklist

The file runs from a file:// URL or VS Code Live Server with no 
console errors.

App title "TimeBlox" appears at the top with gradient text.

Five tabs render and switch instantly: Calendar, Stats, Templates, 
To-Do, Quotes.

Day view shows blocks for the selected day. Adding a block opens the 
edit modal. Block renders with color-coded left border, title, total 
time, expand/collapse, complete button, edit button, delete button.

Segments inside a block auto-sum to block total time. Adding a segment 
updates the total immediately.

Checklist items inside a block are editable and deletable.

Mood emoji selector appears after a block is marked complete.

Morning Routine and Night Routine blocks show "Save as Default Routine" 
in the edit modal. New Morning/Night blocks auto-populate from the 
saved template.

Focus progress bar in Day view recalculates instantly on any change.

Week view shows Mon–Sun columns with block counts. Month view shows a 
full calendar grid with colored dots. Cross-month weeks render 
correctly with no isolated single-day days.

Stats tab shows accurate weekly and monthly Focus Block completion 
counts and bar charts.

Templates tab lists saved templates with name, duration, segment count, 
category. Import, rename, delete, search, and filter all work.

To-Do tab Brain Dump allows fast capture. Eisenhower Matrix shows 4 
quadrants. Tasks can be dragged between quadrants and in from Brain Dump.

Push to Tomorrow moves a task to the next day, increments push counter, 
and shows "pushed N×". Carried-over tasks appear under "↩ Carried Over" 
with yellow left border.

Quotes tab saves quotes with optional author, shows them in a list, 
random button surfaces one with a fade-in animation.

All data persists after page refresh via localStorage.

If SUPABASE_URL and SUPABASE_ANON_KEY are filled in, data syncs 
across every device, browser, profile, and incognito session.

The file is entirely self-contained — one index.html file with no 
external dependencies except the optional Supabase CDN.
