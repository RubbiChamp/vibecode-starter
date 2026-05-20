---
description: Starter et nyt vibe coding-projekt. Stiller 3 spørgsmål og opretter projekt-fundamentet med CLAUDE.md, arkitektur-skitse og lokale regelfiler.
---

Du skal nu køre `/start-project`-flowet. Følg disse instruktioner præcist. Hele flowet er på dansk.

## 1. Velkomstbesked

Send følgende besked (eller en meget tæt parafrase):

> Lad os bygge fundamentet til dit projekt ordentligt.
>
> Det her tager 5-10 minutter og sparer dig timevis af bøvl senere. Jeg stiller dig **3 spørgsmål** om hvad du vil bygge. Svar bare så detaljeret du kan — jo mere kontekst jeg får, jo bedre kan jeg hjælpe dig hele vejen igennem projektet.
>
> Bagefter genererer jeg:
> - En `CLAUDE.md` med projekt-konteksten
> - En `docs/ARCHITECTURE.md` med datamodel og sikkerhed
> - Tre regelfiler i `docs/regler/` om sikkerhed, skalering og struktur
> - Standard projekt-filer (`.gitignore`, `.env.local.example` osv.)
>
> Klar til at starte? Skriv bare "ja" eller "klar", så går vi i gang.

Vent på brugerens bekræftelse før du fortsætter.

## 2. Stil de 3 spørgsmål — ét ad gangen

VIGTIGT: Stil ét spørgsmål ad gangen. Vent på svar mellem hvert. Vis eksempler så det er nemt at svare.

### Spørgsmål 1

> **Fortæl mig om din idé — bare frit fra leveren.**
>
> Hvad er det for en app eller et website du gerne vil bygge? Hvilket problem løser det, hvem er det til, og hvad skal man kunne i appen?
>
> Skriv som om du forklarer det til en ven over kaffe. Jo mere du skriver, jo bedre fundament får vi — gerne 5-10 sætninger. Du må også gerne nævne hvilke 3-5 features der er must-have for første version.

### Spørgsmål 2

> **Skal folk kunne logge ind, og er der forskellige slags brugere?**
>
> Vælg det der minder mest om din app:
>
> - "Nej, alle ser bare det samme — ingen login"
> - "Ja, alle brugere er ens — de logger ind for at se deres egne ting"
> - "Ja, der er fx admin og almindelige brugere — admin kan mere"
> - "Det er multi-tenant — fx firmaer med hver deres data adskilt"
>
> Der er ingen forkerte svar — vælg det der ligger tættest på.

### Spørgsmål 3

> **Hvad er de vigtigste ting din app gemmer — og er noget af det privat?**
>
> Tænk på det som lister du gemmer i appen. Fx "opskrifter, kommentarer, brugerprofiler". Eller "kundekartotek, fakturaer, kvitteringer".
>
> Og er noget af det følsomt? Fx CPR-numre, helbredsdata, betalingsinfo eller børn-data? Eller er det "bare almindelige ting"?

Notér svarene undervejs.

## 3. Generér filerne

Efter alle 3 svar, opret filerne herunder i projektets root (cwd). Brug `Write`-tool'et til hver fil.

### `CLAUDE.md`

Generér ud fra svarene. Hold den under ~100 linjer. Brug denne struktur:

```markdown
# Projekt: [navn udledt af svar 1]

## Hvad bygger vi
[Svar 1 — gengiv som 3-6 sætninger der opsummerer idé, problem, brugere og features]

## Brugere og auth
[Svar 2 — kort om hvem brugerne er og hvilken auth-model der gælder]

## Stack
- Next.js (App Router) + TypeScript + Tailwind
- Supabase (Postgres + Auth + RLS)
- Hostet på Vercel

Bemærk: Dette projekt er bygget med vibecode-starter, som er kalibreret til præcis denne stack. Hvis du senere skifter database eller framework, så opdater eller fjern regelfilerne i `docs/regler/` så du ikke får vildledende råd.

## Vigtige ting i systemet
[Svar 3 — listen af "ting" som korte bullets, samt om noget er følsomt og hvad det betyder]

## Regelsæt — læs disse før du skriver kode
Dette projekt bruger lokale regelfiler. Inden du genererer eller ændrer kode, læs:
- `docs/regler/sikkerhed.md`
- `docs/regler/skalering.md`
- `docs/regler/struktur.md`

Reglerne er kalibreret til Next.js + Supabase. De er ikke firkantede absolutter — de er gennemtænkte standarder. Afvig hvor det giver mening, men nævn det.

## Konventioner
- Alle mutations sker i server actions med Zod-validering
- Alle Supabase-tabeller har RLS aktiveret fra dag ét
- Læs `docs/ARCHITECTURE.md` før større ændringer

## Verifikation
- `pnpm typecheck`
- `pnpm test` (når der er tests)
- `pnpm build`
```

