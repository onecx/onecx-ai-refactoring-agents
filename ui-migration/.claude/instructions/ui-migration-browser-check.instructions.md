# Browser check with the OneCX test page (final verification step)

Run this once at final verification, after `npm run build`, `npm run lint` and `npm run test` are green.
It shows the built app in a real browser and gives the user a URL to open. Do not ask the user about it.

## What it is

`run-test-instance` from `onecx/onecx-design-skills` loads the **built remote app** (`dist/**/remoteEntry.js`)
on a test page that plays the OneCX Shell (topics, user, permissions, theme) and answers BFF calls with mock
data. It is not a copy of the app's DOM or templates, and it is not the real Shell. The test page, mocks and
screenshots live in `~/.cache/onecx-test/<appId>/`, outside the repo. The repo stays unchanged.

## What the developer must have

- `gh` logged in (`gh auth status`) with access to `onecx/onecx-design-skills`, Node and npm. Nothing else:
  the agent installs the rest by itself (step 2 below), without asking.
- Playwright's own Chromium. The runner calls `chromium.launch()` with no `channel`, so it does NOT use
  the system Chrome. The runner never installs it; it only prints the install commands and exits. Step 2
  runs them for you, once per machine, into `~/.cache/onecx-test/tools` (about 400-700 MB, several minutes).
  Nothing is added to the app repo.
- A built app (`npm run build`), so `dist/**/remoteEntry.js` exists.

## What this check can and cannot show

- **It does show:** the app loads without errors, routes render, grid and flex layout, widths, padding,
  margins, gaps, fonts, icons, borders, rounded corners, colours from the app's own `styles.css`, button and
  input styling from the Optimus component styles, overlays appended to `body` or to the dynamic container
  (datepicker, select, multiselect, popover, menu, dialogs, drawer) and console or network errors. Colours and
  overlays need the patch in step 1b.
- **It is not the real Shell.** Routing, federation and the Shell's own theme are simulated, and the Shell's
  scoping of dynamic content is imitated on the page (`reference.md`). A panel that looks right here can still
  differ in the real Shell, so keep the overlays on the Shell checklist.

These are limits of the test page, not app bugs. **Never change the app to make the screenshots look right**:
no hardcoded colours, no `--p-*` or `--op-*` variables, no `!important`, no extra CSS.

## 1. Download the latest release and verify it

Always the latest release, never a checkout of `main`. Needs `gh auth status` logged in, Node and npm.
If the download or checksum fails, report it as BLOCKED and stop. Do not run an unverified runner.

```bash
set -euo pipefail
mkdir -p "$HOME/.cache/onecx-component-tests"
WORK="$(mktemp -d "$HOME/.cache/onecx-component-tests/run.XXXXXXXX")"
TAG="$(gh release view --repo onecx/onecx-design-skills --json tagName --jq .tagName)"
ARCHIVE="$(gh release view "$TAG" --repo onecx/onecx-design-skills \
  --json assets --jq '.assets[].name | select(startswith("run-test-instance-") and endswith(".tar.gz"))')"
test -n "$ARCHIVE"
test "$(printf '%s\n' "$ARCHIVE" | wc -l)" -eq 1
gh release download "$TAG" --repo onecx/onecx-design-skills \
  --pattern "$ARCHIVE" --pattern SHA256SUMS --dir "$WORK"
(
  cd "$WORK"
  awk -v file="$ARCHIVE" '$2 == file { print; found++ } END { if (found != 1) exit 1 }' \
    SHA256SUMS > runner.sha256
  sha256sum --check --strict runner.sha256
  tar -xzf "$ARCHIVE" --no-same-owner --no-same-permissions
)
printf '%s\n' "$TAG" > "$WORK/runner-release.txt"
printf 'Work directory: %s\n' "$WORK"
```

Copy the printed `Work directory` path into every later command (shell variables may not survive between
tool calls). Read `$WORK/run-test-instance/SKILL.md`; read `reference.md` only if `check` reports a problem it
does not explain. Keep the whole folder, including `fonts/`. Below, `T` means
`node $WORK/run-test-instance/onecx-test.mjs`. Run every `T` command from the app repo root.
Do not copy the runner into the repo.

### 1b. Patch the skill (until the released skill has the fixes)

Verified on release v0.1.5, two defects of the test page. Both make the page look unlike the Shell:

