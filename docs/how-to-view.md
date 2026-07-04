# How to Open the Interactive Guide (docs/index.html)

The page is a single self-contained file — no build step, no server required.
It works anywhere a browser exists.

## Option 1: Open directly in a browser (fastest)
- **Windows:** double-click `index.html`, or right-click → Open with → Chrome/Edge/Firefox.
- **macOS:** double-click (opens in Safari), or drag the file onto any browser icon.
- **Linux:** double-click in your file manager, or run `xdg-open index.html`.
- **iPhone/iPad:** save the file to the Files app, tap it — it opens in a preview;
  for full interactivity, use a browser app that can open local files (e.g., open
  Safari and use a `file://` bookmark via the Shortcuts app), or use Option 3.
- **Android:** open the file from the Files app with Chrome ("Open with…").

## Option 2: Local server (best fidelity, matches how GitHub Pages serves it)
From the `docs/` folder:

    python3 -m http.server 8000

Then visit http://localhost:8000 in any browser. (Or use the VS Code
"Live Server" extension: right-click index.html → Open with Live Server.)

## Option 3: GitHub Pages (shareable URL, free)
1. Push this repo to GitHub.
2. Repo → Settings → Pages.
3. Source: "Deploy from a branch" → branch `main`, folder `/docs` → Save.
4. In ~1 minute your guide is live at
   `https://<username>.github.io/<repo>/`
   (Pages serves `docs/index.html` automatically as the site root.)

Note: GitHub Pages sites are public unless the repo is on a paid plan with
private Pages — don't put credentials or private data in the page.

## Option 4: Any HTML viewer
Because everything (CSS + JS) is inline in one file, any app that renders
HTML works: browser, VS Code preview extensions, or an HTML viewer app from
your platform's app store. If fonts don't load (offline), the page falls
back to system fonts and remains fully functional.
