# Contributing to Omni-Stand

First off, thank you for considering contributing to **Omni-Stand**! Modular and community-driven expansion is at the core of this project.

Please take a moment to review this document to ensure a smooth workflow and maintain high parametric consistency across the codebase.

---

## Code of Conduct

This project adheres to standard open-source community guidelines. Please be respectful, constructive, and considerate of other contributors in issues, discussions, and pull requests.

See [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) for details.

---

## Architecture Overview

Omni-Stand uses OpenSCAD's `include <>` mechanism (not `use <>`) so global variables defined in `main.scad` (such as tolerances, base parameters, and margins) remain accessible across all submodules.

```text
omni-stand/
+-- main.scad                    # Customizer parameters + render stage
+-- build.py                     # Bundler: inlines include <> into omni-stand_merged.scad
+-- modules/
    +-- core_logic.scad          # Derived dims, validation asserts, warnings
    +-- stand_body.scad          # Solid extrusion + cutout wiring (omni_stand)
    +-- cutouts/
        +-- cutouts.scad         # Cutouts aggregator
        +-- base/helpers.scad    # trapezoid_profile(), rounded_box_2d()
        +-- pockets.scad         # Floor cavity, lip pocket, rear pocket
        +-- cable_channels.scad  # Cable pass-throughs
        +-- charging.scad        # Dock slot, strain relief, under-base channel, clip
        +-- engraving.scad       # Front recessed text
```

---

## Adding a New Pocket / Cutout

Adding a cutout does not require editing the solid extrusion. Follow these steps:

1. **Create the Leaf Cutout Module:**
   - Add a new `module <name>_cut()` in the matching family file (`pockets.scad`, `cable_channels.scad`, or `engraving.scad`).
   - Keep the cutout centered at the caller transform and bore along the documented axis.
   - Apply `print_tolerance` to every fit-critical diameter or width.

2. **Wire the Assembly:**
   - In `modules/stand_body.scad`, call the new cutout module inside `omni_stand()` within the `difference()` block.

3. **Expose in `main.scad`:**
   - Add the Customizer toggles and dimension variables (`enable_*`, `*_width`, `*_height`, `*_pos_percent`) in the appropriate Customizer section with `// [min:step:max]` annotations.

4. **Validate:**
   - In `modules/core_logic.scad`, add `assert()` guards plus `echo()` warnings for impossible proportions (e.g. pocket wider than the pillar span).

5. **Rebuild the Merged Distribution:**
   - Run `python build.py` to regenerate `omni-stand_merged.scad`.

---

## Adding a New Cable Channel

1. Add position and enable flags in `main.scad` (`cable_<side>_enable`, `cable_<side>_pos_percent`).
2. Implement the bore in `modules/cutouts/cable_channels.scad` using `cable_diameter + print_tolerance`.
3. Clip side bores against the floor-margin mask so geometry cannot bleed outside the base.
4. Add a validation warning in `core_logic.scad` when the bore exceeds the local wall depth.
5. Run `python build.py`.

---

## Development & Testing Checklist

Before opening a pull request:

- [ ] Open `main.scad` in OpenSCAD (v2021.01+) and ensure it renders cleanly without compilation errors.
- [ ] Press **F5** (preview) and check the console for unexpected `WARNING` echoes from `validate_configuration()`.
- [ ] Render with **F6** to guarantee manifold geometry before STL export.
- [ ] Run `python build.py` and ensure `omni-stand_merged.scad` is up-to-date and compiles identically to the modular source.
- [ ] Keep indentation with TABS and headers with `@file` tags, following YAPBPS conventions.

---

## Submitting Pull Requests

1. Fork the repository and create your feature branch:
   ```bash
   git checkout -b feature/new-pocket-name
   ```
2. Commit your changes with clear, descriptive commit messages:
   ```bash
   git commit -m "feat(cutouts): add front lip pocket variant and validation"
   ```
3. Push your branch to GitHub:
   ```bash
   git push origin feature/new-pocket-name
   ```
4. Open a Pull Request targeting the `main` branch. Provide a brief explanation of the added component, its real-world dimensions, and any testing performed.

## Documentation conventions

These rules are shared by every project under [camiu01](https://github.com/camiu01) and keep the
documentation consistent from one repository to the next. Follow them when you add or edit any
Markdown file, and update this section in every repository if a rule changes.

### README.md

- Starts with a centered header: title, one-line tagline (`###`), badges on one line, a hero image
  (photo, GIF, render or screenshot), then quick links separated by ` · `.
- The intro prose follows the header, before the first section.
- Top-level sections, in this order, omitting the ones that do not apply:
  **What it does**, **Getting started**, **Usage**, **Configuration**, **How it works**,
  **Limitations**, **Development**, **License**.
- When a section is renamed, keep the old anchor with `<a id="old-anchor"></a>` above the heading.

### TODO.md

- Title `# TODO`, followed by `> Last reviewed: YYYY-MM-DD`.
- One `##` section per **area** of the project (for example Firmware, Web dashboard, Docs, CI).
  Areas are ordered by how much open work they hold.
- Inside an area, open items come first as `- [ ] **Short title** — context`, then completed
  items as `- [x] ...`. Use `Blocked:` or `Needs hardware:` at the start of the context when an
  item cannot progress.
- Completed items that shipped in a release move to `CHANGELOG.md`; keep only recent ones here.

### CHANGELOG.md

- Follows [Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/).
- Newest first: `## [Unreleased]`, then `## [x.y.z] - YYYY-MM-DD` (or `## YYYY-MM-DD` for projects
  without version numbers).
- Entries are grouped under `### Added`, `### Changed`, `### Deprecated`, `### Removed`,
  `### Fixed` and `### Security`, omitting empty groups.

### docs/

- `docs/README.md` is the index: the title, then a table of every document with a one-line
  description, then any reading paths.
- Every other file in `docs/` starts with its `# Title` followed by the navigation line
  `[← README](../README.md) · [Docs index](README.md)`.
- File-level comments required by a project (such as `@file`/`@brief`) stay above the title.

### Commits and files

- No AI attribution lines (such as `Co-Authored-By`) in commit messages or pull requests.
- Preserve each file's existing line endings and encoding.
