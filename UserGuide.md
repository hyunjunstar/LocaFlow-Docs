# LocaFlow User Guide

LocaFlow is an editor tool for Unreal Engine 5.5 and 5.8 that finds what breaks **after** a game is translated,
and helps you fix it without leaving the editor:

- **Format QA** – missing or renamed `{Arguments}`, broken `|plural(...)` / `|gender(...)` / `|hpp(...)` syntax,
  mismatched rich text tags (`<b>…</>`, `<img/>`).
- **Korean particles** – finds `{Name}을(를)` style pairs and converts them to `{Name}|hpp(을,를)` in one click,
  so the engine picks the right particle at runtime.
- **Missing glyphs** – lists every character of a culture that your font cannot draw, and registers a system font
  (or any .ttf / .otf / .ttc) as a fallback sub-font.
- **Text overflow** – renders your Widget Blueprints in each culture, flags text that does not fit and saves a thumbnail.
- **Translation workflow** – completion per culture, inline editing, CSV / PO exchange with translators, and optional
  batch machine translation through DeepL or any OpenAI-compatible endpoint, with a cost estimate and a per-job cap.

LocaFlow works directly on the standard files written by the **Localization Dashboard**
(`Content/Localization/<Target>/...`). It never changes your gather settings, and the Localization Dashboard keeps
working as before.

---

## 1. Requirements

| | |
|---|---|
| Engine | Unreal Engine 5.5 and 5.8 |
| Platform | Windows 64-bit (Win64) only, for the editor and for packaged games. |
| Project | Any project (Blueprint or C++) with at least one localization target that has been **gathered** once. |
| Network | Only for machine translation (optional). All QA features work offline. |

The editor modules (`LocaFlowCore`, `LocaFlowQA`, `LocaFlowTranslate`, `LocaFlowEditor`) are never packaged into your
game. The small `LocaFlowRuntime` module adds four optional Blueprint nodes for an in-game language menu (section 10).

## 2. Installation

1. Install LocaFlow from Fab to your engine (Fab / Epic Games Launcher → *Install to Engine*).
2. Open your project, go to **Edit → Plugins**, search for **LocaFlow**, enable it and restart the editor.
3. Open **Tools → LocaFlow**.

## 3. Prepare a localization target (once)

LocaFlow reads the data the Localization Dashboard produces. If you have never localized the project:

1. **Tools → Localization Dashboard**.
2. Use the default *Game* target (or add one). Under *Gather Text* enable *Gather from Text Files* and/or
   *Gather from Packages* and point them at your Source / Content folders.
3. Add the cultures you want (e.g. *ko*, *ja*, *zh-Hans*, *de*) and choose the native culture (usually *en*).
4. Press **Gather Text**. This creates `Content/Localization/Game/Game.manifest` and one `Game.archive` per culture.

After that, LocaFlow lists the target and its cultures. Whenever you add new text to the game, press *Gather Text*
again, then **Reload** in LocaFlow.

## 4. The dashboard

| Area | What it does |
|---|---|
| **Target / Culture** | Pick the localization target and the culture to work on. |
| **Reload** | Re-read targets and the current culture from disk (asks to save unsaved edits first). |
| **Save** | Write your edits back into the culture's `.archive`. A `.bak` copy of the archive is made once per session before the first save. |
| **Run All QA** | Format, rich text, Korean particle (ko cultures), empty and stale checks for the current culture. |
| **Translate Missing** | Machine-translate entries that are *Missing* or *Stale* (section 8). |
| **Export / Import CSV, PO** | Exchange translations with translators (section 7). |
| **Completion bars** | Percentage of translated entries for every culture of the target. |
| **Filter** | Search by key, source or translation; filter by state (*Translated, Missing, Stale, Same as source, Has issues, Human edited*). |
| **Entry list** | Key, source, translation (click to edit), state and issue count. Your edits are marked *(human)*. |
| **QA Issues / Font Wizard / Overflow** | The three tool tabs at the bottom. |

`* Unsaved changes` appears in the toolbar while you have edits that are not written to disk yet.

### Entry states