### `docs/ARCHITECTURE.md`

Skitsér på baggrund af svarene:

- **Datamodel**: Én sektion per "ting" fra svar 3. For hver: foreslåede kolonner (med typer), foreign keys, og hvem der ejer rækken (drives af svar 2).
- **RLS-strategi**: Hvilke policies hver tabel skal have (SELECT, INSERT, UPDATE, DELETE). Eksplicit hvem der må hvad.
- **Server actions vs route handlers**: Hvor mutations sker. Hvor evt. webhooks lander.
- **Public surface**: Hvilke routes er åbne uden login, hvilke kræver auth.
- **Privacy-konsekvenser**: Hvis svar 3 indeholdt følsomme data, så hvad det betyder for opbevaring, logging og audit.

### `docs/PROGRESS.md`

Brug faktisk dato i format `YYYY-MM-DD`. Hvis du ikke kender dagens dato, så spørg brugeren, eller udled fra system-kontekst.

```markdown
# Progress log

- YYYY-MM-DD Projekt startet med `/start-project`
```

### `docs/regler/sikkerhed.md`

Skriv NØJAGTIGT følgende indhold:

````markdown
# Sikkerhedsregler

Disse regler gælder for hele dette projekt. Når du genererer eller ændrer kode, så følg dem. De er destilleret fra fælles fælder vibe-coders falder i på Next.js + Supabase.

## Mentor-mandat

Du er mentor for en bruger der ikke nødvendigvis kender hver teknisk detalje. Når du implementerer noget sikkerhedsrelateret (rate limiting, env vars, RLS, auth), så **forklar i 3-4 nummererede trin hvad brugeren skal gøre udenfor koden** — fx tilføj env vars i Vercel, opret en konto hos en service, klik en bestemt knap. Forklar HVORFOR i én linje. Vær konkret, ikke abstrakt.

## RLS og Supabase

- Enable RLS på ALLE tabeller fra dag ét, FØR første query køres
- Skriv eksplicitte policies for hver operation: SELECT, INSERT, UPDATE, DELETE
- Brug `auth.uid()` i policies — ikke kun `auth.role()`
- ALDRIG `service_role`-nøgle i `NEXT_PUBLIC_`-prefiks eller client-side kode
- `anon`-nøglen client-side; `service_role` kun i server actions og kun når strengt nødvendigt
- Tjek altid ownership server-side selvom RLS er aktiv (defense in depth)

## Secrets-håndtering

- ALDRIG hardcoded API-keys i kildekode. Patterns at afvise: `sk-`, `sk_live_`, `AKIA`, `ghp_`, `sk-ant-`, `xoxb-`
- ALDRIG `NEXT_PUBLIC_`-prefiks på secrets — det eksponeres i browser-bundlet
- `.env.local` SKAL være i `.gitignore`
- `.env.local.example` committes med kun pladsholdere
- Brug `process.env.X` server-side og verificér eksistens ved opstart

## Server actions og database-adgang

- ALDRIG direkte Supabase-client-mutationer fra client components — wrap alt i server actions eller route handlers
- Kald `auth.getUser()` (IKKE `getSession()`) server-side før mutationer — `getSession()` validerer ikke JWT'en
- Tjek ownership (`user_id === user.id`) på alle reads og writes — forhindrer IDOR
- Validér ALLE inputs med Zod FØR de når databasen

## Input-validering og output

