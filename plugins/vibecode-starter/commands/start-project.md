---
description: Lægger en sikkerheds- og strukturkøreplan ind i projektet, som Claude vil følge når den bygger.
---

Du skal nu lægge fundamentet til et nyt vibe coding-projekt. Hele flowet er på dansk. Stil ingen spørgsmål — bare opret filerne.

## Opret tre filer

Brug `Write`-tool'et til hver fil.

### `CLAUDE.md`

```markdown
# Projekt

Dette projekt bruger lokale regelfiler som du SKAL læse før du skriver eller ændrer kode:

- `docs/regler/sikkerhed.md` — sikkerhedsregler (RLS, secrets, server actions, rate limiting m.m.)
- `docs/regler/struktur.md` — mappestruktur og kodekonventioner

Reglerne er kalibreret til **Next.js (App Router) + TypeScript + Tailwind + Supabase + Vercel**. Hvis projektet skifter stack, opdater eller fjern reglerne.

## Når du bygger projektet

Når brugeren beder dig bygge dette projekt (typisk ud fra en `PRD.md` eller anden specifikation):

1. Læs regelfilerne først
2. Følg dem fra første linje kode
3. Foretræk **Supabase MCP** (`mcp__supabase__*`) til database-arbejde når den er konfigureret
4. Foretræk **GitHub CLI** (`gh repo create`, `gh pr` osv.) til GitHub-arbejde
5. Stop og spørg brugeren om bekræftelse før du opretter eksterne ressourcer (Supabase-projekter, GitHub repos, deploys)
6. Udfyld denne `CLAUDE.md` med projekt-konteksten når du bygger — overskriv denne sektion med "Hvad bygger vi", "Brugere og auth", "Vigtige ting i systemet" osv.

## Mentor-mandat

Brugeren er ofte ikke-developer. Når du implementerer noget der kræver opsætning udenfor koden (env vars, eksterne services, integrationer), forklar i 3-4 nummererede trin hvad de skal gøre, og hvorfor det er vigtigt — i én linje.
```

### `docs/regler/sikkerhed.md`

````markdown
# Sikkerhedsregler

Disse regler gælder for hele dette projekt. Følg dem når du genererer eller ændrer kode. De er destilleret fra fælles fælder vibe-coders falder i på Next.js + Supabase.

## Mentor-mandat

Brugeren er ofte ikke-developer. Når du implementerer noget sikkerhedsrelateret (rate limiting, env vars, RLS, auth), så **forklar i 3-4 nummererede trin hvad brugeren skal gøre udenfor koden** — fx tilføj env vars i Vercel, opret en konto hos en service, klik en bestemt knap. Forklar HVORFOR i én linje. Vær konkret, ikke abstrakt.

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

### `docs/regler/struktur.md`

````markdown
# Struktur-regler

Basale, opinionated retningslinjer for hvordan filer og mapper organiseres. Brug dem som standard, afvig hvor det giver mening.

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
  regler/
    sikkerhed.md
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

## .gitignore — ekstra entries efter scaffolding

Når Next.js scaffoldes via `create-next-app`, opretter scriptet en `.gitignore`. Tilføj følgende linjer til den efter scaffolding:

```
# Local env files
.env*
!.env.local.example

# Claude Code
.claude/settings.local.json
```

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

## Slutbesked

Send følgende:

```
✓ Sikkerheds- og strukturkøreplanen er klar.

Tre filer er lagt ind i projektet:
- CLAUDE.md (auto-læses af mig i hver session)
- docs/regler/sikkerhed.md
- docs/regler/struktur.md

Næste skridt: bed mig om at bygge projektet — gerne med reference til en PRD-fil hvis du har én. Fx:

  "Byg dette projekt fra PRD.md og udfyld CLAUDE.md."

Reglerne ligger i dit projekt og bliver auto-læst af mig i hver session. Du må gerne ændre dem hvis noget ikke passer dit specifikke projekt.
```

## Stop

Stop her. Byg ikke features. Stil ikke spørgsmål. Brugeren tager over.