1. Styles: the page links `styles.css` with a plain `<link>`. The Shell injects it as
   `<style data-app-styles="<scopeId>">` inside `@scope`, which is what lets the app's own `updateAppStyle()` rewrite
   `--p-` to the scoped prefix. With a `<link>`, colour utilities such as `tw-bg-surface-50` and
   `tw-border-surface-300` render transparent or black.
2. Dynamic content: the Shell puts everything appended to `<body>` (datepicker, select, multiselect and popover panels,
   dialogs, drawers) into a wrapper that carries the scope data of the element the user touched last. The page does
   not, so those panels sit outside the scope and render without any style.

The patch below makes the page do both the Shell way. It is safe to run every time: it skips what the skill already
does, and it stops with an error if the skill's code has changed shape (then report it; do not guess).

```bash
cat > "$WORK/patch-skill.cjs" <<'EOF'
const fs = require('fs');
const dir = process.argv[2];
const rp = dir + '/onecx-test.mjs', tp = dir + '/test.html';
let m = fs.readFileSync(rp, 'utf8'), t = fs.readFileSync(tp, 'utf8');
const needStyles = !t.includes('data-app-styles'), needScope = !t.includes('dynamicWrapperElement');
if (!needStyles && !needScope) { console.log('skill already has both fixes: no patch needed'); process.exit(0); }
const fail = () => { console.error('PATCH FAILED: the skill changed shape. Report it; do not guess.'); process.exit(1); };
const done = [];
if (needStyles) {
  const oldLine = "    .replaceAll('{{STYLES_LINK}}', styles ? `<link rel=\"stylesheet\" href=\"${base}${styles}\" />` : '')";
  const newLine = "    .replaceAll('{{STYLES_URL}}', styles ? `${base}${styles}` : '')";
  const oldPlaceholder = '{{STYLES_LINK}}\n';
  const marker = '  // 8. Run the app\'s bootstrap';
  const block = [
  "  // 7b. Load the app's styles.css like the shell: a <style data-app-styles> inside @scope, not a <link>. The app finds this",
  "  //     element by its scope id and rewrites the --p- variables to its scoped prefix; a <link> is never rewritten, so colour",
  "  //     utilities such as bg-surface-50 would render transparent. 'html' and ':root' become ':scope' as the shell does it.",
  "  const stylesUrl = '{{STYLES_URL}}';",
  "  if (stylesUrl) {",
  "    const scopeId = TEST.productName + '|' + TEST.appId;",
  "    const css = await fetch(stylesUrl).then((r) => r.text());",
  "    const style = document.createElement('style');",
  "    style.setAttribute('data-app-styles', scopeId);",
  "    style.textContent = `@scope([data-style-id=\"${scopeId}\"]:is([data-no-portal-layout-styles], [data-mfe-element])) to ([data-style-isolation]) { ${css.replaceAll('html', ':scope').replaceAll(':root', ':scope')} }`;",
  "    document.head.appendChild(style);",
  "  }",
  "",
  ""].join('\n');
  if (m.split(oldLine).length !== 2 || t.split(oldPlaceholder).length !== 2 || t.split(marker).length !== 2) fail();
  m = m.replace(oldLine, () => newLine);
  t = t.replace(oldPlaceholder, () => '').replace(marker, () => block + marker);
  done.push('app styles are loaded like the Shell');
}
if (needScope) {
  const hooksOld = [
  "// 2. Hooks the shell puts on document. The build of the app rewrites PrimeNG / Material code to call them.",
  "const createEl = (ctx, tag, opts) => document.createElement(typeof tag === 'string' ? tag : ctx?.element, opts);",
  "document.createOnecxElement = createEl;",
  "document.createElementFromSharedStylesHost = createEl;",
  "document.createElementFromPrimeNg = createEl;",
  "document.createElementFromMaterial = createEl;",
  "document.createOnecxDynamicContainer = (tag) => { const c = document.createElement(tag); document.body.appendChild(c); return c; };",
  ""].join('\n');
  const hooksNew = [
  "// 2. Hooks the shell puts on document. The build of the app rewrites PrimeNG / Material code to call them.",
  "const createEl = (ctx, tag, opts) => document.createElement(typeof tag === 'string' ? tag : ctx?.element, opts);",
  "",
  "// Scope data (data-style-id ...) of the closest element that has it. The shell reads it the same way.",
  "const styleDataOf = (el) => {",
  "  for (let e = el; e instanceof HTMLElement; e = e.parentElement) {",
  "    const d = e.dataset;",
  "    if (d.styleId || d.intermediateStyleId) {",
  "      return { styleId: d.styleId ?? d.intermediateStyleId,",
  "               noPortalLayoutStyles: d.noPortalLayoutStyles ?? d.intermediateNoPortalLayoutStyles,",
  "               mfeElement: d.mfeElement ?? d.intermediateMfeElement };",
  "    }",
  "  }",
  "  return null;",
  "};",
  "const STYLE_KEYS = ['styleIsolation', 'styleId', 'noPortalLayoutStyles', 'mfeElement'];",
  "const cap = (s) => s[0].toUpperCase() + s.slice(1);",
  "const setStyleData = (el, d, intermediate) => {",
  "  const key = (k) => (intermediate ? 'intermediate' + cap(k) : k);",
  "  el.dataset[key('styleIsolation')] = '';",
  "  if (d.styleId) el.dataset[key('styleId')] = d.styleId;",
  "  if (d.noPortalLayoutStyles || d.noPortalLayoutStyles === '') el.dataset[key('noPortalLayoutStyles')] = d.noPortalLayoutStyles;",
  "  if (d.mfeElement || d.mfeElement === '') el.dataset[key('mfeElement')] = d.mfeElement;",
  "};",
  "const clearStyleData = (el) => {",
  "  if (el.dataset) for (const k of STYLE_KEYS) { delete el.dataset[k]; delete el.dataset['intermediate' + cap(k)]; }",
  "  for (const c of Array.from(el.children)) if (c instanceof HTMLElement) clearStyleData(c);",
  "};",
  "",
  "// The shell takes the scope of dynamic content from the element the user touched last.",
  "let triggerElement = null;",
  "for (const type of ['mouseover', 'focusin']) document.addEventListener(type, (e) => { triggerElement = e.target; });",
  "const triggerStyleData = () => styleDataOf(triggerElement) ?? styleDataOf(document.querySelector('#onecx-test-app [data-style-id]'));",
  "",
  "document.createOnecxElement = (ctx, tag, opts) => {",
  "  const el = createEl(ctx, tag, opts);",
  "  const d = styleDataOf(ctx?.this?.el?.nativeElement) ?? triggerStyleData();",
  "  if (d) setStyleData(el, d, true);",
  "  return el;",
  "};",
  "document.createElementFromPrimeNg = document.createOnecxElement;",
  "document.createElementFromMaterial = (ctx, tag, opts) => {",
  "  const el = createEl(ctx, tag, opts);",
  "  const d = triggerStyleData();",
  "  if (d) setStyleData(el, d, true);",
  "  return el;",
  "};",
  "document.createElementFromSharedStylesHost = createEl;",
  "",
  "// Like the shell, put every element that is appended to <body> (PrimeNG / Optimus overlays, dialogs) into a wrapper that",
  "// carries the scope data of the element the user touched last. Without it the scoped styles do not reach the overlay.",
  "// The wrapper is removed with its content.",
  "const WRAPPER = 'dynamicWrapperElement';",
  "// The script runs in <head>, so patch the prototype of <body> instead of the not yet existing element.",
  "const bodyAppendChild = HTMLBodyElement.prototype.appendChild;",
  "const bodyRemoveChild = HTMLBodyElement.prototype.removeChild;",
  "HTMLBodyElement.prototype.appendChild = function (node) {",
  "  const skip = !(node instanceof HTMLElement) || node.classList.contains('cdk-overlay-container')",
  "    || node.classList.contains('cdk-visually-hidden') || node.dataset.onecxDynamicContainer !== undefined;",
  "  if (skip) return bodyAppendChild.call(this, node);",
  "  const wrapper = document.createElement('div');",
  "  wrapper.dataset[WRAPPER] = '';",
  "  const d = styleDataOf(node) ?? triggerStyleData();",
  "  if (d) setStyleData(wrapper, d, false);",
  "  clearStyleData(node);",
  "  wrapper.appendChild(node);",
  "  new MutationObserver((_, observer) => { if (!wrapper.childNodes.length) { wrapper.remove(); observer.disconnect(); } })",
  "    .observe(wrapper, { childList: true });",
  "  return bodyAppendChild.call(this, wrapper);",
  "};",
  "HTMLBodyElement.prototype.removeChild = function (node) {",
  "  let e = node;",
  "  while (e && e.dataset?.[WRAPPER] !== '' && e.parentElement) e = e.parentElement;",
  "  return bodyRemoveChild.call(this, e?.dataset?.[WRAPPER] === '' ? e : node);",
  "};",
  "",
  "document.createOnecxDynamicContainer = (tag, appElementName) => {",
  "  const c = document.createElement(tag);",
  "  const d = styleDataOf(document.getElementsByTagName(appElementName)[0]);",
  "  if (d) setStyleData(c, d, false);",
  "  c.dataset.onecxDynamicContainer = '';",
  "  document.body.appendChild(c);",
  "  return c;",
  "};",
  ""].join('\n');
  if (t.split(hooksOld).length !== 2) fail();
  t = t.replace(hooksOld, () => hooksNew);
  done.push('overlays and dialogs are scoped like the Shell');
}
fs.writeFileSync(rp, m);
fs.writeFileSync(tp, t);
console.log('patched the skill copy: ' + done.join('; '));
EOF
node "$WORK/patch-skill.cjs" "$WORK/run-test-instance"
```

The patch edits only the downloaded copy in `$WORK`, never the app repo. Record its output line in the report.

## 2. Run it

1. Build the app: `npm run build` (this repo uses the Angular CLI, not Nx, so ignore `nx build` in the skill).
2. Make sure Playwright and its Chromium exist. Run this every time; it installs only when the browser is
   missing, and the download takes several minutes, so wait for it to finish. Do not ask the developer. It
   installs into `~/.cache`, never into the repo. It also removes older Playwright browser builds in
   `~/.cache/ms-playwright`, so other tools on the machine that use an older Playwright will re-download theirs.

   ```bash
   set -euo pipefail
   TOOLS="$HOME/.cache/onecx-test/tools"
   if ! node -e "const {chromium}=require('$TOOLS/node_modules/playwright');process.exit(require('fs').existsSync(chromium.executablePath())?0:1)" 2>/dev/null; then
     echo "Installing Playwright + Chromium into $TOOLS (about 400-700 MB, several minutes)..."
     npm install --no-save --prefix "$TOOLS" playwright
     npx --prefix "$TOOLS" playwright install chromium
   fi
   node -e "const {chromium}=require('$TOOLS/node_modules/playwright');console.log('Chromium ready:',chromium.executablePath())"
   ```

   If the install fails (no network, no disk space), report `BLOCKED: <the error>` and stop the browser check.
   The runner's own message `Playwright is not installed` also appears when only the browser download is
   incomplete; running this step again fixes both.
3. `T init`. It must print `Test page: <file>` and the routes under `Other pages`. If it lists BFF operations,
   fill the `MOCKS` entries as `SKILL.md` step 3 says; with `BFF operations (0)` no mock data is needed.
4. `T serve` as a background process. Wait for `Test page: http://localhost:<port>/test`.
5. `T check /test/<route>` for each page that shows the migration work. Use the full app route, which includes
   the parent segment (for example `/test/primeng/datepicker`, `/test/primeng/select`, `/test/primeng/popover`).
   Pass `--click "<button text>"` to open an overlay first. Done when it prints `App loaded: yes` and
   `OK: no problems found`; otherwise fix the first entry under `PROBLEMS` using the table in `SKILL.md`.
6. Leave the server running until you have reported, then `T stop`.

Leave `THEME` as `init` set it unless the user asks otherwise.

## 3. What to tell the user (always include all four)

- **URL to open:** `http://localhost:<port>/test`, plus one URL per page you checked, such as
  `http://localhost:<port>/test/primeng/datepicker`. It works only while `T serve` is running. Opening the
  `test.html` file directly from disk does not work, because the page loads the app from the local server.
- **Test page file:** `~/.cache/onecx-test/<appId>/test.html` (the real path `init` printed).
- **Screenshot paths** printed by `check`, and what each shows.
- **Runner release:** the tag from `$WORK/runner-release.txt`, and `checksum OK`.

State plainly that this is a test page, not the real Shell. It publishes a theme and user, but does not
prove body-appended overlays, routing or federation (see the limits above). List the overlay pages
as still needing the real Shell. Include the patch result line from step 1b. Do not write PASSED for the Shell.

If you stop the server, say so and give the one command to start it again: `T serve`.