- Zod-schemas for alle form-inputs, API-bodies, search-params, route params
- Client-side validering er UX, ikke sikkerhed — gentag alt server-side
- ALDRIG returnér rå database-fejl til client — log server-side, send generisk besked
- ALDRIG `dangerouslySetInnerHTML` uden DOMPurify
- ALDRIG `eval()` eller `new Function()` med user input

## Auth og cookies

- Cookies SKAL have: `httpOnly: true`, `secure: true` i production, `sameSite: 'lax'` eller `'strict'`
- ALDRIG `algorithm: "none"` i JWT
- Altid `expiresIn` på JWT'er
- ALDRIG `jwt.decode()` uden `jwt.verify()` først

## Rate limiting — det vigtigste at få ret

Implementér rate limiting på endpoints der kan koste penge eller misbruges. Konkrete tærskler:

| Endpoint-type | Rate limit fra start? |
|---|---|
| Login, signup, password reset | **Supabase Auth har det indbygget** — tjek det er aktivt i Supabase Dashboard |
| AI-endpoints (OpenAI, Anthropic) | **Ja, altid** — 10 calls per bruger per time, 20 per IP per time |
| Email/SMS-endpoints (Resend, Twilio) | **Ja, altid** — 5 per bruger per time |
| File uploads | **Ja** — 5 uploads per bruger per time, max 5MB per fil |
| Almindelige form submissions | Vent til lige før launch |
| Search og list-endpoints | Vent til lige før launch |

