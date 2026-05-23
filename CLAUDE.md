# CLAUDE.md - Strict Build Rules

## Mandatory Compilation Standards
* **Zero Unused Symbols / Dead Branches:** Every import, variable, and function MUST be used. No unreachable branches, unused state setters, or orphaned context-menu entries. The project uses `noUnusedLocals` and `noUnusedParameters`. If you remove a component, you MUST delete its import (especially from `lucide-react`).
* **No Dead Exports / No Unnecessary Exports:** TypeScript flags unused symbols only *within* a file — it does NOT detect cross-file dead code. Every exported function, type, constant, and public class method MUST have at least one consumer in another file. If nothing imports it, either delete it or remove the `export` keyword (downgrade to module-private). The same rule applies to public class methods: defined-but-never-called methods are dead code and must be removed. Before adding `export` to a new symbol, confirm an external caller exists or is being added in the same change.
* **No Implicit 'any':** All types must be explicitly defined. Use interfaces or types for all props and state.
* **Safe Access Only:** Use optional chaining (`?.`) or null checks for ALL object properties, API data, and array elements. Assume everything can be `undefined`.
* **React 19 Cleanliness:** Use modern functional components. Do not import `React` unless using hooks (`useState`, `useEffect`).

## React Hooks Discipline
* **No setState-in-Effect for Derived State:** If a value is a pure function of props/state, derive it during render (`const x = ...`) or via `useMemo`. Never write `useEffect(() => setX(computed))`. The `react-hooks/set-state-in-effect` escape is reserved for true cleanup (e.g. pruning a Set), and must be guarded against re-fire with a size/equality check plus an inline comment explaining the guard.
* **Exhaustive Deps, No Suppression:** Every non-stable value referenced inside `useEffect` / `useCallback` / `useMemo` MUST appear in its dep array. Stabilize helpers with `useCallback`, derived collections with `useMemo`. Do not add `// eslint-disable-next-line react-hooks/exhaustive-deps` — fix the root cause instead.
* **Refs Mirror Inside Effects:** `ref.current = value` must live in a `useEffect` keyed on `value`, not the render body. Writing a ref during render fires on every render and conflicts with StrictMode.
* **Underscore for Unused:** Intentionally unused arguments, locals, and caught errors MUST be prefixed with `_` (per `eslint.config.js`). Any unprefixed unused symbol is a lint error.

## Comment Style
* **Comments Are Optional — Focus on WHY:** Don't explain what the code does; well-named identifiers already do that. Only add a comment when the reason is non-obvious — a hidden constraint, a workaround, a subtle invariant, surprising behavior. If removing a comment wouldn't confuse a future reader, don't write it.
* **English Only — No Mixed-Language Comments:** All comments (JSDoc, inline `//`, block `/* */`, file-level docs) MUST be in English. The codebase is consistently English; Hungarian fragments break that consistency. User-facing strings (UI text, error messages shown to players) remain in Hungarian — this rule is for source code comments only.

## Hungarian UI Text Style
User-facing strings are in Hungarian; source-code comments stay English (see "Comment Style" above). When you create or modify any user-visible Hungarian text, match these conventions so the UI stays consistent in tone and form.

### Address & voice
* **Tegezés only.** Never use magázás (`Ön`, `Önnek`, `Kérjük`). Imperatives use second-person singular: `Adj nevet…`, `Próbáld újra.`, `Törlöd…?`.
* **Possessive for player-owned game state.** Use the `-od/-ed/-ad/-d` suffix whenever the meaning is "yours": `A jelenlegi játékállásod törlődik`, `a paklid`, `a kártyáid`. Do not use detached compound nouns (`a játékállás`, `a pakli`) for things the player owns.

### Confirm dialogs (`ConfirmDialog`)
* **Title** = action-noun phrase (`-ás/-és`), no terminal punctuation.
    * Yes: `Pakli törlése`, `Kilépés a szobából`, `Játék újraindítása`.
    * No: `Biztosan törlöd?`, `Pakli törlése?`, `Pakli törlése.`.
* **Description** = either a te-form question ending in `?` or a declarative statement ending in `.`. Drop redundant `Biztosan` openers.
    * `Törlöd a "${name}" paklit a mentett paklik közül?`
    * `A jelenlegi játékállásod törlődik, és újraosztódnak a kártyáid.`
* **Confirm button** = action-noun matching the title: `Törlés`, `Kilépés`, `Eltávolítás`, `Újraindítás`. Confirm-button labels are the **only** action affordance where the action-noun form is correct — they mirror the dialog title.
* **Cancel button** = `Mégsem` (do not use `Mégse`, `Vissza`, `Nem`).

