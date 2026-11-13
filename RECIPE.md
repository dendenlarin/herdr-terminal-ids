# Terminal IDs

Herdr's agents sidebar and pane header with each terminal's address (`t51`, or `vps/t51` on another machine) shown and copyable: click it to copy and point another agent at that terminal. One mod, two layout slots chosen by `props.role`.

![Terminal IDs](preview.png)

## What you see

- **Agents sidebar** (`role: "agents"`), `chrome` background, 1px `border` on top, 13px text:
  - **Header**, 41px, 16px side padding: `Agents` in 11.5px medium `dim`; on the right the sort icon (14px, `dim`, `foreground` on hover): `clock.svg` for Priority, `layers.svg` for Grouped by space.
  - **Group headers**, 30px with 8px top padding, 11px medium: Priority sections `Needs you` (`blocked`), `Done` (`done`), `Working` (`working`), `Idle` (`dim`) with a member count; or space names when grouped; `Pinned` first when any agent is pinned. Folded: the count, the most urgent status mark (grouped only) and `chevron-right.svg`.
  - **Agent rows**, 44px, 8px side padding, radius 8, gap 9; `selected` background for the focused agent, `hover` on hover. Left: the program icon, 16px. Two lines:
    - top: the space name (`foreground`, `muted` when idle and not focused, truncated) and the terminal address in 11px `accent`, at most 110px, truncated; `foreground` on hover;
    - bottom, 11px `dim`: the tab's title (a named tab or the agent's own title, never a bare tab number unless two rows would read the same), else the program name; a fold chevron with the sub-agent count when it has sub-agents; on hover a pin / unpin button (13px `pin.svg` / `pin-off.svg`), otherwise the time since its status last changed (`<1m`, `5m`, `2h`, `3d`); the status mark.
  - **Sub-agent rows**, 40px, 39px left padding: tree lines (9px wide, 1.5px `input` borders, `├` / `└` with a rounded corner), the program icon 14px, the name (`foreground` if shown, else `muted`) over its address in the same style.
  - **Status marks**: working — 12px `spinner.svg` in `working`, spinning every 900ms; blocked — an 18px `Needs you` pill, 10.5px semibold `blocked` on `blocked` at 15% (a 14px `!` box inside Priority sections, where the header already says it); done — a 7px `done` dot; idle — a 7px ring, 1.5px `idle` (or `dim`).
  - **Rail**, when the sidebar is under 160px wide: 26px rows with the position number (`muted`, right-aligned) and the compact status mark; a `»` button at the bottom.
  - **Empty**: `No agent status yet. Agents show up here when they report their status. Older versions and agents inside tmux may not.` in 12px `dim`.
  - **Sort popup**, 200×140, `popover` background: `Sort agents` (11px `dim`), then `Priority` and `Grouped by space` rows (31px, radius 7, 14px icon in `muted`, `check.svg` on the chosen one).
- **Pane header** (`role: "pane"`), the layout's header height (30px in the snippet), `background` with a 1px `border` at the bottom, 10px side padding, gap 8, 12px text: the program icon 14px `dim`; the agent's name or the terminal title, medium `foreground`, at most 260px; the folder (`~` for home) in `dim`, truncated; the address in 11px `muted` (6px/2px padding, radius 4, `hover` background); then on the right the status (working spinner, `Needs you` pill or `done` dot as above) and, while hovered, 14px `dim` buttons: split right, split down, zoom / unzoom, close.

## How it behaves

- **Address click** (both views): copies the full address (remote prefix included, even when truncated on screen) with `cx.write_to_clipboard`, shows `notify({ title: "Address copied", body: address, timeout: 1600 })`, and stops propagation, so the row isn't selected or focused.
- **Agent row click**: `pane.focus` on its pane. Clicking the program icon folds or unfolds its sub-agents. Sub-agent row click: `terminal.peek`. Pin button: `terminal.pin` toggled.
- **Priority sort**: by the most urgent of the agent and its live sub-agents (blocked > done > working > idle), then most recent status change, then newest; pinned agents first; the order goes to `setAgentOrder`. A done agent you look at turns idle but stays in Done until you leave it.
- **Status age**: when each agent's status last changed (done and idle count as one), kept in the mod's `localStorage` key `changed`; entries for agents gone over 30 days are dropped; a damaged value starts afresh. The list redraws every 30s for the ages.
- **Sort and rail**: the sort is shared `agents.sort`. `»` on the rail sets `sidebar.collapsed` to false and `layout.resize`s `sidebar` back to shared `sidebar.width` (268px by default); collapsing the other way saves the width there and resizes to 76px on macOS, 44px elsewhere.
- **Pane header click**: `pane.focus`. Buttons: `pane.split {direction: "right" | "down"}`, `pane.zoom`, `pane.close`. No address when the pane has no terminal.

## Built from

- **Mod** `mods/terminal-ids/`: `mod.json` `{ "place": "none", "title": "Terminal IDs" }`; `main.js` (one `View` that builds `agents.js` when `props().role === "agents"`, else `pane.js`, and delegates `init` / `render`); `agents.js` and `pane.js`, derived from Herdr 0.1.13's shipped `agents` and `pane` mods (Apache-2.0, see NOTICE); `split-right.svg`, `split-down.svg`, `zoom.svg`, `unzoom.svg`, `close.svg` (Lucide).
- **Layout**: the agents block's content `{ "mod": "terminal-ids", "props": { "role": "agents" } }`; the active tab block's `content.pane.header` `{ "content": { "mod": "terminal-ids", "props": { "role": "pane" } }, "reveal": "always", "height": 30, "position": "top" }`, optionally `content.pane.lone: true`. The sort popup is `openOverlay({ mod: "terminal-ids", kind: "popup", width: 200, height: 140, props: { role: "agents", sortMenu: true } })`.
- **herdr**: `agents`, `terminals`, `panes`, `focused`, `spaces`, `layout`, `window`, `theme`, `home`, `props`, `shared` / `setShared` (`agents.sort`, `sidebar.width`, `sidebar.collapsed`), `setAgentOrder`, `openOverlay` / `closeOverlay`, `notify`, `run` (`pane.focus`, `pane.split`, `pane.zoom`, `pane.close`, `terminal.peek`, `terminal.pin`, `layout.resize`). Art from `mods/workspace-chrome/`: `spinner.svg`, `clock.svg`, `layers.svg`, `check.svg`, `chevron-down.svg`, `chevron-right.svg`, `pin.svg`, `pin-off.svg`.
- No network, programs or files outside the mod.

## Watch out

- Write `mod.json` (`"place": "none"`) before `main.js`: a new mod folder without it is docked into the shown layout on first sight.
- Herdr's shipped layouts are read-only: `layout.clone` one first (`herdr-app do layout.clone '{"name":"main","as":"main-terminal-ids"}'`) and edit the copy, keeping unrelated fields and panels.
- Older installs used `agents-address` and `pane-address`: point those references to `terminal-ids` with `role: "agents"` / `role: "pane"`, and remove the old folders only when no layout uses them.
- `localStorage` is per mod: status ages start fresh after moving from Herdr's `agents` mod or the old ones.
- The views follow Herdr 0.1.13's `agents` and `pane` mods; compare with the shipped ones when Herdr changes them.

## Done when

- `herdr-app check ~/.config/herdr-app/mods/terminal-ids` passes and `herdr-app status` lists no errors.
- The sidebar shows each agent's space and address on the first line; the pane header shows the address beside the folder.
- Clicking an address copies it in full and shows `Address copied` without focusing another agent.
- Sorting, pins, sub-agent folding and the rail work as in Herdr's own agents list.
