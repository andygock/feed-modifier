# Feed rate modifier for CNC machining

Drag and drop a NC file into the left hand textarea.

Modify feed rate with percentage slider.

Download output modified NC file.

The app looks for `F*` values and changes them.

<https://feed-mod.surge.sh/>

## Developer notes

Install dependencies

    pnpm i

Run dev server

    pnpm dev

Publish to surge

    pnpm deploy
