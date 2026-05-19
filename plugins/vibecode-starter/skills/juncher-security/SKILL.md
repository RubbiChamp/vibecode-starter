---
name: juncher-security
description: Sikkerheds-regler for Next.js + Supabase vibe coding. Bruges PROACTIVELY når der skrives server actions, route handlers, database queries, Supabase client setup, auth flows, miljøvariabler, JWT-håndtering eller noget med user input. Også PROACTIVELY ved alt der involverer secrets, API keys, cookies, sessions, RLS policies, persondata, file uploads, eller eksterne API-kald. Skal aktiveres når brugeren skriver kode mod Supabase, opretter tabeller, definerer auth-logik, eller eksponerer endpoints.
---

# Juncher security-regler

Disse regler gælder for ALLE Next.js + Supabase-projekter. Følg dem som standard — afvig kun hvis brugeren eksplicit beder om det og forstår konsekvensen.

## RLS og Supabase

- Enable RLS på ALLE tabeller fra dag ét, FØR første query køres
- Skriv eksplicitte policies for hver operation: SELECT, INSERT, UPDATE, DELETE — ingen "FOR ALL" som tankeløs default
- Brug `auth.uid()` i policies — aldrig kun `auth.role()` til ownership-check
- ALDRIG `service_role`-nøgle i `NEXT_PUBLIC_`-prefiks eller client-side kode — det er en total kompromittering
- Brug `anon`-nøglen client-side; `service_role` kun i server actions og kun når nødvendigt (fx admin-tasks)
- Tjek altid ownership server-side selvom RLS er aktiv (defense in depth)

## Secrets-håndtering

- ALDRIG hardcoded API-keys i kildekode. Patterns at afvise: `sk-`, `sk_live_`, `AKIA`, `ghp_`, `sk-ant-`, `xoxb-`
- ALDRIG `NEXT_PUBLIC_`-prefiks på secrets — det eksponeres i browser-bundlet
- `.env.local` SKAL være i `.gitignore`
- `.env.local.example` committes med kun pladsholdere
- Brug `process.env.X` server-side og verificer eksistens ved opstart — fejl tidligt med klar besked

## Server actions og database-adgang

- ALDRIG direkte Supabase-client-mutationer fra client components — wrap alt i server actions eller route handlers
- Kald `auth.getUser()` (IKKE `getSession()`) server-side før mutationer — `getSession()` validerer ikke JWT'en
- Tjek ownership (`user_id === user.id`) på alle reads og writes — forhindrer BOLA/IDOR
- Validér ALLE inputs med Zod FØR de når databasen

## Input-validering og output

- Zod-schemas for alle form-inputs, API-bodies, search-params, route params
- Client-side validering er UX, ikke sikkerhed — gentag alt server-side
- ALDRIG returnér rå database-fejl til client — log server-side, send generisk besked
- ALDRIG `dangerouslySetInnerHTML` uden DOMPurify
- ALDRIG `eval()` eller `new Function()` med user input

## Auth og JWT

- ALDRIG `algorithm: "none"` i JWT
- Altid `expiresIn` på JWT'er — ingen evige tokens
- ALDRIG `jwt.decode()` uden `jwt.verify()` først
- Cookies SKAL have: `httpOnly: true`, `secure: true` i production, `sameSite: 'lax'` eller `'strict'`

## Rate limiting og misbrug

- Implementér rate limiting på public endpoints — Upstash Ratelimit er default-anbefalingen
- Begræns request body-størrelse på endpoints der modtager bodies (default max 1MB hvis ikke andet er nødvendigt)
- Login, signup, password-reset og AI-endpoints SKAL rate-limites pr. IP og pr. konto

## Headers og CORS

- CORS: konkret origin, ALDRIG `*` i production
- Sæt CSP, X-Frame-Options (DENY), Strict-Transport-Security i `next.config.js`
- Fjern `X-Powered-By` og evt. `Server` headers der lækker stack-info

## SQL og injection

- Brug Supabase-client eller Drizzle ORM — ALDRIG template literals i raw SQL
- Parametriserede queries altid
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
