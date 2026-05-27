---
description: Lægger en kort sikkerheds- og strukturkøreplan ind i projektet, som Claude følger når den bygger.
---

Du skal nu lægge fundamentet til et nyt vibe coding-projekt. Hele flowet er på dansk. Stil ingen spørgsmål — bare opret filerne.

## Opret tre filer

Brug `Write`-tool'et til hver fil.

### `CLAUDE.md`

```markdown
# Projekt

Dette projekt bruger to lokale regelfiler som du SKAL læse før du skriver eller ændrer kode:

- `docs/regler/sikkerhed.md` — de vigtigste sikkerhedsregler
- `docs/regler/struktur.md` — de basale strukturregler

Reglerne er kalibreret til **Next.js (App Router) + TypeScript + Tailwind + Supabase + Vercel**.

## Sådan arbejder du i dette projekt

Disse regler gælder altid — uanset om det er første build eller du tilføjer features senere:

1. Læs regelfilerne først
2. Følg dem fra første linje kode
3. Foretræk **Supabase MCP** (`mcp__supabase__*`) til database-arbejde når den er konfigureret
4. Foretræk **GitHub CLI** (`gh repo create`, `gh pr` osv.) til GitHub-arbejde
5. Stop og spørg brugeren om bekræftelse før du opretter eksterne ressourcer (Supabase-projekter, GitHub repos, deploys)
6. **Tilføj sektioner NEDENFOR** med projekt-konteksten ("Hvad bygger vi", "Brugere og auth", "Vigtige ting i systemet" osv.) — men **lad sektionerne ovenfor stå urørt**, så regelfil-referencerne stadig konsulteres i fremtidige sessions.

## Mentor-mandat

Brugeren er ofte ikke-developer. Når du implementerer noget der kræver opsætning udenfor koden (env vars, eksterne services, integrationer), forklar i 3-4 nummererede trin hvad de skal gøre — og hvorfor det er vigtigt i én linje. Når du selv kan ordne det via Supabase MCP eller `gh` CLI, så bare gør det — ingen trin nødvendige.

---

<!-- Projekt-kontekst tilføjes nedenfor af Claude når projektet bygges -->
```

### `docs/regler/sikkerhed.md`

````markdown
# Sikkerhedsregler

De lavt hængende frugter. Følg dem altid — uanset om appen er en MVP eller skal launches. Det er reglerne der forhindrer at databasen er åben for verden, eller at en bedrager kan trigge "betaling lykkedes".

**Reglerne er konditionelle.** Hvis projektet ikke har Stripe, gælder Stripe-reglen ikke. Hvis det ikke har auth eller `service_role`, gælder de regler heller ikke. Brug dem som tjekliste når situationen opstår — ikke som påbud om at indføre nye features.

## Mentor-mandat

Når du implementerer noget sikkerhedsrelateret OG brugeren skal gøre noget udenfor koden (tilføje env vars i Vercel, klikke en knap i Stripe Dashboard, oprette en konto), forklar det i 3-4 nummererede trin og hvorfor i én linje. Brugeren er ofte ikke-developer.

Når du selv kan ordne det via Supabase MCP, `gh` CLI eller andre værktøjer, så bare gør det — ingen trin nødvendige.

## Reglerne

1. **RLS aktiveret på ALLE tabeller fra dag ét.** Skriv eksplicitte policies for SELECT/INSERT/UPDATE/DELETE med `auth.uid()`. Uden RLS kan anon-nøglen læse og skrive alt.

2. **`service_role`-nøglen ALDRIG client-side eller med `NEXT_PUBLIC_`-prefiks.** Den giver fuld DB-adgang og må kun bruges i server actions/route handlers — og kun når det er nødvendigt (admin-tasks, omgå RLS).

3. **Ingen hardcoded API-keys i kildekoden.** Patterns at afvise: `sk-`, `sk_live_`, `AKIA`, `ghp_`, `sk-ant-`. Alt skal i `.env.local` der er i `.gitignore`. `.env.local.example` committes med pladsholdere.

4. **Brug `auth.getUser()` (IKKE `getSession()`) server-side.** `getSession()` validerer ikke JWT'en — den kan forfalskes. `getUser()` rammer Supabase og verificerer.

5. **Når du bruger `service_role` i server action: tjek ownership eksplicit** (`user_id === user.id`). RLS gælder ikke når du bruger `service_role`. Når du bruger anon-klienten, stol på RLS — duplikér ikke tjek.

6. **Validér inputs med Zod på server actions** der skriver til DB eller kalder eksterne services. RLS beskytter mod uautoriseret adgang, ikke mod dårlige data.

7. **HVIS projektet bruger webhooks** (Stripe, Resend, Supabase auth-hooks osv.): verificér signaturen FØR du gør noget. Kritisk gotcha i Next.js App Router — body SKAL være rå tekst, ikke parsed JSON:

    ```ts
    // app/api/webhooks/stripe/route.ts
    const body = await req.text();  // IKKE req.json()
    const signature = req.headers.get('stripe-signature')!;
    const event = stripe.webhooks.constructEvent(body, signature, process.env.STRIPE_WEBHOOK_SECRET!);
    ```

    Uden `req.text()` fejler signatur-verifikationen tavst i App Router.

## Småting der ofte glemmes

- **Supabase Storage** har sin egen access-model. Sæt bucket-policies op før første upload — public buckets lækker som åbne RLS-tabeller.
- **`dangerouslySetInnerHTML`** kun med DOMPurify. Ellers er det XSS = kontooverskridelse.
- **Returnér ALDRIG rå database-fejl til client** — log server-side, send generisk besked. Fejl indeholder ofte struktur-info angribere kan bruge.
````

### `docs/regler/struktur.md`

````markdown
# Strukturregler

Den åbenlyse vej, ikke en kodestil-bibel. Følg dem som standard, afvig hvor det giver mening.

## Server vs client components

- Server components som default i App Router
- `'use client'` KUN når nødvendigt: state, effects, browser APIs, event handlers
- Data-fetching i server components — undgå `useEffect` til fetching når muligt

## Server actions vs route handlers

- **Server actions** (`actions.ts`): form-submits, mutations fra egen UI
- **Route handlers** (`app/api/.../route.ts`): webhooks, eksterne integrationer, public API

## Mappe-organisering

- Hold feature-kode samlet: når et område vokser til 3+ filer, lav en mappe til det
- `components/ui/` til genbrugelige primitiver (Button, Input, Card)
- `lib/supabase/server.ts` og `lib/supabase/client.ts` — adskilte clients
- ALDRIG en `utils.ts`-bunke — navngiv specifikt: `date-format.ts`, `currency.ts`, `slug.ts`

## `.gitignore` — ekstra entries efter scaffolding

Når Next.js scaffoldes via `create-next-app`, opretter scriptet en `.gitignore`. Tilføj følgende linjer til den efter scaffolding:

```
# Local env files
.env*
!.env.local.example

# Claude Code
.claude/settings.local.json
```

## Beslutninger der kan udskydes

- Caching-strategi — først når der måles langsomhed
- Background jobs — først når noget reelt er langsomt
- Premature abstraktioner — DRY for tidligt er teknisk gæld

Når du er i tvivl om struktur: vælg det der er nemmest at læse om to uger, ikke det der virker mest "professionelt".
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
