What It Is

Nullscape GUI is a client-side mod menu for the Roblox game Nullscape, built on the juanitahaxx UI Library (by samet, design by eskolzz). The script is loaded via an executor, downloads the UI library remotely, and renders an in-game window ("Nullscape GUI") with nine feature pages, global keybinds, and background automation loops. It also integrates with the game's remotes (gift magnet, upgrades, enemy notifications) and replicates server-trusted behaviors locally where possible.
The project folder contains the ported script (null_juanitahaxx.lua) and the original pre-port version (null.lua) kept for reference.
What It Does
On execution it verifies the place ID, guards against duplicate execution, loads the UI library, then builds its modules staged (one per frame) so nothing bursts at load time. Background listeners and the movement-safety guard come online first; pages stream in after. Saved configs are applied gradually rather than in one synchronous burst.

Features
Main — Gift Collection
- Auto-collect Normal / Golden gifts with tween pathing that avoids active tripmines and enemies
- Collection range slider (drives the game's gift magnet remote) + reset
- Live counters: Gifts, Golden Gifts, Passage Gifts, Tripmines (value + attribute synced)

Upgrades
- Add/Remove ~21 client-side upgrades (Adrenaline, Double Jump, Gift Magnet, Radar suites, wings, etc.) via firesignal spoofing of the upgrade remote

Enemy Management
- Disable / Break AI / Destroy all enemies, or client-side ones only
- Per-enemy auto-handling toggles (Auto Disable, Auto Break AI, Auto Destroy) for Bell, Mart, Husk, Springer, ICBM, Baby, Flesh, Telefragger, Celestial, Cadence, Voidbound Baby…
- "Add Enemy" spawner (Kolona, Operator, Voidbreaker, Scrapmaw, Husk) for next round
- Guardian bullet protection (repels/pushes player out of bullet radii)

Map Utilities
- Anti-Void: teleport-to-spawn, launch-up (adjustable power), or snap-to-closest-gift when falling into the void
- Visible Void, altar finder/teleporter/activator, tripmine protection bubbles, seamine & void-implosion disabling, fake-beacon destruction
- Tile modifier: auto-remove ice/flesh floor tiles
- Pylon (Bloom) selector + teleporter

Player
- WalkSpeed / JumpPower overrides with sliders, watchers, and stat restoration on toggle-off
- Visible hitbox, Razorbloom destroyer

Visuals
- Closest-gift tracer ESP, Medal ESP, Cadence instrument ESP (adornments + off-screen clamped tracers)
- FOV slider, velocity vector visualizer

Keybinds
13 actions bound by default (N/0/Eight/H/J/T/Y/Two/Q/LeftAlt/Home/Insert/End) each with individual enable toggles — collecting, medal grab, enemy wipes, pad/orb bringers, instant grapple, glider fly-hold, spawn/beacon teleports, cancel-collecting.

Debug / Safety
- Copy lobby code, kill character, rejoin, panic-unload (full teardown of connections, drawings, instances)
- Charge-noclip guard: silently re-enforces root.CanCollide / AutoRotate every frame (fixes the game's stuck charge state)
- Manual movement repair + full diagnostics dump (humanoid state, collision groups, network owner, ground probe)
- Place-ID check with native Roblox notification if executed in the wrong game

Objective
- Fix all bugs in the Roblox executor script null_juanitahaxx.lua (a juanitahaxx UI library port for the game Nullscape), including a persistent charge-noclip movement bug.
- Make script execution fully staged (no module loads all at once), including gradual config application.

Important Details
- Charge-noclip ROOT CAUSE (user-confirmed fixed): game leaves root.CanCollide=false + AutoRotate=false after charge; silent Heartbeat guard enforces both every frame when autoFixCharge=true (~:1965). Do NOT regress this.
- juanitahaxx Library (remote loadstring, audited): has zero physics code; BUT element constructors fire callbacks with defaults during build, AND Keybind constructors fire their action callbacks at build time (confirmed by user's log: teleport-to-beacon fired, 6× "No enemies available", 5× "Already collecting").
- Autoload: library replays shared juanitaaaaaaa/autoload.json synchronously in Window:Init(). Current design: capture into savedAutoload, blank file before Init, restore file after, never auto-replay; manual Debug button "Load Saved Config Now" → applyConfigStaged(savedAutoload) (one setting per frame).
- uiReady flag gates side-effectful callbacks (WS/JP sliders+toggles, FOV, VisibleHitbox, VisibleVoid, magSlider magnet fire). Set true in finalize runner after Window:Init().
- Stage system: stage(fn, early) declared at :191; early stages = gift listeners (:395), connections/loops incl. heartbeat guard (:1875–2190); 9 page stages build one per frame; finalize runner iterates { earlyStages, buildStages }.
- User wants autoload to WORK again but applied gradually/staged (not suppressed forever).

Work State
Completed
- All ~20 initial bug fixes (notif shadowing, pathBlocked Model crash, adornment recreation, gift-list dedupe, grapple pcall, anti-void spam throttle, Material.Air→Neon, tile nil-guard, altar guard, fresh tripmines, speed restore, subUpgrade signal, counter attribute bindings, notifyBindable helper, filter unification, WaitForChild refs, LeftAlt glider key, perf nits).
- Latent scoping hoists to file scope: antiVoidSelection, lp, lastAvNoGift, partsConnected, customPlaying, currentCustom, ew/ej/ws/jp, runLoop, savedAutoload, autoFixCharge, stopCustomMusic, restoreHumanoid, uiReady.
- Charge-noclip final fix (CanCollide/AutoRotate guard) — user confirmed working.
- Full staged execution: window shell inside Main stage, each Page created in its own stage, loops as early stage, runLoop nil-guarded in destroyGui.
- applyConfigStaged(configJson) defined (~:305): JSON-decodes, applies one flag per frame via Library.SetFlags[name] with Key/Color table handling.
- C:\Users\kee\Downloads\Nullscape-Framework\fixes.txt written (first-person summary of all fixes).
- Todo list created for current round (see Active).

Active
- Round in progress, NO edits made yet. Plan (todos):
1. Gate all 13 keybind callbacks behind uiReady/new applyingConfig flag (kills construction-time fires: teleports, collects, disableAll spam).
2. Add local applyingConfig = false near uiReady decl (~:183); set true/false around applyConfigStaged loop body.
3. Restore autoload functionality: in finalize runner after uiReady = true, call applyConfigStaged(savedAutoload) automatically (staged).
4. Syntax check.

Blocked
- (none)

Next Move
1. Insert local applyingConfig = false next to uiReady declaration (~:183 area, near savedAutoload/autoFixCharge block).
2. Edit all 13 keybind callbacks in the KEYBINDS stage — prepend if not uiReady or applyingConfig then return end (GliderBoost special-case: if not uiReady or applyingConfig then gliderBoost = false; return end). Known anchors (Flag lines unique): KB_CollectNormal & KB_CollectGolden are single-line callbacks (Callback = function() if canEzCollectNormal then ... end end) needing expansion; multi-line ones start with guards like if not canEzCollectMedal then return end, if canEzDisableAll then disableAll(false,false,false) end, if not canFullReset then return end, if not canBringPad then return end, if not canBringTria then return end, if not canInstaGrapple or not canPress then return end, Callback = function(holding)\n if not holding then gliderBoost = false; return end, if not canGoHome then return end, if not canGoBeacon then return end, KB_CancelTween single-line.
3. In applyConfigStaged: wrap the task.spawn loop with applyingConfig = true before / applyingConfig = false after (before the notif).
4. In finalize runner: after uiReady = true, add applyConfigStaged(savedAutoload) (auto staged re-apply).
5. Run node check.js "C:\Users\kee\Downloads\Nullscape-Framework\null_juanitahaxx.lua" from luacheck workdir; confirm SYNTAX OK; report to user.

Relevant Files
- C:\Users\kee\Downloads\Nullscape-Framework\null_juanitahaxx.lua — the only file being modified (~2270 lines).
- C:\Users\kee\Downloads\Nullscape-Framework\null.lua — pre-port original; reference only, intentionally NOT modified.
