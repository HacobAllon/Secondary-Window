# Secondary Window

A floating popout for EuroScope that draws TopSky and Groundradar style polygon maps and
altitude filtered traffic in a separate, always on top window.

---

## Install

1. Download the latest `Secondary Window.zip` from the **Releases**
    Inside:
   - `SecondaryWindow.dll`
   - `SecondaryWindowMap.txt`     ← put your map definitions here
   - `SecondaryWindowSettings.txt` ← put your settings here
2. Drop all three files into the same folder.
3. In EuroScope: **Other Set → Plug-ins → Load**, pick `SecondaryWindow.dll`.
4. A small floating window labeled *Secondary Window* appears above EuroScope.

---

## File layout

| File                          | Purpose                                                                                   |
| ----------------------------- | ----------------------------------------------------------------------------------------- |
| `SecondaryWindowMap.txt`      | Map definitions: colors, polygons, lines, polylines, labels. Altitude filter for traffic. |
| `SecondaryWindowSettings.txt` | Visual tuning: background, dots, fonts, tag format.                                       |

### Commands

```
.sw reload
```
`.sw load <full-path>` / `.sw settings <full-path>` to use a file
elsewhere.

---

## Map file (`SecondaryWindowMap.txt`)

Plain text. Each line is a directive: `KEY:value...`. 
Coordinates use TopSky format: `hemisphere + DDD.MM.SS.fff`,
e.g. `N014.30.00.000` or `E121.05.29.662`.

### Colors

Declare named colors once near the top, then reference by name:

```
COLORDEF:Black:0:0:0
```

### Map blocks

Every map starts with `MAP:`. Everything that follows belongs to that map
until the next `MAP:` directive.

```
MAP:RPLL:A                 // required — display name
FOLDER:CTR                 // optional — grouping label
ACTIVE:1                   // optional — 0 hides on load, 1 shows (default)
COLOR:Blue                 // sets the active color
```

`COLOR:` can be issued multiple times inside a map to change color
between shapes.

### Shapes

Three closed/open variants, all the syntax Ground Radar / TopSky use:

| Directive | Behavior |
|---|---|
| `COORDTYPE:OTHER:REGION` + `COORD:` lines | **Filled** closed area |
| `COORDTYPE:OTHER:POLYGON` + `COORD:` lines | **Outline-only** closed shape (last vertex connects to first) |
| `COORDTYPE:OTHER:POLYLINE` + `COORD:` lines | Open line strip, no closing segment |
| ------------------------------------------- | ------------------------------------------------------------- |
| `COORDTYPE:OTHER:REGION` + `COORD:` lines   | **Filled** closed area                                        |
| `COORDTYPE:OTHER:POLYGON` + `COORD:` lines  | **Outline-only** closed shape (last vertex connects to first) |
| `COORDTYPE:OTHER:POLYLINE` + `COORD:` lines | Open line strip, no closing segment                           |

Each shape ends at the next `COORDTYPE:`, `MAP:`, or end of file.

```
// Filled area:
COORDTYPE:OTHER:REGION
COORD:N014.30.00.000:E120.30.00.000
COORD:N014.30.00.000:E121.30.00.000
COORD:N015.30.00.000:E121.30.00.000
COORD:N015.30.00.000:E120.30.00.000

// no fill:
COORDTYPE:OTHER:POLYGON
COORD:N014.37.36.529:E120.59.29.134
COORD:N014.37.35.053:E120.59.48.519
... more coords ...

// Approach arc with no closing line:
COORDTYPE:OTHER:POLYLINE
COORD:N014.28.29.254:E121.05.29.662
COORD:N014.28.24.532:E121.05.27.335
... more coords ...
```

Other:

```
POLYGON
COORD:N..:E..
COORD:N..:E..
ENDPOLYGON
```

### Single line segments

For one off straight lines without setting up a polyline block:

```
LINE:N014.30.00.000:E120.30.00.000:N014.30.00.000:E121.30.00.000
//   ^from-lat       ^from-lon       ^to-lat         ^to-lon
```

### Text labels

Drops a label at a geographic coordinate in the current color:

```
TEXT:N014.34.35.000:E120.54.02.800:2000ft
TEXT:N014.42.05.500:E121.06.55.600:2500ft
TEXT:N014.35.50.679:E120.59.24.237:5500ft
```

#### Per-section label size

Use `TEXT_SIZE:` to set the font size (in pixels) for every `TEXT:` and
inline ESE label that follows. It sticks until the next `TEXT_SIZE:`
directive. Use `TEXT_SIZE:0` to reset to the global default
(`LABEL_FONT_SIZE` in settings).

