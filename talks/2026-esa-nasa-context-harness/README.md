# Context & Harness Engineering — lessons from AKD

15-minute talk for the **2nd ESA-NASA International Workshop on AI Foundation Models for Earth Observation**, Day 4 / Track 1 (GeoAI Agent Tutorial), NASA Marshall Space Flight Center.

## Files

```
.
├── slides.md          # source of truth (Marp markdown)
├── reveal/index.html  # Reveal.js shell that loads ../slides.md verbatim
├── assets/            # hand-authored hero SVGs
├── diagrams/          # Mermaid sources for the three flowcharts
└── README.md          # you are here
```

`slides.md` is authored once and consumed by both Marp and Reveal.js. Diagrams render inline:

- **Marp** parses fenced ```` ```mermaid ```` blocks via the Marp Mermaid plugin (or the `kroki` route).
- **Reveal.js** renders them through `reveal.js-mermaid-plugin` loaded from CDN.

## Render — Marp

```bash
# PDF (best for sharing / archiving)
npx -y @marp-team/marp-cli@latest \
  --html --allow-local-files \
  slides.md -o slides.pdf

# HTML (standalone single-file)
npx -y @marp-team/marp-cli@latest slides.md -o slides.html

# PPTX (for editing in PowerPoint/Keynote)
npx -y @marp-team/marp-cli@latest slides.md -o slides.pptx
```

For inline Mermaid in Marp, install the Mermaid plugin pack or use Kroki:

```bash
npx -y @marp-team/marp-cli@latest --theme-set themes/ slides.md \
  --engine @kazumatu981/markdown-it-kroki
```

## Render — Reveal.js

```bash
# Any static server works.
cd reveal && python3 -m http.server 8000
# then open http://localhost:8000
```

Reveal.js is loaded from a CDN, so no `npm install` is required. The shell at `reveal/index.html` pulls `../slides.md` through the `markdown` plugin, splits on `---`, and renders inline.

## Speaker notes

In Marp these are after `<!-- _footer: ... -->` comment markers (Marp presenter mode reads them). In Reveal.js the same markdown blocks are auto-extracted by the markdown plugin's `notes-separator`.

## Quick edit cycle

```bash
# Live preview Marp:
npx -y @marp-team/marp-cli@latest slides.md -w --preview
```

## Attribution

- AKD content authored by Nishan Pantha (NASA-IMPACT).
- AKD agent-development-lifecycle Mermaid diagram reused from `akd-suite/README.md`.
- External quotes are sourced; see `slides.md` slide 15.