| State | Meaning |
|---|---|
| Translated | Has a translation. |
| Missing | No translation yet. |
| Stale | The source text changed after the translation was written. Unreal ignores such a translation when it compiles the localization, so the game shows the source text until the entry is translated or approved again. |
| Same as source | The translation is identical to the source in a non-native culture (often untranslated). |

## 5. QA checks

Press **Run All QA**. The **QA Issues** tab groups findings by type. Click an issue to select its entry; where a fix
is known, **Apply fix** replaces the translation with the suggestion.

| Check | Severity | Example |
|---|---|---|
| Format arguments | Error | Source `Welcome, {Name}!` → translation `환영합니다!` (`{Name}` lost) |
| Plural / gender / hpp syntax | Error | `{Count}|plural(one=item,other=items` (missing `)`) |
| Rich text tags | Error | Source `Press <b>Start</> to begin.` → translation without `<b>` |
| Korean particles | Info, with fix | `{Name}을(를) 초대했습니다.` → `{Name}|hpp(을,를) 초대했습니다.` |
| Empty translations | Warning | Missing entries |
| Stale translations | Warning | Source changed since translation |

Notes:

- An argument that only appears inside a `|plural(...)` block counts as present, so translations may drop the
  `{Count} {Count}|plural(...)` pattern and write `{Count}개` instead.
- Escaped braces use the Unreal convention (a backtick before `{` or `}`).
- **Convert all 을(를) to hpp** (QA Issues tab, ko cultures) converts every particle pair at once. Supported pairs:
  을(를), 이(가), 은(는), 와(과), 으로(로), 이라(라) – when they directly follow an `{Argument}`.

## 6. Fonts: missing glyphs and fallback fonts

Open the **Font Wizard** tab.

1. Choose your **Font** asset (a *Runtime* composite font – the default for fonts created in UE5). You can set a
   default in *Editor Preferences → Plugins → LocaFlow → Reference Font*.
2. Press **Check glyphs**. LocaFlow tests every character used by the current culture's translations against the
   font's default typeface, fallback typeface and sub-fonts (respecting their character ranges) and lists the
   characters that cannot be drawn.
3. To fix it, either press **Use system font** (Malgun Gothic / Yu Gothic / Microsoft YaHei or
   JhengHei) or **Browse...** to any .ttf / .otf / .ttc file, then
   **Register fallback**. The font file is imported as a Font Face asset under `/Game/LocaFlow/Fonts` and added to your
   font as a sub-font that covers the culture's script (Hangul, Kana + Kanji, or Hanzi).
4. Save the font asset (it is marked dirty) and check again – the list should be empty.

> **Licensing:** system fonts are licensed for use on your computer, not necessarily for redistribution inside a game.
> Before shipping, register a font you are allowed to ship (for example Noto Sans CJK, which is under the SIL Open Font
> License).

Fonts using the *Offline* cache type (bitmap fonts) cannot be checked.

## 7. Working with translators: CSV and PO

- **Export CSV** writes `Namespace,Key,Source,Translation,Comment` (UTF-8 with BOM, opens correctly in Excel).
- **Export PO** writes a GNU gettext file (`msgctxt` = `Namespace,Key`) for tools such as Poedit, Crowdin or Weblate.
- **Import CSV / Import PO** merges translations back by namespace and key. Imported entries are marked as human
  edited, so machine translation never overwrites them. Press **Save** afterwards.
  Rows whose *Source* column differs from the current source were translated from an older text; they are skipped
  and reported, so an outdated translation is never approved by accident. Rows without a source are applied.

## 8. Machine translation (optional)

### Setup

Open **Edit → Editor Preferences → Plugins → LocaFlow**:

| Setting | Description |
|---|---|
| Provider | *DeepL* or *OpenAI-compatible*. |
| DeepL API Key | From your DeepL account. Free keys end with `:fx` and use the free endpoint automatically. |
| OpenAI Base URL / API Key / Model | Any OpenAI-compatible Chat Completions endpoint (OpenAI, Azure OpenAI proxy, a local server...). Default `https://api.openai.com/v1`, model `gpt-4o-mini`. |
| Price Per Million Chars | Used for the cost estimate (DeepL Pro is about 25 USD). |
| Max Cost Per Job USD | Jobs whose estimate is above this cap do not start. 0 = no cap. |
| Glossary CSV | Optional `source,target` file (one pair per line, optional header, `#` comments). |
| Batch Size / Max Requests Per Second | Request size and rate limit. |

