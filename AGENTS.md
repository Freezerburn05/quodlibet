# AGENTS.md

Quod Libet (music library/player) + Ex Falso (tag editor) + Operon (CLI). Python 3.10+,
GTK3/PyGObject + GStreamer. Dev docs: `docs/development/` (esp. `devenv.rst`, `testing.rst`, `guidelines.rst`).

## Environment

- Python deps via Poetry 2 (`poetry install`, add `-E plugins` for optional plugin deps) or uv (`uv sync --extra plugins`). Both `poetry.lock` and `uv.lock` exist — keep both in sync when changing dependencies.
- Native deps (GTK3, PyGObject, GStreamer incl. good/bad/ugly/libav plugins, dbus, gettext, Xvfb) are the hard part; `nix develop` provides a complete dev shell via `flake.nix`.
- Run the apps from the checkout: `./quodlibet.py`, `./exfalso.py` — no install step needed.

## Tests

- Full suite: `poetry run pytest tests/` or `poetry run ./setup.py test` (what CI runs). Options: `--no-quality`, `--no-network`, `--exitfirst/-x`, `--strict` (glib warnings fatal).
- Single test: `poetry run pytest tests/test_formats_mp3.py::TMP3File::test_channels`.
- Lint + typecheck run as pytest tests marked `quality` (ruff check/format + mypy, `tests/quality/`). Run just those: `poetry run pytest tests/quality`. Ruff is pinned (`==0.7.3`); mypy pinned `<1.19`. `pytest tests/quality` is the lint+typecheck gate.
- Markers: `network` (needs internet), `quality`.
- Importing anything from the `tests` package triggers `tests/__init__.py::init_test_environ()`: isolated fake `$HOME`, session dbus, virtual display (Xvfb via pyvirtualdisplay on Linux), forced Adwaita theme, `GSETTINGS_BACKEND=memory`. GTK/GStreamer must be present even for "non-GUI" tests.
- `tests.TestCase` swaps `assertEqual(actual, expected)` argument order (actual first — opposite of unittest default).

## Code conventions

- All user-visible strings must be translatable: `from quodlibet import _` / `ngettext` (also `C_`, `N_`, `npgettext` via `quodlibet.util.i18n`). NOTE: `docs/development/guidelines.rst` still claims `_()` is a builtin — it is not anymore; it must be imported.
- All terminal output goes through `print_d` / `print_w` / `print_e` (`quodlibet/util/dprint.py`), never bare `print`.
- Add your name to the copyright header of every file you touch (project convention, see `docs/development/guidelines.rst`).
- Ruff has many per-file naming ignores (`qltk/` allows `N802`/`N806`, `ext/` allows `N802`, `query/`, `pattern/`, `mmkeys/`, `util/i18n.py` etc.). Don't "fix" nonstandard names there — it's intentional.
- Line length 88, target py310.

## Structure

- `quodlibet/` single package: `browsers/` (View-menu browsers), `formats/` (file-format support), `library/`, `qltk/` (GTK widgets), `player/` (audio backends), `plugins/` (plugin base classes), `ext/` (bundled plugins, organized by type), `operon/` (CLI), `query/`, `pattern/`, `util/`.
- Good entry points for understanding the code: `browsers/_base.py`, `formats/_audio.py`, `library/libraries.py`.
- `setup.py` uses a custom `gdist` distutils distribution (local `gdist/` package); most custom commands live there. `setup.py test` wraps pytest via `tests.unit()`.

## Gotchas

- Version lives in three places: `pyproject.toml`, `quodlibet/const.py` (`VERSION_TUPLE`), and `data/*.appdata.xml.in`. Release checklist: `docs/development/maintainers.rst`.
- `po/` translations are updated via Weblate bots — don't hand-edit `.po` files in PRs.
- Useful env vars for running from source: `QUODLIBET_USERDIR` (fresh/isolated config dir), `QUODLIBET_BACKEND` (audio backend override), `QUODLIBET_DEBUG`.
- CI runs the suite on Linux (poetry + apt), Windows/msys2 (`MSYSTEM=` is deliberately unset for poetry), macOS, and flatpak (`.ci/`).