```
TEXT_SIZE:18
TEXT:N014.30.00.000:E121.00.00.000:RPLL TWR

TEXT_SIZE:10
TEXT:N014.30.45.000:E121.00.30.000:Bay 18
TEXT:N014.30.46.000:E121.00.32.000:Bay 19

TEXT_SIZE:0     // back to default
```

The font face and bold are configured globally in
`SecondaryWindowSettings.txt` (see `LABEL_FONT_FACE` / `LABEL_FONT_BOLD`).

### Importing labels from `.ese` files

EuroScope `.ese` label format is `lat:lon:category:label`, e.g.

```
N014.30.53.908:E121.00.32.721:RPLL BAYS-GroundLayout:61
N014.27.15.000:E120.55.01.000:RPLL HELI RPT_PTS:Binakayan
N014.31.10.449:E121.00.44.997:RPLL BAYS-GroundLayout:115
```

Relative paths resolve against the map file's folder. Absolute paths
work too. Add as many `INCLUDE_ESE:` lines as you want.


```
.sw ese "C:\.ese"
```

it will skips lines starting with `;` (ESE comments)
or `[` (section headers like `[FREETEXT]`), so you can point it at a full
`.ese` file with many sections and it'll just pick out the label formated
lines. Imported labels render in white by default.

**Text Size** 
Put `TEXT_SIZE:#` before the coords to apply that pixel size to everything that follows.

```
TEXT_SIZE:8
N014.30.53.908:E121.00.32.721:RPLL BAYS-GroundLayout:61
N014.31.10.449:E121.00.44.997:RPLL BAYS-GroundLayout:115

TEXT_SIZE:0      // reset to default
```

### Traffic altitude filter

```
FILTER_ALT_MIN:0
FILTER_ALT_MAX:24000
```

---

## Settings file (`SecondaryWindowSettings.txt`)

Same `KEY:value` syntax. Any key you omit falls back to the built-in default.

### Background

```
BACKGROUND_COLOR:0:0:0       // R:G:B of the map area
```

### Window

All R:G:B. Omit any line to keep the built-in default.

```
BORDER_COLOR:40:40:40          // x outline around the window
TITLE_BAR_COLOR:80:80:80       // strip behind the title
TITLE_TEXT_COLOR:220:220:220   // "Secondary Window" text
CLOSE_ICON_COLOR:220:220:220   // X icon in the top-right
RESIZE_GRIP_BG:60:60:60        // bottom-right grip square fill
RESIZE_GRIP_COLOR:140:140:140  // diagonal hatch on the grip
```

### Map rendering

```
LINE_WIDTH:1                   // pixels, applies to lines, outlines, polylines
FILL_POLYGONS:true             // true = filled regions are filled; false = outline only
```

### Label font (TEXT + inline ESE labels)

Per-section overrides via `TEXT_SIZE:n` in the map file take precedence.

```
LABEL_FONT_FACE:Consolas
LABEL_FONT_SIZE:12             // pixel height
LABEL_FONT_BOLD:false
```

### Traffic dots

```
TRAFFIC_DOT_COLOR:255:255:0    // dot color
TRAFFIC_DOT_RADIUS:3           // px (half-width of the square dot)
```

### Tags (labels next to each dot)

```
SHOW_TAGS:true
TAG_COLOR:255:255:0
TAG_FONT_FACE:Consolas
TAG_FONT_SIZE:12               // pixel height
TAG_FONT_BOLD:false
TAG_OFFSET_X:5                 // px from dot center (positive = right)
TAG_OFFSET_Y:-3                // px from dot center (negative = up)
TAG_CLICK_WIDTH:80             // clickable rect (currently unused for clicks)
TAG_CLICK_HEIGHT:14
```

### Tag content

The text shown next to each aircraft dot is built from one or more
`TAG_LINE:` entries, in order. Each is a format string with
`{placeholder}` tokens. **You can add as many `TAG_LINE:` entries as you
want — each becomes a new line** in the tag.

```
TAG_LINE:{callsign}
TAG_LINE:{type} {alt} {cfl}
TAG_LINE:{gs}kt {hdg}
TAG_LINE:{scratch}
```

That gives every tag four lines:
```
PAL123
A320 19000/24000
200kt 045
BAY105
```

If no `TAG_LINE:` is declared, the tag falls back to a single line showing
just `{callsign}`.

#### Placeholder reference

Position / radar data (always available):

| Placeholder  | What it shows                                                         |
| ------------ | --------------------------------------------------------------------- |
| `{callsign}` | Aircraft callsign                                                     |
| `{squawk}`   | Actual transponder code reported by the aircraft                      |
| `{alt}`      | Pressure altitude as 3-digit FL (`190` = 19 000 ft, `050` = 5 000 ft) |
| `{altft}`    | Pressure altitude in raw feet (e.g. `19000`)                          |
| `{fl}`       | Same as `{alt}` — alias for clarity                                   |
| `{gs}`       | Reported ground speed (knots)                                         |
| `{hdg}`      | Reported heading (degrees)                                            |
| `{vs}`       | Vertical speed (feet per minute)                                      |

