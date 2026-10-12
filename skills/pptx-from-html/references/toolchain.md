# Toolchain (already installed in the Paperclip image)

Available in the container, no install needed:
- `pptxgenjs` and `playwright-core` (global, `NODE_PATH=/usr/local/lib/node_modules`)
- Chromium at `/usr/bin/chromium` (use `executablePath` and `args: ['--no-sandbox']`)
- `soffice` (LibreOffice Impress/Writer) and `pdftoppm` (poppler)
- `python-pptx`, `python3-pil`
- Fonts: Liberation, DejaVu, Carlito (= Calibri metrics), Caladea (= Cambria metrics), Noto. Verdana/Arial/Calibri are substituted in renders; the PPTX itself keeps the font name.

Do not install anything with apt/npm/pip during a run. If a tool is missing, say so in the task.

## Measure the HTML boxes (Chromium)
```js
const {chromium}=require('playwright-core');
const b=await chromium.launch({executablePath:'/usr/bin/chromium',args:['--no-sandbox']});
const pg=await b.newPage({viewport:{width:1280,height:720}});
await pg.goto('file:///path/deck.html');
const boxes=await pg.$$eval('.slide [data-pptx]',els=>els.map(e=>{const r=e.getBoundingClientRect();const s=e.closest('.slide').getBoundingClientRect();
  return {id:e.dataset.pptx,x:r.x-s.x,y:r.y-s.y,w:r.width,h:r.height,slide:[...document.querySelectorAll('.slide')].indexOf(e.closest('.slide'))}}));
```
Convert px to inches with `/96`. Mark every element that becomes a PPTX object with `data-pptx`.

## pptxgenjs pointers
`pptx.layout='LAYOUT_WIDE'` (13.33x7.5 in), `pptx.defineSlideMaster({title,background,objects,slideNumber})`,
`slide.addChart(pptx.charts.BAR,data,opts)`, `slide.addShape(pptx.shapes.ROUNDED_RECTANGLE,{...})`, `addTable`, `addText`.

## Render check
`soffice --headless --convert-to pdf deck.pptx && pdftoppm -r 60 -png deck.pdf slide`