### In-context single-step actions on a specific item
This covers menu items in `CardContextMenu` and `title=` tooltips on icon buttons that act on one entity (delete, rename, remove, move, close, increment, decrement, draw, shuffle, search, reveal, hide).
* **Imperative form** (T-form / 3rd-person-singular indicative — i.e. the bare verb stem), no punctuation: `Húz`, `Kever`, `Elforgat`, `Elrejt`, `Felfed`, `Keres`, `Kivesz`, `Megtekint`, `Eltávolít`, `Átnevez`, `Áthelyez`, `Töröl`, `Szerkeszt`, `Bezár`, `Csökkent`, `Növel`.
* **Do not use the action-noun form** (`-ás/-és`) for these — that form is reserved for standalone primary buttons and confirm-dialog buttons.
* Within one menu, never mix forms.
* **Use `title=` only — do not add `aria-label=`** for these labels. The codebase convention is `title=`; `title` already feeds into the screen-reader accessible-name calculation, and a single source of truth keeps icon-button labelling consistent.
* **The same principle applies project-wide — do not introduce `aria-*` attributes elsewhere either.** Visible UI text (`<label>` for form inputs, `<h2>` for section titles, element content for buttons) is the single source of truth for accessible names. The two existing exceptions (`src/components/legal/PrivacyPolicy.tsx` `aria-label` on a Google-Translate host span; `src/components/DeckBuilder/StableLabel.tsx` `aria-hidden` on an invisibly-duplicated layout helper) have specific technical reasons — match an equivalent reason before adding new `aria-*` usage.
* **Exception — state toggles / channel state are noun-phrase descriptors, not actions, and stay as-is:** `Némítás`, `Némítás feloldása`, `Belépés a hangcsatornába`, `Kilépés a hangcsatornából`, `Aktív hangkapcsolat folyamatban`. These describe what is or what will be, not a single-step action.

### Standalone buttons (primary CTAs and full-width actions)
* **Action-noun form** (`-ás/-és`): `Mentés`, `Mentés újként`, `Felülírás`, `Betöltés`, `Csatlakozás`, `Kilépés`, `Visszaállítás`, `Belépés`.

### Form labels & placeholders
* **Labels** = noun phrase, no trailing punctuation: `Szoba neve`.
* **Placeholders** end with a single ellipsis character `…` (U+2026), never three dots `...`: `Keresés…`, `Pakli neve…`, `Pakli link vagy deckbuilderID…`.
* **Example values** prefix `pl. ` (lowercase, period, space): `pl. titkosteszt`.

### Toasts, errors & validation messages
* **Full sentence ending in `.`** Every error/status string is a complete clause terminated by a period: `A név nem lehet üres.`, `Maximum ${MAX_DECKS} pakli menthető.`, `A saját pakli sérült.`.
* **Compound recovery** = problem statement + imperative, each as its own clause with its own period: `Hiba a mentés során. Próbáld újra.`.
* **No `!` in errors.** Reserve `!` for upbeat call-to-action tooltips (`Aktív hívás – kattints a csatlakozáshoz!`).
* **In-progress status** ends with `…`, no period: `Újrakapcsolódás…`, `Kártyaadatbázis betöltése…`.

### Empty states
* **Always end with a period.** Whether the message is a short status (`Nincs paklid.`, `Nincs találat.`, `A Cserepakli üres.`) or a longer instructional sentence (`Importálj deckbuilder-linkkel, vagy állítsd össze kártyánként a keresővel és mentsd el, hogy később könnyen betölthesd.`), terminate with `.`. The `…` (U+2026) ending is reserved for **In-progress status** (see below) — not for empty states.

### Tooltips (descriptive, not action affordances)
* Short noun phrase or short imperative, **no period**: `Teljes képernyő`, `+1 perc`, `Dobj 2d6-tal`, `Visszaállít (45:00)`.
* For state + action pairs, separate with a spaced en-dash (` – `): `Szoba nyitott – kattints a záráshoz`.

### Headers & section titles
* Noun phrase, no punctuation: `Szoba`, `Játékosok`, `Saját paklik`, `Pakli importálása`.

### Brand name
* The product is `M.A.G.U.S. Kártya VTT` — keep the dots in the acronym. Do not abbreviate to `MK VTT` in any user-visible text or documentation.

### When unsure
* Read 2–3 nearby strings serving the same UI role (`Grep` for similar dialogs, buttons, placeholders) and match their pattern. The existing UI is the source of truth.
* If a new string genuinely needs a pattern not covered above, **stop and clarify with the developer first** — do not silently deviate, and do not invent a new convention on the fly.
* If the developer's request itself conflicts with these rules, surface the conflict and confirm the direction before proceeding — do not silently override the rules even when asked.

## Typography Tokens
The text typography scale is reduced to **5 tokens** — every text element MUST use one of these and only these:

