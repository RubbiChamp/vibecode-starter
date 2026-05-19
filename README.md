# vibecode-starter

Et Claude Code-plugin der gør Claude skarpere på security, skalering og struktur når du vibe coder Next.js + Supabase-projekter.

Lavet af [Ruben Juncher](https://rubenjuncher.dk) til hans kursusmedlemmer.

## Hvad er i kassen

- **`juncher-security`** — auto-aktiverer på server actions, database queries, auth-flows, secrets, JWT, cookies, og alt med user input. Forhindrer RLS-fejl, secret-lækager, IDOR/BOLA, dårlig validering, og resten af de klassiske fælder.
- **`juncher-scaling`** — auto-aktiverer på lister, paginering, queries, billeder og dashboards. Forhindrer `SELECT *`, manglende pagination, ikke-virtualiserede lister og bundle-mareridt.
- **`juncher-architecture`** — auto-aktiverer ved nye filer, refaktoreringer og struktur-spørgsmål. Holder mappestruktur, fil-størrelser og server/client-grænser sunde.
- **`/start-project`** — én slash-command der bygger projekt-fundamentet (CLAUDE.md, ARCHITECTURE.md, PROGRESS.md, `.env.local.example`, `.gitignore`, `.worktreeinclude`) efter 5 hurtige spørgsmål.

Plugin'et **bygger ikke koden for dig**, det **scanner ikke koden**, og det **validerer ikke noget**. Det er en stille mentor i baggrunden, der får Claude til at træffe de rigtige valg fra start.

## Installation

I Claude Code:

```
/plugin marketplace add RubbiChamp/vibecode-starter
/plugin install vibecode-starter@juncher
/reload-plugins
```

Derefter er skills aktive automatisk i alle dine projekter.

## Sådan starter du et nyt projekt

Naviger til en tom mappe og kør:

```
/start-project
```

Claude stiller fem strategiske spørgsmål og opretter herefter projekt-fundamentet.

## Opdatering

```
/plugin update vibecode-starter@juncher
/reload-plugins
```

## Understøttede overflader

| Overflade | Status |
|-----------|--------|
| Claude Code terminal (Mac) | ✅ |
| Claude Code terminal (Windows) | ✅ |
| Claude Code i VS Code | ✅ (sandsynligvis) |
| Claude desktop-app | ❌ (plugins virker pt. ikke der) |

## Credit

Security-reglerne er destilleret fra det enorme arbejde Christoffer Ohlsen har lagt i [SafeVibe.Codes](https://safevibe.codes) — over 190 statiske og dynamiske checks for vibe-coded apps. Plugin'et importerer ingen kode derfra; det er ren viden, oversat til adfærdsregler for Claude.

## Licens

MIT.
