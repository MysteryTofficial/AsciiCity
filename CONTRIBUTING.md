# Contributing to ASCII CITY

Thanks for wanting to make the city better. A few ground rules keep this project small and portable:

## Ground rules

- **Keep it one file.** The whole app lives in `index.html`. No frameworks, no npm, no CDN links, no network requests.
- **Keep it dependency-free.** It must still run from `file://` with a double-click.
- **Keep it fast.** The render loop runs ~30 times a second; prefer batched `fillRect` calls and typed arrays over per-glyph DOM work.
- **Keep it seed-stable.** Changing generation logic changes what existing seeds produce. If you must, note it in your PR.

## Ways to help

- **Bug reports** — include your browser/OS, the exact URL parameters, the seed, and a screenshot if possible.
- **New palettes** — palettes live in the `MT` array near the top of the script (each entry is `[fgDay, fgNight, bgDay, bgNight]`).
- **Wallpaper recipes** — add your desktop/window-manager instructions to [`docs/WALLPAPER.md`](docs/WALLPAPER.md).
- **Documentation** — clearer words, more examples, and new screenshot sets are always welcome.
- **Performance** — profiles and micro-optimisations are especially appreciated for 4K and multi-monitor setups.

## Testing locally

```sh
python3 -m http.server 8000
# open http://localhost:8000/index.html?seed=test&ui=1
```

Check at least:

1. Day and night (`hour=12`, `hour=23`) with lamps on/off (<kbd>L</kbd>).
2. Every weather mode, including `auto`.
3. `still=1` screenshot export with a couple of seeds.
4. Two windows side by side to confirm vehicle hand-off still works.
5. No console errors.

## Submitting

1. Fork the repo and create a branch.
2. Make your change, test it, and update the README if behaviour changes.
3. Open a pull request describing **what** changed and **why**, plus the seed you tested with.

By contributing you agree that your work is licensed under the [MIT License](LICENSE).