Flight-plan / controller-assigned data (empty if no flight plan correlated):

| Placeholder | What it shows                                                                           |
| ----------- | --------------------------------------------------------------------------------------- |
| `{type}`    | Aircraft ICAO type (e.g. `A320`, `B738`)                                                |
| `{cfl}`     | Cleared altitude as 3-digit FL (`240` = 24 000 ft, `030` = 3 000 ft). Empty if not set. |
| `{cflft}`   | Cleared altitude in raw feet                                                            |
| `{asquawk}` | Assigned (not necessarily set yet) squawk code                                          |
| `{spd}`     | Assigned speed                                                                          |
| `{ahdg}`    | Assigned heading                                                                        |
| `{scratch}` | Controller scratchpad string                                                            |
| `{dep}`     | Departure airport (origin)                                                              |
| `{arr}`     | Arrival airport (destination)                                                           |
| `{sid}`     | SID name                                                                                |
| `{star}`    | STAR name                                                                               |
| `{rwy}`     | Departure runway                                                                        |

Anything between `{` and `}` that isn't a known placeholder is echoed back
literally, so typos are visible (`{calsign}` shows as `{calsign}` in the
tag). Any text outside `{...}` is rendered literally.

#### Example tag formats

Minimal callsign-only (the default if you set nothing):

```
TAG_LINE:{callsign}
```

Compact ATC-style:

```
TAG_LINE:{callsign} {squawk}
TAG_LINE:{fl} {gs}
```

Full IFR clearance summary:

```
TAG_LINE:{callsign}
TAG_LINE:{type} {dep}-{arr}
TAG_LINE:{fl}/{cfl} {gs}kt
TAG_LINE:{scratch}
```

Ground-radar-ish (squawk + scratchpad first):

```
TAG_LINE:{callsign} {squawk}
TAG_LINE:{scratch}
TAG_LINE:{rwy} {sid}
```

---

## Chat commands

All issued in the EuroScope command bar:

