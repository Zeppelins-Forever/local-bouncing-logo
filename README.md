# Bouncing logo

A DVD-style bouncing-logo screensaver. Logos come from a folder, settings from one optional file.

```
bouncing-logo/
├── index.html    the page (don't edit)
├── config.ini    optional settings and per-logo rules
├── README.md
└── logos/        every .svg / .png / .webp in here is used
```

## Run it

```sh
cd bouncing-logo
python3 -m http.server 8000
```

Open <http://localhost:8000/>. It needs a web server, so double-clicking `index.html` won't work.

## Add a logo

Drop a file into `logos/` and reload. It's used with all defaults.

- **Transparent background:** SVG, or PNG/WebP with transparency. Any size or margins: each image is cropped to its visible artwork.
- **File names:** name files `01-…`, `02-…` so their numbers stay fixed (see `logos` below).
- **Folder listing:** if your server can't list folders, add an empty `[full-file-name.svg]` section for each file in `config.ini`.

## Settings: config.ini or the URL

Every `[screen]` setting works in either place. The URL wins over the file.

```ini
[screen]
count = 4
frame = rounded, pill
```
is the same as `http://localhost:8000/?count=4&frame=rounded,pill`

| Setting | Default | What it does | Example |
|---|---|---|---|
| `count` | `1` | Logos on screen (1–40). Extra ones slide in from the edges. | `count = 5` |
| `logos` | `all` | Which files: numbers (alphabetical order) or words from the file name | `logos = 1-3` · `logos = 1,3` · `logos = white` |
| `frame` | `all` | Frames to show: `off` `rounded` `double` `brackets` `pill` | `frame = rounded, pill` |
| `recolor` | `auto` | `auto` recolors only logos without a frame. Or a list of `on` `match` `off`. | `recolor = match` |
| `same` | `no` | `yes` = every logo identical (file, frame, colour mode) | `same = yes` |
| `glow` | `1` | How strongly `rounded` and `pill` frames glow. `0` = no glow. | `glow = 2` |
| `size` | `0.5-1.15` | Random size range (1 = normal). One number = fixed. | `size = 0.8` |
| `shrink` | `0.3` | How much logos shrink as `count` rises. `0` = never. | `shrink = 0` |
| `fill` | `0.35` | Max share of the screen all logos together may cover | `fill = 0.5` |
| `speed` | `0.6-1.6` | Random cruising speed range (1 = normal). One number = fixed. | `speed = 1` |
| `speed-cap` | `off` | Hard top speed, even right after a hit. `on` = top of the `speed` range, or a number (× normal). | `speed-cap = on` · `speed-cap = 1.2` |
| `spin` | `off` | Logos spin when they hit each other. `on` = eases back upright, `free` = stays tilted. | `spin = on` |
| `gap` | `2.5` | Seconds between logos entering. `0` = all at once. | `gap = 0` |
| `collide` | `yes` | Logos bounce off each other | `collide = no` |
| `jump` | `70` | Minimum hue change per bounce (degrees, 0–120) | `jump = 120` |
| `messages` | `yes` | On-screen text for keyboard shortcuts | `messages = no` |
| `background` | `#111111` | Page colour. URL: `%23000000` (`#` must be written `%23`) | `background = #000000` |

## Per-logo rules (config.ini only)

A section named after a file sets what that logo is **allowed** to do. `[screen]` then chooses among what's allowed, so these rules are never broken, even by the URL or keyboard.

```ini
[01-ta-color.svg]              ; the .svg may be left off
frame   = rounded, double, brackets, pill
recolor = off
```

| Setting | Default | Values |
|---|---|---|
| `frame` | `all` | `off` `rounded` `double` `brackets` `pill` |
| `recolor` | `all` | `on` `match` `off` |
| `size` | `1` | Relative size (`0.8`), or a fixed artwork width (`240px`) |
| `weight` | `1` | How often it's picked vs. other logos. `0` = never. |

A list is complete: only the values you name are used. A logo with no section allows everything.

**Frames.** The frame always takes the changing colour.
- `off`: no frame. The logo collides using its own outline (a heptagon bounces as a heptagon). Hollow or concave shapes use their outer outline.
- `rounded`: rounded frame with a glow (called `glow` before; the old name still works).
- `double`: thick and thin double line.
- `brackets`: four corner brackets.
- `pill`: a capsule with a glow. Its rounded ends deflect logos at angles.

**Recolor modes.**
- `off`: the logo keeps its own colours.
- `on`: the logo is drawn in one changing colour. With a frame, it takes the opposite colour to the frame.
- `match`: the logo takes the frame's colour. Without a frame this is the same as `on`.

### Recipes

```ini
; Brand colours are untouchable; frames are fine
[brand.svg]
recolor = off

; Shown exactly as supplied: no frame, own colours
[mascot.svg]
frame   = off
recolor = off

; Only ever a solid changing colour, no frame, always 300px wide
[icon.svg]
frame   = off
recolor = on
size    = 300px

; Rarely shown
[seasonal.svg]
weight = 0.2
```

Screen-wide examples:

| Want | config.ini `[screen]` | URL |
|---|---|---|
| Classic: one logo, no frame, changing colour | `frame = off` | `?frame=off` |
| Six logos, all pill frames, logo matches frame | `count = 6` `frame = pill` `recolor = match` | `?count=6&frame=pill&recolor=match` |
| Three of the same, picked from files 1–2 | `count = 3` `logos = 1-2` `same = yes` | `?count=3&logos=1-2&same=yes` |
| Lots of small logos with no collisions | `count = 25` `collide = no` | `?count=25&collide=no` |
| Strong glow, black background | `glow = 2` `background = #000000` | `?glow=2&background=%23000000` |
| Busy screen that spins but never gets frantic | `count = 8` `spin = on` `speed-cap = on` | `?count=8&spin=on&speed-cap=on` |

## Keyboard

| Key | Does |
|---|---|
| `+` / `-` | Add / remove a logo |
| `1`–`5` | Frame for all: none, rounded, double, brackets, pill |
| `r` | Cycle recolor: auto → on → match → off |
| `s` | Toggle "all the same" |
| `0` | Back to config.ini / URL |

Keyboard changes last until reload, and skip any logo whose rules forbid them.

## How it behaves

- **Sizes:** logos are sized by area, so wide, square and tall logos look equally big. Sizes adapt to the screen and to `count`, and stay within `fill`.
- **Collisions:** logos exchange momentum. Bigger logos push smaller ones harder. A logo knocked faster or slower (between 0.5× and 2× its own speed) eases back to its own speed over a few seconds. `speed-cap` sets an absolute ceiling on top of that.
- **Spin** (when on): off-centre and glancing hits between logos set them spinning, up to about one turn a second, and the spin fades over a few seconds. Walls never add spin. A spinning logo's rotated outline is what touches walls and other logos.
- **Direction:** no logo travels within 10° of straight across or straight up and down.
- **Colour:** every bounce (wall or logo) changes a logo's colour by at least `jump` degrees.
- **Entering:** new logos enter at a free spot and pass through others until they're fully on screen and clear.
- **Performance:** with `spin = off`, spin costs nothing. Turning it on roughly doubles the physics work, which is still small. Drawing usually costs more than physics, especially rotating glows. On slower hardware (e.g. a Pi 3), keep `spin = off`, and try `glow = 0` if it stutters.
- **Problems:** they show on screen (no logos, nothing allowed, opened without a server). Config typos are reported in the browser console (F12).