**Anbefalet service:** Upstash Ratelimit (gratis tier dækker MVP'er). Vis brugeren begge opsætnings-veje:

1. **Hurtigst (Vercel-brugere):** Vercel → projekt → Storage → "Add Upstash". Env vars sættes automatisk.
2. **Mest fleksibelt:** Opret konto på upstash.com → lav Redis-database → kopier `UPSTASH_REDIS_REST_URL` og `UPSTASH_REDIS_REST_TOKEN` til `.env.local`.

Body-size limit på ALLE endpoints der modtager input: 1MB default.

## Headers, CORS og injection

- CORS: konkret origin, ALDRIG `*` i production
- Sæt CSP, X-Frame-Options (DENY), Strict-Transport-Security i `next.config.js`
- Brug Supabase-client / Drizzle ORM — ALDRIG template literals i raw SQL
- Path traversal: validér file paths og brug `path.resolve()` + prefiks-tjek før læsning

## File uploads

- Whitelist MIME-types — aldrig blacklist
- Tjek faktisk fil-indhold, ikke kun extension
- Max-size limit på alle uploads
- Gem ALDRIG uploads under web-root med eksekverbar adgang

## GØR / IKKE GØR

```ts
// GØR
const { data: { user } } = await supabase.auth.getUser();
if (!user) return { error: 'unauthorized' };
const parsed = TilbudSchema.parse(input);

// IKKE GØR
const { data: { session } } = await supabase.auth.getSession(); // ikke verificeret
const tilbud = await supabase.from('tilbud').insert(input);     // ingen validering, ingen ownership-check
```
````

### `docs/regler/skalering.md`

Skriv NØJAGTIGT følgende indhold:

````markdown
# Skaleringsregler

Pragmatiske regler. Vi forhindrer katastrofer, ikke premature optimization. MVP'er skal kunne flyve uden bagage.

## Den vigtigste regel: eksplicit limit + pagination når listen kan vokse

Alle Supabase-queries der returnerer arrays SKAL have et eksplicit `.limit()`. Men `.limit()` alene skaber en **silent bug** hvis listen kan vokse forbi din limit — brugeren ser 20 ud af 125 tilbud og opdager aldrig at de mangler resten.

Derfor: limit og pagination hører sammen. Beslut én af de tre kategorier før du skriver query'en:

### Kategori A — Liste der kan vokse (95% af tilfældene)

Opskrifter, tilbud, ordrer, kommentarer, posts, opgaver, billeder. Alt brugergeneret eller alt der vokser over tid.

→ **`.limit(20-50)` PLUS pagination eller "indlæs flere" fra start.**

```ts
// GØR — limit + pagination via .range() fra dag ét
const PAGE_SIZE = 20;
const { data, count } = await supabase
  .from('tilbud')
  .select('id, titel, pris', { count: 'exact' })
  .order('created_at', { ascending: false })
  .range(page * PAGE_SIZE, (page + 1) * PAGE_SIZE - 1);

// Eller "indlæs flere"-knap der øger limit
const { data } = await supabase
  .from('tilbud')
  .select('id, titel, pris')
  .order('created_at', { ascending: false })
  .limit(visibleCount);
```

### Kategori B — Liste med naturligt fast loft

Lande-dropdown (~250), statusser (~5), kategorier (~10-30), månedsvalg (12), brugerens egne indstillinger.

→ **`.limit(N)` hvor N dækker alle realistiske entries med margin. Ingen pagination nødvendig.**

```ts
// Lande — der findes ~250 i verden, sæt limit der dækker
const { data } = await supabase.from('lande').select('kode, navn').limit(300);
```

### Kategori C — Detail-views

Hentet via `.single()` eller `.maybeSingle()` med en `.eq('id', ...)`-filter.

→ **Ingen limit nødvendig — der kommer kun én række tilbage per design.**

## Når du er i tvivl

Spørg dig selv: **"Kan denne liste vokse forbi det jeg har sat som limit?"**

- Hvis ja → kategori A. Pagination eller "indlæs flere" fra dag ét. Det er langt billigere at sætte op nu end at fixe efter brugere har klaget over manglende data.
- Hvis nej → kategori B. Sæt limit'en højt nok til at dække realistiske entries.

ALDRIG `.select('*')` uden nogen form for limit — det er den eneste regel der er absolut.

## Hvornår virtualisering bliver relevant

- 50-200 items synlige: pagination/load-more er nok
- 500+ items synlige i DOM på samme tid: tilføj virtualisering (`react-window` eller `@tanstack/react-virtual`)

## Database — undgå katastrofer

- ALDRIG `SELECT *` på voksende tabeller — vælg konkrete kolonner
- Indexer på kolonner brugt i `WHERE`, `ORDER BY`, `JOIN`
- Composite indexes på filterkombinationer der bruges sammen
- Undgå N+1 — brug Supabase's `.select('*, related(*)')` eller batch-fetches
- Brug `revalidateTag` / `revalidatePath` til at cache read-heavy queries

## Frontend lister og søgning

- Søgning: debounce 300ms + server-side filtering
- ALDRIG filter store datasæt client-side når det kan gøres server-side
- Skeleton loaders ved første load, ikke spinners (mere stabilt layout)

## Billeder

- Brug Next.js `<Image>` med lazy loading som default
- `priority` KUN på above-the-fold billeder
- Sæt korrekt `sizes`-prop på alle Image-komponenter
- ALDRIG importér store JSON-filer ind i bundle — load via fetch ved behov

## Bundle og performance

- Server components som default — kun `'use client'` når nødvendigt
- Undgå at importere tunge libs (`recharts`, `pdfjs`, `monaco`) i shared layouts
- Dynamic imports (`next/dynamic`) for tunge components der ikke vises ved første load

## API-endpoints

- Tilbyd `?limit` og `?cursor` (eller `?page`) params på liste-endpoints der har pagination
- Cache headers (`Cache-Control: s-maxage=...`) på public read-endpoints der ikke er bruger-specifikke

## Realtime og polling

- Supabase Realtime: subscribe KUN på specifikke filtre, aldrig hele tabeller
- Hvis polling: minimum 5-10 sekunder, og kun når faner er aktive (`document.visibilityState`)
````

### `docs/regler/struktur.md`

Skriv NØJAGTIGT følgende indhold:

````markdown
# Struktur-regler

Basale, opinionated retningslinjer for hvordan filer og mapper organiseres. Ikke firkantede — brug dem som standard, afvig hvor det giver mening.

## Mappestruktur — feature-baseret

```text
app/
  tilbud/
    page.tsx
    actions.ts
    components/
      TilbudList.tsx
      TilbudForm.tsx
  kunder/
    page.tsx
    ...
components/
  ui/                  ← genbrugelige primitiver (Button, Input, Card)
lib/
  supabase/
    server.ts
    client.ts
  validation.ts        ← Zod-schemas
docs/
  ARCHITECTURE.md
  regler/
    sikkerhed.md
    skalering.md
    struktur.md
```

Undgå en stor `components/`-bunke med alle komponenter blandet sammen. Hold feature-kode samlet.

## Fil-størrelse og ansvar

- ~200 linjer per fil som tommelfingerregel — split når det vokser, men ikke før
- Én ansvar per fil
- Forretningslogik i `lib/` eller feature-actions, IKKE inde i components
- Components er rendering + interaktion

## Server vs client components

- Server components som default i App Router
- `'use client'` KUN når nødvendigt: state, effects, browser APIs, event handlers
- Hold `'use client'`-komponenter små og blade-løse — push state ned, ikke op
- Data-fetching i server components — undgå `useEffect` til fetching når muligt

## Server actions vs route handlers

- **Server actions** (`actions.ts`): form-submits, mutations fra egen UI
- **Route handlers** (`app/api/.../route.ts`): webhooks, eksterne integrationer, public API
- Bland dem ikke unødigt — en feature bruger som regel kun det ene

## Typer og navngivning

- Typer i samme fil hvor de bruges (co-location). Shared types i `lib/types/`
- ALDRIG `utils.ts`-bunkere — navngiv specifikt: `date-format.ts`, `currency.ts`, `slug.ts`
- Barrel files (`index.ts`) KUN ved public boundaries, ikke overalt

## Beslutninger der SKAL træffes før kode skrives

- Hvilke tabeller findes? Skitsér datamodel
- Hvem ejer hvad? RLS-strategi
- Hvor sker mutationer? Server actions vs API routes
- Hvilken auth-flow? Hvem kan se hvad?

## Beslutninger der KAN udskydes

- Caching-strategi — først når der måles langsomhed
- Background jobs — først når noget reelt er langsomt
- Premature abstraktioner — DRY for tidligt er teknisk gæld

## Tegn på at det er tid til at refaktorere

- Samme query gentages 4+ steder → træk ud i service
- Component har 8+ props → splittes
- Fil overstiger 300 linjer → splittes
- Du er bange for at refaktorere → mangler tests
````

### `.worktreeinclude`

```
.env*
.mcp.json
.claude/settings.local.json
```

### `.env.local.example`

```
# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# Tilføj her hvad svar 1-3 ellers indikerer (Stripe, Resend, OpenAI, Upstash osv.)
```

### `.gitignore`

Generér nøjagtigt dette indhold:

```
# Dependencies
node_modules
.pnp
.pnp.js

# Testing
coverage

# Next.js
.next
out

# Production
build
dist

# Misc
.DS_Store
*.pem

# Debug
npm-debug.log*
yarn-debug.log*
yarn-error.log*
.pnpm-debug.log*

# Local env files
.env*
!.env.local.example

# Vercel
.vercel

# TypeScript
*.tsbuildinfo
next-env.d.ts

# Claude Code
.claude/settings.local.json
```

## 4. Datamodel og RLS-forslag (inline i chatten)

Efter filerne er oprettet, præsenter et kort forslag i chatten:

- Skitsér datamodellen (tabeller + relationer)
- Forslå RLS-policies pr. tabel (én linje pr. operation)
- Marker hvad der er sikkerhedskritisk på baggrund af følsomheds-svaret

## 5. Slutbesked

```
✓ Fundamentet er klar.

Hvad jeg har lavet:
- CLAUDE.md med projekt-konteksten
- docs/ARCHITECTURE.md med datamodel og RLS-strategi
- docs/regler/{sikkerhed,skalering,struktur}.md med opinionated regler
- .gitignore, .env.local.example, .worktreeinclude

Reglerne ligger i dit projekt og bliver auto-læst af Claude Code i hver session. Hvis du senere vil ændre en regel, åbn bare filen og ret den.

Næste skridt: opret Supabase-tabellerne med RLS aktiveret før første query. Når det er klar, så fortæl mig hvad vi bygger først.
```

## 6. Stop

Stop her. Byg ikke features. Stil ikke flere spørgsmål. Brugeren tager over herfra.
