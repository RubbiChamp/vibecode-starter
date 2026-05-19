---
description: Starter et nyt vibe coding-projekt med Junchers fundament. Stiller strategiske spørgsmål og opretter projekt-strukturen.
---

Du skal nu køre `/start-project`-flowet. Følg disse instruktioner præcist.

## 1. Velkomstbesked

Send følgende besked (eller en meget tæt parafrase — direkte, lidt humor, Rubens stil):

> Lad os bygge fundamentet ordentligt. Tager 3 minutter, sparer dig 30 timer senere. Jeg stiller dig 5 hurtige spørgsmål — svar kort, så samler jeg det op til en ordentlig projektstart. Klar?

Vent på at brugeren bekræfter (eller bare svarer) før du fortsætter.

## 2. Stil de 5 spørgsmål — ét ad gangen

VIGTIGT: Stil ét spørgsmål ad gangen. Vent på svar mellem hvert. Ingen lister, ingen "her er alle spørgsmålene på én gang".

1. **Hvad bygger vi?** (én sætning er nok)
2. **Hvem er brugerne, og skal de logge ind?**
3. **Hvilke 3-5 'ting' findes i systemet?** (dvs. de centrale entities/tabeller — fx "tilbud, kunder, fakturaer")
4. **Hvad er det privateste data der gemmes?** (fx CPR, betalingsoplysninger, helbredsdata, intet særligt)
5. **Hvor skal det hostes?** (Default-anbefaling: Vercel. Spørg om det passer.)

Notér svarene mens du går.

## 3. Generér filerne

Efter alle 5 svar, opret følgende filer i projektets root (cwd):

### `CLAUDE.md` (max ~80 linjer)

```markdown
# Projekt: [projekt-navn udledt af svar 1]

## Hvad bygger vi
[Svar 1 — én sætning]

## Brugere og auth
[Svar 2 — kort om hvem brugerne er og om de logger ind]

## Stack
- Next.js 15 App Router + TypeScript + Tailwind
- Supabase (Postgres + Auth + RLS)
- Hostet på [svar 5]

## Entities
[Listen fra svar 3, kort skitse — én linje per entity med 2-3 nøglefelter]

## Privacy-kategori
[Svar 4 — og hvad det betyder for arkitekturen, fx "CPR-data kræver ekstra RLS-disciplin og audit-log"]

## Aktive guardrails
Dette projekt bruger `juncher-security`, `juncher-scaling` og `juncher-architecture` skills fra `vibecode-starter`-plugin'et. Skills auto-aktiveres baseret på kontekst, så Claude følger god praksis uden at du behøver bede om det.

Skill-filerne ligger i `~/.claude/plugins/marketplaces/juncher/plugins/vibecode-starter/skills/`.

## Konventioner
- ALLE mutations sker i server actions med Zod-validering
- ALLE Supabase-tabeller har RLS aktiveret fra dag ét
- Læs `docs/ARCHITECTURE.md` før større ændringer

## Verifikation
- `pnpm typecheck`
- `pnpm test`
- `pnpm build`
```

### `docs/ARCHITECTURE.md`

Indhold:
- Foreslået datamodel baseret på svar 3 — tabel pr. entity med foreslåede kolonner og foreign keys
- RLS-strategi baseret på svar 2 og 4 — hvem kan se hvad, hvilke policies skal eksistere
- Beslutning om server actions vs route handlers
- Hvilke routes er public vs auth'd

### `docs/PROGRESS.md`

Brug faktisk dato i format `YYYY-MM-DD` — fx `2026-05-13`. Hvis du ikke kender dagens dato, så spørg brugeren først eller udled den fra system-konteksten (svarende til `new Date().toISOString().split('T')[0]`).

```markdown
# Progress log

- YYYY-MM-DD Projekt startet med `/start-project`
```

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

# Tilføj her hvad svar 1-5 ellers indikerer (Stripe, Resend, OpenAI, osv.)
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

- Skitser datamodellen (entities + relationer)
- Forslå RLS-policies pr. tabel (én linje pr. operation)
- Marker hvad der er sikkerhedskritisk på baggrund af svar 4

## 5. Slutbesked

```
✓ Fundamentet er klar.

Tre guardrails er aktive i dette projekt:
- juncher-security (sikkerhed)
- juncher-scaling (skalering)
- juncher-architecture (struktur)

Næste skridt: opret Supabase-tabellerne med RLS aktiveret før første query. Når det er klar, så fortæl mig hvad vi bygger først.
```

## 6. Stop

Stop her. Byg ikke features. Stil ikke flere spørgsmål. Brugeren tager over herfra.
