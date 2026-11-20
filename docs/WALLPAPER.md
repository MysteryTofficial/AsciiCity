# Wallpaper guide

ASCII CITY can be used two ways: as a **static PNG** (any desktop, zero runtime cost) or as a **live animated wallpaper** (a borderless browser window behind your icons).

All commands below assume you are in this repository's folder. Replace `<repo>` with your actual path if needed.

[← Back to README](../README.md)

---

## 1. Static PNG (all platforms)

Render a deterministic frame straight from the browser's command line:

```sh
chromium --headless --disable-gpu --hide-scrollbars \
  --window-size=3840,2160 \
  --virtual-time-budget=4000 \
  --screenshot=/tmp/city.png \
  "file:///path/to/index.html?seed=harbor7&still=1&ui=0&hour=17&font=18&grain=0"
```

Useful tweaks:

- `window-size` controls the output resolution. `1920,1080`, `2560,1440`, `3840,2160` all work.
- `font` controls glyph size — larger values suit high-resolution displays.
- `hour` sets the lighting; `17` is golden hour, `23` is night with lamps.
- `still=1` runs the simulation briefly and renders one frame, so the output is reproducible.
- `grain=0` gives cleaner pixels; keep grain on for a filmic look.
- Try `weather=storm` or `weather=snow` for variety.

You can also press <kbd>S</kbd> in a normal window to download the current frame.

### Linux — set the PNG

```sh
# feh
feh --bg-fill /tmp/city.png

# swww (Wayland)
swww img /tmp/city.png

# GNOME
gsettings set org.gnome.desktop.background picture-uri "file:///tmp/city.png"
gsettings set org.gnome.desktop.background picture-uri-dark "file:///tmp/city.png"

# KDE Plasma
plasma-apply-wallpaperimage /tmp/city.png

# XFCE (path varies by monitor; list with xfconf-query -l | grep backdrop)
xfconf-query -c xfce4-desktop \
  -p /backdrop/screen0/monitor0/image-path -s /tmp/city.png
```

### macOS — set the PNG

```sh
osascript -e 'tell application "System Events" to set picture of every desktop to "/tmp/city.png"'
```

### Windows — set the PNG (PowerShell)

```powershell
Add-Type -TypeDefinition @'
using System;
using System.Runtime.InteropServices;
public static class Wallpaper {
  [DllImport("user32.dll", CharSet = CharSet.Auto)]
  public static extern int SystemParametersInfo(int action, int param, string path, int flags);
}
'@
[Wallpaper]::SystemParametersInfo(20, 0, "C:\city.png", 3)
```

---

## 2. Live wallpaper on X11

Use `xwinwrap` to pin a browser window to the desktop root:

```sh
xwinwrap -ni -fs -s -st -sp -nf -b -- \
  chromium --app="file:///path/to/index.html?seed=harbor7&ui=0" \
  --window-position=0,0
```

Flags worth knowing:

- `-ni` ignores the window manager, `-fs` fullscreen, `-s` stretches, `-st` sets the window type to desktop.
- `-b` puts it behind other windows; drop it if your compositor misbehaves.
- Add `--disable-gpu` to Chromium if animation stutters on software rendering.

Without `xwinwrap`, a quick approximation is a borderless app window:

```sh
chromium --app="file:///path/to/index.html?seed=harbor7&ui=0" --start-fullscreen
```

Then use your window manager to lower it below other windows.

---

## 3. Live wallpaper on Wayland

Wayland compositors don't let normal windows sit below the desktop, so you need a **layer-shell** client. Run the `--app` command above inside a browser-based layer-shell wallpaper wrapper (for example a `hyprpaper-live`-style wrapper, or a browser surface managed by your compositor's layer-shell support).

The page itself does not care how it is hosted — as long as the browser gets a window, the animation runs. Static PNGs via `swww` remain the simplest and most battery-friendly option.

---

## 4. Live wallpaper on macOS / Windows

Neither OS exposes a supported "window behind the desktop" API to browsers, so the practical options are:

- Use a **web-wallpaper utility** that pins a Chromium window to the desktop (for example Plash on macOS or a WebView2-based wallpaper tool on Windows).
- Or keep a borderless app-mode window in fullscreen and switch to a mostly empty workspace:

```sh
chromium --app="file:///path/to/index.html?seed=harbor7&ui=0" --start-fullscreen
```

- Or simply use the static PNG recipe above — it looks nearly identical and uses no CPU.

---

## Multi-monitor setup

1. Give every window the **same seed** and hide the HUD:

   ```
   chromium --app="file:///path/to/index.html?seed=harbor7&ui=0" --start-fullscreen
   ```

2. Place the windows so they touch edge-to-edge in your display arrangement (left monitor's right edge against the right monitor's left edge). The page reads `screenX`/`screenY` and window size to figure out which neighbour is east, west, north or south.

3. Press <kbd>H</kbd> in any window. The HUD shows `link EW` (or similar) — the letters are the connected sides. `w0`, `w1`, … are the linked "world" indices, one per screen.

4. The oldest window leads the clock and weather. Change time or weather there and the others follow. Press <kbd>R</kbd> in one window to reseed all of them.

5. Watch the edges: traffic lights and road stubs appear where a neighbouring city connects, and vehicles cross over.

If windows fail to link, make sure they are same-origin (same file path or same HTTP host), that the browser allows `BroadcastChannel`/`localStorage`, and give them a second to exchange hellos.

---

## Performance tips

| Symptom | Try |
| :--- | :--- |
| High CPU on a big display | Larger `font` (e.g. `20`), `cols`/`rows` caps, or static `still=1` |
| Too many vehicles | `traffic=0.5`, `people=0.5` |
| Grain shimmer distracts | `grain=0` |
| Want a calmer scene | `modes=road,rail` to disable boats, planes and pedestrians |
| Screenshot looks washed out | `vignette=0`, `grain=0`, choose `hour=17` or `hour=23` |

---

## Troubleshooting

- **Black screen** — the browser may block `file://` canvas reads in export mode; serve the folder over HTTP (`python3 -m http.server`) and retry.
- **Vehicles vanish at the edge** — a neighbour is missing or not linked; check the HUD's `link` status.
- **Windows disagree on time** — the oldest window is the leader; reload the followers.
- **No lamps at night** — `lights=0` disables them; remove that parameter or press <kbd>L</kbd>.
