---
name: juncher-architecture
description: Arkitektur-regler for Next.js-projekter. Bruges PROACTIVELY når der oprettes nye filer, mapper eller components, ved refaktorering, når en fil vokser, eller ved arkitekturelle valg som "hvor skal denne logik bo" eller "skal dette være server eller client component". Også PROACTIVELY ved navngivning, mappestruktur-spørgsmål, beslutninger om server actions vs route handlers, og når kompleksitet stiger i en feature. Skal aktiveres ved oprettelse af `app/`-routes, `components/`, `lib/`-moduler, eller når brugeren spørger "hvor skal X ligge".
---

# Juncher architecture-regler

Træf de rigtige strukturelle beslutninger fra start. Dårlig struktur koster mere end manglende features.

## Mappestruktur

- Feature-baseret mappestruktur — `app/tilbud/`, `app/kunder/`, IKKE en stor `components/`-bunke
- `app/` til routes (App Router)
- `app/actions/` ELLER `app/<feature>/actions.ts` til server actions
- `lib/` til shared utilities, hooks, config
- `lib/supabase/server.ts` og `lib/supabase/client.ts` — adskilte clients
- `components/ui/` KUN til genbrugelige UI-primitiver (Button, Input, Card)
- `components/<feature>/` til feature-specifikke components

## Fil-størrelse og ansvar

- Maks ~200 linjer per fil — split når det vokser
- Én ansvar per fil
- Business logic i `lib/` eller `services/`, IKKE i components
- Components er rendering + interaktion, ikke forretningslogik

## Typer og navngivning

- Typer i samme fil hvor de bruges (co-location)
- Shared types kun i `types/` eller `lib/types/`
- ALDRIG `utils.ts`-bunkere — navngiv specifikt: `date-format.ts`, `currency.ts`, `slug.ts`
- Barrel files (`index.ts`) KUN ved public boundaries, ikke overalt

## Server vs client components

- Server components som default i App Router
- `'use client'` KUN når nødvendigt: state, effects, browser APIs, event handlers
- Hold `'use client'`-komponenter små og blade-løse — push state ned, ikke op
- Data-fetching i server components — undgå `useEffect` til fetching når muligt

## Server actions vs route handlers

- Server actions: form-submits, mutations fra egen UI
- Route handlers (`app/api/.../route.ts`): webhooks, eksterne integrationer, public API
- Mix dem ikke — en feature bruger som regel kun det ene

## Beslutninger der SKAL træffes før kode skrives

- Hvilke entities (database-tabeller) findes? Skitsér datamodel
- Hvem ejer data? RLS-strategi defineret fra dag ét
- Hvor sker mutationer? Server actions vs API routes
- Hvad er public surface? Hvilke routes/endpoints er åbne?
- Hvilken auth-flow? Hvem kan se hvad?

## Beslutninger der KAN udskydes (premature optimization)

- Caching-strategi — først når der måles langsomhed
- Background jobs — først når noget reelt er langsomt
- Microservices — næsten aldrig for vibe-coded MVPs
- Premature abstraktioner — DRY for tidligt er teknisk gæld

## Tegn på undgået beslutning der må tages nu

- Samme query gentages 4+ steder → træk ud i service
- Component har 8+ props → splittes eller komponer anderledes
- Fil overstiger 300 linjer → splittes
- Du er bange for at refaktorere → mangler tests

## GØR / IKKE GØR

```text
// GØR — feature-baseret
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

// IKKE GØR — type-baseret bunke
components/
  TilbudList.tsx
  TilbudForm.tsx
  KundeList.tsx
  KundeForm.tsx
  ...50 andre filer
```

```ts
// GØR — co-location af typer
// app/tilbud/actions.ts
const TilbudSchema = z.object({ titel: z.string(), pris: z.number() });
type Tilbud = z.infer<typeof TilbudSchema>;

// IKKE GØR — alt i types/index.ts
// types/index.ts (500 linjer, alle typer i hele appen)
```
