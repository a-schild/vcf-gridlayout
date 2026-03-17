# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Vaadin Flow component add-on that recreates the Vaadin 7 `GridLayout` for Vaadin Flow (24+). It is a Maven multi-module project:
- `vcf-gridlayout` — the component library (JAR)
- `vcf-gridlayout-demo` — a demo web application (WAR)

## Build Commands

```bash
# Build and install the add-on to local Maven repository
mvn clean install

# Run the demo locally (http://localhost:8080)
cd vcf-gridlayout-demo && mvn jetty:run

# Build for Vaadin Directory distribution (creates ZIP)
mvn clean install -Pdirectory

# Production build of the demo
cd vcf-gridlayout-demo && mvn jetty:run -Pproduction
```

Maven is at `c:\Program Files\Apache Software Foundation\apache-maven-3.8.6`. Java 17 is required for the component module (`c:\Program Files\Zulu\zulu-17`).

## Architecture

`GridLayout` extends `Composite<Div>` — it wraps a plain `<div>` and configures it as a **CSS Grid**. Key design points:

- Components added to the grid are each wrapped in a `<div class="v-slot">` before being appended to the root div.
- Grid sizing is applied via inline CSS (`grid-template-columns`, `grid-template-rows`).
- Each slot's position is set via `grid-column-start/end` and `grid-row-start/end`.
- An internal `Area[][]` grid map tracks occupied cells and detects overlaps (`OverlapsException`) and out-of-bounds placements (`OutOfBoundsException`).
- A cursor (x/y) enables sequential `addComponent(component)` calls without explicit coordinates.

### Key classes

| Class | Purpose |
|---|---|
| `GridLayout` | Main component; manages the CSS Grid, cursor, margin/spacing state, and column widths |
| `Alignment` | 9 pre-defined alignment constants (TOP_LEFT … BOTTOM_RIGHT); bitmask-based; maps to CSS justify/align values |
| `AlignmentInfo` | Lower-level alignment bits used internally |
| `MarginInfo` | Bitmask describing which edges (top/right/bottom/left) have margin applied |

CSS is in `vcf-gridlayout/src/main/resources/META-INF/resources/frontend/styles/grid-layout.css` and uses Lumo design tokens (`--lumo-space-m`, `--lumo-space-xl`) for spacing/margin.

### Deprecated API

`setColumnExpandRatio(int column, float ratio)` is deprecated — use `setColumnWidth(int column, String width)` or `setColumnWidth(int column, float width, Unit unit)` instead.

## No Tests

The project currently has no unit or integration tests (`src/test` directories do not exist).

## Publishing

Artifacts are published to GitHub Packages (Aarboard-AG organization). The CI workflow (`.github/workflows/maven.yml`) builds and deploys on push to `main`, `master`, `develop`, and `24-vaadin` branches using `GITHUB_TOKEN`.