These settings are stored per user in `Saved/Config/.../EditorPerProjectUserSettings.ini`, which is normally not
checked in, so API keys do not end up in source control.

### Running a job

1. Select a culture other than the native one.
2. Press **Translate Missing**. LocaFlow shows how many entries and characters will be sent and the estimated cost.
3. Confirm. Progress appears in the toolbar; **Cancel** stops the job (finished entries stay).
4. Review the results, run QA, and press **Save**.

What is protected:

- Only *Missing* and *Stale* entries are sent. Entries you edited, imported or approved (*human*) are never overwritten.
- `{Arguments}`, plural / gender / hpp blocks and rich text tags are replaced by tokens before sending and restored
  afterwards. A result that loses or changes a token is discarded and counted as failed.
- Failed requests are retried (rate limits and server errors, with back-off). An invalid key stops the job with a
  clear message.

**Privacy:** the source text of the selected entries is sent to the provider you configure. Nothing is sent unless you
press *Translate Missing* and confirm.

**Review the results:** machine translations are drafts. LocaFlow protects arguments, plural / gender / hpp blocks and
rich text tags, but it does not judge wording or tone, so have a person review them and run QA before you ship.

**Verification status:** translation through DeepL was tested against the live DeepL API on UE 5.5 and 5.8
(translation, placeholder and tag preservation, save and reload, invalid key, cancel). For OpenAI-compatible
endpoints, the request flow, error handling (invalid key, no credit) and cancel were checked against the live OpenAI
API, but a successful live translation through an OpenAI-compatible service has not been verified yet.

## 9. Text overflow

Open the **Overflow** tab.

1. **Add Widget Blueprint** for each screen you want to check (menus, buttons, HUD...).
2. Tick the cultures to test.
3. Press **Run overflow check**.

For every culture LocaFlow switches the editor's *game localization preview* to that culture, creates the widget,
lays it out and compares the size each Text Block / Rich Text Block needs with the size it gets. Findings show the
culture, widget and text, with a thumbnail of the rendered widget (saved under `Saved/LocaFlow/OverflowThumbs`). The editor's own
UI language is not changed, and the previous preview language is restored afterwards.

Requirements: the culture must be **compiled** (Localization Dashboard → *Compile Text*) so the translated text is
available, and the check needs the normal editor renderer (it is skipped with `-nullrhi`).

## 10. Runtime Blueprint nodes (optional)

Category **LocaFlow**, usable in any Win64 game:

| Node | Description |
|---|---|
| Get Available Languages | Cultures that have localization data packaged with the game. |
| Get Language Native Name | e.g. "한국어" for `ko`, "日本語" for `ja` – for a language menu. |
| Get Current Language | Current culture code. |
| Set Language | Switches the language; with *Save* it is remembered in GameUserSettings.ini. |

## 11. Tips and known limitations

- After saving in LocaFlow, run **Compile Text** in the Localization Dashboard (or package the game) to build the
  `.locres` files the game uses.
- Running **Gather Text** again keeps all translations. LocaFlow keeps its *human edited* markers in
  `<Culture>/<Target>.locaflow.json` next to the archive (commit it with the archive); a marker is dropped when the
  translation is changed outside LocaFlow.
- When the source text of an entry changes, the engine's gather creates a new entry; it shows as *Missing* and the
  old translation is dropped by the engine, as in the Localization Dashboard.
- Bitmap fonts (*Offline* cache type) are not supported by the glyph check.

## 12. Troubleshooting

| Problem | Solution |
|---|---|
| Target list is empty | Gather the target once in the Localization Dashboard, then press **Reload**. |
| "API key is missing" | Enter the key in *Editor Preferences → Plugins → LocaFlow*. |
| "Estimated cost exceeds the cap" | Raise *Max Cost Per Job USD* or translate fewer entries at a time. |
| Overflow check shows only the native text | Compile the localization (Localization Dashboard → *Compile Text*) and run again. |
| Glyph check: "uses the Offline cache type" | Use a Runtime (composite) font asset. |
| Need help | Email khj476611@naver.com. Please include the engine version and the Output Log lines starting with `LogLocaFlow`. |
