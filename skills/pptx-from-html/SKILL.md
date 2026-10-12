---
name: pptx-from-html
description: >
  Build an editable, good-looking PPTX by designing each slide in HTML/CSS first and then converting it to native
  PowerPoint objects (text, shapes, tables, charts). Use when a deliverable is a presentation, slide deck or PPTX
  template. Do not use for web pages, PDFs-only deliverables or when only the text of an existing PPTX must change.
---

# PPTX from HTML

Goal: a PPTX that looks designed (type, shapes, data visuals) and stays editable in PowerPoint.

## Rules
- 16:9. Design slides at 1280x720 px (HTML) = 13.333 x 7.5 in (PPTX). 1 px = 1/96 in.
- Native objects only: text boxes, shapes, tables, charts with embedded data. No full-slide images. Images only for photos/logos/slots the brief lists.
- Fonts: use fonts PowerPoint has everywhere (Calibri, Arial, Verdana, Georgia) unless the brief names a brand font. State the fallback. A render with a substituted font looks worse than the file; judge layout with that in mind and keep 15% spare width in text boxes.
- Min sizes: body 18 pt, captions 12 pt, titles 32 pt+. Contrast WCAG AA.
- Visual quality bar: one idea per slide, a dominant element (big number, chart, diagram, image slot), strong hierarchy, consistent grid, intentional white space. Not a bullet wall, not a wireframe.
- Data visuals: big stat blocks with shape behind, bar/line charts as native charts, timelines and process flows from shapes + connectors, icon grids from simple vector shapes.
- Demo data: always label "DEMONSTRATIEDATA".

## Steps
1. Write `deck.html`: `<section class="slide">` per slide, CSS variables for the tokens from the brief, absolute positioning inside the slide for anything that becomes a shape. Open and render it (headless browser if available) to check overlap/clipping.
2. Convert with pptxgenjs (see `references/toolchain.md`): map each HTML element to a pptxgenjs call using its measured box (x, y, w, h in px -> inches). Use `slide.addShape`, `addText`, `addTable`, `addChart`. Define masters with `defineSlideMaster` for reusable layouts (title, section, content, ...), and placeholders for editable content.
3. Render: `soffice --headless --convert-to pdf deck.pptx`, then `pdftoppm -r 60 -png` to inspect every slide. Fix overlap, clipping, unreadable text. One fix round.
4. Verify editability: open the PPTX with a zip/XML check: no `<p:pic>` covering full slides, charts have embedded workbook, text is in `<a:t>`.
5. Attach PPTX, PDF preview and overview PNG as work products. Note what was not tested (e.g. not opened in PowerPoint).

## Do not
- Do not reinstall the toolchain every run: check `references/toolchain.md` first.
- Do not loop on pixel-perfect conversion. Two failed attempts: report the blocker with the slide number.
