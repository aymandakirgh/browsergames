# Browser Games

Two self-contained first-person 3D games. Single HTML file each, Three.js r128 from CDN, zero assets — all geometry, textures and audio are generated procedurally at runtime. Open the file in a browser and play.

## HOLLOW SIGNAL — `horror.html`

First-person horror in a procedurally generated maze.

- Collect 5 pages, then escape through the green door
- Flashlight with battery drain + battery pickups; light helps you see but lets the entity see you from 3x farther
- Battery warning: a chime and HUD hint fire once when the battery first drops below 25% (re-arms after your next pickup), so a dying flashlight doesn't catch you by surprise mid-hunt
- Phasmophobia-inspired hunt cycles: the entity roams, then periodically hunts. It tracks your **last known position** — sprint away (it's slower than you) or go dark, stand still and let it lose you
- Sprint drains stamina, and emptying it leaves you **out of breath** (audible gasp) — locked to a walk until it recovers past a third. Don't burn it all before a hunt starts
- **Noise matters.** Your footsteps carry: sprinting is audible ~14m away, walking ~7m, crouching and standing still are silent. Noise inside that radius puts the entity into a new **investigate** state — it walks to where the sound came from (2.4 m/s, faster than roaming, well under a hunt) and looks around for ~7-11s before giving up. Investigating never grabs you; it just parks the thing near you until the next mistake starts a real hunt. This is what finally makes the crouch speed penalty worth paying when nothing is chasing you: crouching is the only way to move without leaving a trail
- **Crouch (hold C)** to sneak: barely a crawl and you can't sprint from it, but it cuts the entity's spotting range to 45% (6.8m lit / 2.3m dark instead of 15m / 5m) and softens your footsteps to a scuff. Crouching in the dark is the quietest you can be — the cost is that the hunt clock keeps running while you crawl
- Sanity system: drains in darkness and near the entity; low sanity = more frequent hunts, whispers, unsteady camera. Recovers slowly in a safe zone (flashlight on, entity far, no active hunt) — rewards careful play, but costs battery so it's never a free reset. The bar tints green while recovering.
- Procedural audio: drone, heartbeat by proximity, growls, breathing radar, distant screams, knocking, jumpscare. Growls, the entity's footsteps and the breathing radar are stereo-panned to its bearing — you can hear which side it's on
- The flashlight's proximity and hunt flicker run on a fixed ~14Hz clock, so the strobe looks the same on a 60Hz and a 144Hz display
- **Distraction throwable:** three pebbles per run (G). A thrown pebble lands 2-4 cells down whichever cardinal corridor you're facing (stopping early at a wall) and, if the entity is close enough to hear it land, feeds that cell into the same `investigate()` gear your footsteps trigger — it goes to check the noise instead of you. Wasted if the entity's too far away to hear it, or already mid-hunt. The pebble counter tints red once you're out.
- **Settings menu** (pause → SETTINGS): mouse sensitivity (0.4x-2x) and master volume sliders. Plain in-memory state, resets on reload by design — same as the rest of the game's state
- The heartbeat cue now lands with a matching visual thump on the fear vignette, so mounting dread is felt as well as heard instead of relying on audio alone
- The death jumpscare now picks one of three faces (eye colour + mouth shape) at random, so repeated retries — which this game expects a lot of — don't stare back with the identical still every time

> **Difficulty note:** the entity can now actually catch you. Proximity was measured in 3D against a camera sitting 1.65m above an entity standing on the floor, so the grab distance could never fall below 1.65 — while the kill threshold was 1.55. The game was unloseable. Distances are now measured on the floor plane and the grab is re-tested after the entity moves.

Controls: WASD · SHIFT sprint · **C crouch** · F flashlight · **G throw pebble** · mouse look · ESC pause

> Crouch is bound to **C**, not Ctrl: Ctrl+W closes the tab in every major browser, so holding Ctrl to sneak forward would end the run. Ctrl still works as an undocumented alias for anyone whose muscle memory demands it.

## CORDITE — `shooter.html`

Wave-based arena FPS.

- Hitscan rifle: spread bloom, recoil, reload, tracers, muzzle flash, headshots deal 3x
- Aim down sights (hold RMB): tighter spread, zoomed FOV and steadier look for ranged precision — at the cost of movement speed; sprinting cancels it
- Enemy frames steer around cover, strafe, fire dodgeable projectiles, melee up close; drop health/ammo
- Platforming: jumpable crates, staircases and sniper decks — enemies can't climb
- Escalating waves, score, screen shake, damage feedback, synthesized audio
- Low-integrity warning: a pulsing red vignette and a slow thump kick in below 35% HP so you feel the danger without checking the bar
- Between waves a live countdown shows when the next wave drops, tinting amber in its final 3 seconds; enemy shots flash a muzzle spark at the firing frame so you can read where fire is coming from. Clearing a wave now plays its own rising fanfare, distinct from the wave-start cue
- Melee contact now refreshes the integrity bar and plays a throttled hurt cue — previously the bar sat frozen and the damage was silent, so being clawed down read as a bug
- **Pooled enemies:** an enemy frame is a 6-mesh group whose geometry was already shared, but every spawn still built a fresh body material plus five `THREE.Mesh` wrappers and every kill disposed them — a 12-enemy wave churned ~72 objects. Rigs now return to a free list and are re-dressed on the next spawn (body materials stay with their rig, since each enemy flashes its own emissive on hit). Steady-state waves allocate no enemy meshes at all
- **Pooled effects:** sparks, debris and tracers are recycled from a free list instead of allocating a fresh geometry + material per particle and disposing them a second later. A single kill used to churn ~23 GPU buffer create/destroy pairs; after warm-up the effect system now allocates nothing per frame
- **Pooled projectiles & pickups:** the last two unpooled spawns — enemy shots and health/ammo drops — now come from free lists with shared geometry and cached-by-type materials, same as the enemy rigs and particle effects above
- **Ammo feedback:** the ammo counter tints red at 5 rounds or fewer, and holding fire with an empty mag *and* empty reserve now plays a throttled dry-click instead of firing silently forever — `startReload()` no-ops once reserve is also empty, and that path never touched the fire-rate cooldown, so it used to retrigger every frame with no cue
- **Settings menu** (pause → SETTINGS): mouse sensitivity (0.4x-2x, also scales the aim-down-sights multiplier) and master volume sliders
- **Kill combos:** consecutive kills inside a 2.2s window stack a score bonus (+20 per kill in the streak) and a banner — DOUBLE/TRIPLE/QUAD KILL, RAMPAGE at 5+ — with its own chime distinct from a normal kill. Rewards clearing a cluster fast over picking enemies off one at a time; any gap longer than the window resets the streak

Controls: WASD · mouse aim · hold LMB fire · RMB aim down sights · R reload · SHIFT sprint · SPACE jump

Sprint works with either Shift key in both games.

## Roadmap

Larger ideas deferred to keep each change incremental and reversible:

- Mobile / touch controls (virtual stick + look drag)
- Hiding spots in the horror maze (lockers / alcoves that break line of sight entirely), now that crouch, the noise model and the pebble throw give stealth three mechanics to build on

---

Built with Claude. No build step, no dependencies beyond the Three.js CDN script.