| Command               | Effect                                                    |
| --------------------- | --------------------------------------------------------- |
| `.sw reload`          | Re-read both `.txt` files from disk                       |
| `.sw load <path>`     | Load a different map file (e.g. Ground Radar's)           |
| `.sw settings <path>` | Load a different settings file                            |
| `.sw ese <path>`      | Import labels from a `.ese` file (one category → one map) |
| `.sw new`             | Open another Secondary Window (up to 5 total)             |
| `.sw show`            | Show all hidden windows                                   |
| `.sw hide`            | Hide all windows                                          |

---

## All References

Every `KEY:value` line you can put in either of the `.txt` files, in one
place. Anything not listed is ignored, so the parser silently accepts
unknown lines without complaining.

### `SecondaryWindowMap.txt`

| Key               | Value                    | Remarks                                                                                                                 |
| ----------------- | ------------------------ | ----------------------------------------------------------------------------------------------------------------------- |
| `COLORDEF:`       | `Name:R:G:B`             | Defines a named color. R/G/B are 0–255.                                                                                 |
| `MAP:`            | `display name`           | Starts a new map block; everything below belongs to it until the next `MAP:`                                            |
| `FOLDER:`         | `name`                   | Optional folder/grouping label (not currently shown in UI but parsed).                                                  |
| `AIRPORT`:        | `ICAO`                   | Optional metadata; stored, not rendered.                                                                                |
| `ACTIVE:`         | `0` or `1`               | Initial visibility. Default `1` (visible).                                                                              |
| `COLOR:`          | `name`                   | Sets the active color (must match a prior `COLORDEF`). Applies to all shapes/labels that follow until the next `COLOR:` |
| `COORDTYPE:`      | `OTHER:REGION`           | Starts a **filled** closed polygon — see `COORD:` lines below. Block ends at next `COORDTYPE:` / `MAP:` / EOF.          |
| `COORDTYPE:`      | `OTHER:POLYGON`          | Starts an **outline only** closed shape. Last vertex auto connects to the first.                                        |
| `COORDTYPE:`      | `OTHER:POLYLINE`         | Starts an **open** line strip (no closing segment).                                                                     |
| `COORD:`          | `lat:lon`                | Adds a vertex to the active polygon / polyline. Lat/lon in DMS format (`N014.30.00.000` etc.).                          |
| `POLYGON`         | *(none)*                 | Legacy alias for `COORDTYPE:OTHER:REGION`. Closed by an `ENDPOLYGON` line.                                              |
| `ENDPOLYGON`      | *(none)*                 | Ends a legacy `POLYGON` block.                                                                                          |
| `LINE:`           | `lat1:lon1:lat2:lon2`    | One-shot straight segment in current color.                                                                             |
| `TEXT:`           | `lat:lon:label-text`     | Label at a coord, current color, current size. Label may contain colons.                                                |
| `TEXT_SIZE:`      | `n`                      | Sticky font size in pixels for following `TEXT:` and ESE labels. `0` = default.                                         |
| `INCLUDE_ESE:`    | `path`                   | Imports labels from a `.ese` files. Path can be relative to the map file or absolute.                                   |
| *raw line*        | `lat:lon:category:label` | Inline ESE label. Category becomes its own map under folder `ESE`.                                                      |
| `FILTER_ALT_MIN:` | `feet`                   | Lower bound of traffic-altitude filter. Targets below are hidden. Default: `INT_MIN`.                                   |
| `FILTER_ALT_MAX:` | `feet`                   | Upper bound of traffic-altitude filter. Targets above are hidden. Default: `INT_MAX`.                                   |

### `SecondaryWindowSettings.txt`

| Key                  | Value            | Default       | Remarks                                                                                                               |
| -------------------- | ---------------- | ------------- | --------------------------------------------------------------------------------------------------------------------- |
| `//`                 | anything         | —             | Comment line.                                                                                                         |
| `BACKGROUND_COLOR`   | `R:G:B`          | `128:128:128` | Map area background.                                                                                                  |
| `BORDER_COLOR`       | `R:G:B`          | `40:40:40`    | Outline around the window.                                                                                            |
| `TITLE_BAR_COLOR`    | `R:G:B`          | `80:80:80`    | Strip behind the title text.                                                                                          |
| `TITLE_TEXT_COLOR`   | `R:G:B`          | `220:220:220` |                                                                                                                       |
| `RESIZE_GRIP_COLOR`  | `R:G:B`          | `140:140:140` |                                                                                                                       |
| `LINE_WIDTH`         | `n` (px)         | `1`           | Width for lines, polygon outlines, polylines.                                                                         |
| `FILL_POLYGONS`      | `true` / `false` | `true`        | `false` makes `REGION` shapes outline-only.                                                                           |
| `LABEL_FONT_FACE`    | font name        | `Consolas`    | Font for `TEXT:` labels                                                                                               |
| `LABEL_FONT_SIZE`    | `n` (px)         | `12`          | Default label height. Overridable per section with `TEXT_SIZE:`.                                                      |
| `LABEL_FONT_BOLD`    | `true` / `false` | `false`       | Bold label text.                                                                                                      |
| `TRAFFIC_DOT_COLOR`  | `R:G:B`          | `255:255:0`   | Aircraft dot color.                                                                                                   |
| `TRAFFIC_DOT_RADIUS` | `n` (px)         | `3`           | Width of the square dots.                                                                                             |
| `SHOW_TAGS`          | `true` / `false` | `true`        | Master switch for callsign labels.                                                                                    |
| `TAG_COLOR`          | `R:G:B`          | `255:255:0`   | Default tag text color.                                                                                               |
| `COLOR_DEPARTURE`    | `R:G:B`          | *(unset)*     | Override tag color when the aircraft's origin airport is active for departure in EuroScope.                            |
| `COLOR_ARRIVAL`      | `R:G:B`          | *(unset)*     | Override tag color when the aircraft's destination airport is active for arrival in EuroScope. Wins over departure.    |
| `TAG_FONT_FACE`      | font name        | `Consolas`    | Tag font.                                                                                                             |
| `TAG_FONT_SIZE`      | `n` (px)         | `12`          | Tag font height.                                                                                                      |
| `TAG_FONT_BOLD`      | `true` / `false` | `false`       | Bold tag text.                                                                                                        |
| `TAG_OFFSET_X`       | `n` (px)         | `5`           | Default horizontal offset of tag from the dot. Per-aircraft offset wins once a tag is dragged.                        |
| `TAG_OFFSET_Y`       | `n` (px)         | `-3`          | Default vertical offset (negative = above the dot).                                                                   |
| `TAG_LINE`           | format string    | —             | Adds one line to every tag. Repeat for multiple lines. Supports `{placeholder}`. See the *Tag content* section above. |

---

## Window controls

| Action                       | Result                                                                 |
| ---------------------------- | ---------------------------------------------------------------------- |
| Drag title bar               | Move the window                                                        |
| Drag bottom-right grip       | Resize                                                                 |
| Drag inside map area         | Pan the map view                                                       |
| Mouse wheel                  | Zoom                                                                   |
| Right-click map area         | Open the maps popup (toggle individual maps on/off, reload from files) |
| Click the X in the title bar | Hide the window (`.sw show` to bring it back)                          |

---
