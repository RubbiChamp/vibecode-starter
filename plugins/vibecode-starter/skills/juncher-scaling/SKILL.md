---
name: juncher-scaling
description: Skaleringsregler for Next.js + Supabase. Bruges PROACTIVELY når der bygges lister, tabeller, søgefelter, dashboards, paginering, database-queries der returnerer arrays, eller noget der kan vokse i datavolumen. Også PROACTIVELY ved billede-håndtering, fetches der returnerer mange items, API-endpoints, caching-beslutninger, og når der diskuteres performance. Skal aktiveres når brugeren skriver SELECT-queries, useEffect med data-fetching, map() over store arrays, eller bygger paginerede UI'er.
---

# Juncher scaling-regler

Forhindre dårlige skaleringsbeslutninger fra dag ét. Det er billigere at bygge skalerbart fra start end at refaktorere senere.

## Database queries

- ALDRIG `SELECT *` uden `LIMIT` — heller ikke i prototyper
- Pagination på ALLE list-views fra dag ét. Default 20, max 100 per side
- Cursor-baseret pagination foretrækkes over `OFFSET` ved tabeller med 10.000+ rækker
- Indexer på alle kolonner brugt i `WHERE`, `ORDER BY`, `JOIN`
- Composite indexes på filterkombinationer der bruges sammen (fx `(user_id, created_at)`)
- Undgå N+1 — brug joins, Supabase's `.select('*, related(*)')`, eller batch-fetches
- Brug `revalidateTag` / `revalidatePath` til at cache read-heavy queries

## Frontend lister

- Lister med 50+ items SKAL virtualiseres (`react-window` eller `@tanstack/react-virtual`)
- Søgning: debounce 300ms + server-side filtering
- ALDRIG filter store datasæt client-side — server skal gøre arbejdet
- Skeleton loaders ved første load, ikke spinners (mere stabilt layout)

## Billeder og assets

- Brug Next.js `<Image>` med lazy loading som default
- `priority` KUN på above-the-fold billeder
- Sæt korrekt `sizes`-prop på alle Image-komponenter (ellers downloades fuld størrelse)
- ALDRIG importér store JSON-filer ind i bundle — load via fetch ved behov

## API og endpoints

- Default page size 20, max 100
- Tilbyd `?limit` og `?cursor` (eller `?page`) params på alle list-endpoints
- Cache headers på public read-endpoints
- Brug `Cache-Control: s-maxage=...` på Vercel-deployerede endpoints hvor data ikke er bruger-specifikt

## Performance og bundle

- Server components som default (Next.js App Router) — kun `'use client'` når strengt nødvendigt
- Code splitting: undgå at importere tunge libs (`recharts`, `pdfjs`, `monaco`) i shared layouts
- Dynamic imports (`next/dynamic`) for tunge components der ikke vises ved første load
- Tjek bundle-størrelse ved hjælp af `@next/bundle-analyzer` hvis appen bliver træg

## Realtime og polling

- Supabase Realtime: subscribe KUN på specifikke filtre, aldrig hele tabeller
- Hvis polling: minimum 5-10 sekunder, og kun når faner er aktive (`document.visibilityState`)
- Brug Server-Sent Events eller Realtime fremfor polling når muligt

## GØR / IKKE GØR

```ts
// GØR
const { data: tilbud } = await supabase
  .from('tilbud')
  .select('id, titel, pris, created_at')
  .order('created_at', { ascending: false })
  .limit(20);

// IKKE GØR
const { data: tilbud } = await supabase.from('tilbud').select('*');  // henter ALT, hele rækken
```

```tsx
// GØR
<Image src={url} alt={...} width={400} height={300} sizes="(max-width: 768px) 100vw, 400px" />

// IKKE GØR
<img src={url} />  // ingen optimering, ingen lazy loading
```

## Tidlige advarselstegn

- Query returnerer ingen `LIMIT` → tilføj nu
- Component renderer 100+ items uden virtualisering → virtualiser nu
- `useEffect` der refetcher ved hver render → fix dependency-arrayet
- Filterlogik kører `array.filter()` på 1000+ items i browseren → flyt til server