* `text-xs` (12px) — small labels, metadata, badges, kbd hints, status indicators
* `text-sm` (14px) — default body, menu items, button text
* `text-base` (16px) — modal titles, emphasized body, form section headings
* `text-lg` (18px) — section headings, legal h2
* `text-2xl` (24px) — page titles only (lobby, legal, auth screens)

**Forbidden for text:**
* Other Tailwind text-size classes: `text-xl`, `text-3xl`, … `text-9xl`
* Arbitrary values: `text-[Npx]`, `text-[Nrem]`, etc.
* Inline `style={{ fontSize: ... }}` for textual content

**Single documented exception — GameBoard functional numerals.** Pixel-pegged for layout reasons (MP main number, card markers on the board, zone badges); stay as inline `style={{ fontSize: N }}`. These files are the *only* place inline `fontSize` is allowed:
* `src/components/GameBoard/MpCounter.tsx`
* `src/components/GameBoard/CardToken.tsx`
* `src/components/GameBoard/ZoneArea.tsx`
* `src/components/GameBoard/ZoneActivityOverlay.tsx`

If a new role genuinely needs a size outside the 4-token scale, surface the conflict and clarify with the developer before introducing a new token — do not silently add `text-[Npx]`.

## Execution Protocol
1.  **Mental Build:** Before providing code, simulate `npm run build` (tsc + vite).
2.  **Cleanup First:** If a change makes an import redundant, remove it in the same response.
3.  **Strict Mode:** Treat all TypeScript warnings as fatal errors.
4.  **Build Must Pass:** After every set of changes, run `npm run build` and fix ALL errors — including pre-existing ones in files you didn't touch. The project must always be in a buildable state.
5.  **Tests Must Pass:** After changes that touch tested code (see `src/**/*.test.ts`), run `npm test` and resolve any failing tests in the same response.
6.  **Lint Must Pass:** After every set of changes, run `npm run lint` and fix ALL errors and warnings — including pre-existing ones in files you didn't touch. The project must always be lint-clean.
7.  **Comment Audit:** Before declaring a change complete, scan every new or modified comment and verify it is in English. Replace any Hungarian fragments with English equivalents — no exceptions, no mixing.
8.  **Tracking Audit:** When adding a user-facing feature or extending one, consider whether `src/services/tracker.ts` needs a new event or an existing event extended (new field / new source variant). Skip only with an explicit reason — don't silently leave new behaviour untracked.
9.  **Documentation Audit:** When you add, remove, or rename any user-facing Hungarian string (button label, menu item, section title, dialog title/description, placeholder, tooltip), grep `src/content/felhasznaloi-kezikonyv.md` and `src/components/legal/*.tsx` for the old label and any prose describing the affected behaviour. Update the docs in the same change. The user manual and legal pages must describe the live UI exactly — drift is a bug.
10. **Firebase Rules Audit:** When you add, remove, or rename fields on records written by `src/services/FirebaseService.ts` (paths under `rooms/`, `user-presence/`, `lobby-board/messages/`), or change the path structure of any of these, update `database.rules.json` in the same change. The committed file is the source of truth for what should be deployed; drift between the file and the deployed rules silently lets invalid writes through or blocks valid ones, and is only caught at runtime (or not at all).
11. **PostHog Followups Audit:** When you ship a fix that was identified via PostHog telemetry AND the fix is expected to change the rate or presence of tracked events (anomaly counts, error rates, funnel-step counts, drop-off ratios, etc.), append a structured entry to `FOLLOWUPS.md` in the same change. Use the template at the top of that file. Every entry MUST include: how the issue was first surfaced (the original PostHog signal + scope), the hypothesis, the fix (file:line), the fix date, a verify-by date (default `fix_date + 7 days`; widen if the affected event is low-volume), the exact PostHog query to re-run, and an expected-outcome that includes a sanity-check condition so an empty result cannot be confused with "no traffic". The point: a future session must be able to pick the file up cold and decide "fixed" or "still broken" from observable data alone. Do not invoke this rule for code changes the user did not request to log as a followup, nor for changes whose effect is invisible to telemetry (pure refactors, UI tweaks with no tracked event).

## Communication & Commit Style
* **Concise Feedback:** Provide brief, fact-focused responses. No software metaphors. No fluff (e.g., avoid "excellent", "super").
* **Automated Commit Suggestion:** At the end of every response involving code changes, always suggest a concise commit message in English.
* **Commit Format:** Use the Conventional Commits standard (`type: description`).
    * **Language:** **English only — no exceptions.** Even when the entire conversation is in Hungarian, the suggested commit message MUST be in English. Do not translate user-facing Hungarian strings; describe the change in English prose.
    * **Types:** `feat` (new feature), `fix` (bug fix), `chore` (maintenance/workflow), `refactor` (code cleanup), `docs` (documentation).
    * **Style:** Lowercase, no period at the end, concise but descriptive.
    * **Example:** `chore: update PR promotion workflow to use gh cli`
