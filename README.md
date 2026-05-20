# vibecode-starter

Et lille Claude Code-plugin der bootstrapper dit Next.js + Supabase-projekt med en gennemtænkt `CLAUDE.md`, en arkitektur-skitse og tre lokale regelfiler om sikkerhed, skalering og struktur.

Lavet af [Ruben Juncher](https://rubenjuncher.dk) til hans kursusmedlemmer.

## Hvad gør plugin'et

**Én ting:** Når du står i en tom projektmappe og kører `/start-project`, opretter Claude:

- `CLAUDE.md` med projekt-konteksten (Claude læser den automatisk hver session)
- `docs/ARCHITECTURE.md` med foreslået datamodel og RLS-strategi
- `docs/PROGRESS.md` — log over hvad der sker i projektet
- `docs/regler/sikkerhed.md` — destilleret fra fælles fælder på Next.js + Supabase
- `docs/regler/skalering.md` — pragmatiske regler, ingen premature optimization
- `docs/regler/struktur.md` — feature-baseret mappestruktur og basale konventioner
- `.env.local.example` og `.worktreeinclude`

Plugin'et gør **intet andet** end at oprette de filer. Der er ingen globale skills, ingen baggrunds-magi, intet der kører i dine andre projekter.

## Det anbefalede workflow

```
1. I almindelig Claude:    Diskuter idé → få en PRD ud
2. Lokalt:                  Tom mappe + læg PRD.md ind
3. Åbn Claude Code:         /start-project (læser PRD, lægger fundament)
4. Sig til Claude:          "Byg projektet fra PRD.md og CLAUDE.md"
5. Sig så til Claude:       "Opret Supabase-tabellerne med RLS som ARCHITECTURE.md siger"
6. Begynd at bygge features
```

**Hvis du ikke har en PRD:** Kør `/start-project` alligevel — Claude stiller 3 hurtige spørgsmål i stedet og bygger fundamentet ud fra dem.

## Hvorfor det her er smart

- **Du ejer reglerne.** Filerne ligger i dit projekt og committes til Git. Vil du ændre en regel, åbn filen og ret den.
- **Reglerne overlever plugin-fjernelse.** Hvis du afinstallerer plugin'et, ligger fundamentet stadig i dit projekt.
- **Ingen globale bivirkninger.** Andre projekter på din computer mærker intet til plugin'et.
- **Rules-first.** Reglerne lægges ned FØR Claude skriver første linje kode — så den allerførste kode er allerede compliant.
- **Claude Code læser CLAUDE.md automatisk.** Reglerne er aktive i hver session — fordi Claude faktisk åbner dem, ikke fordi der sker noget magisk i baggrunden.

## Stack-anbefaling (ikke -låsning)

Plugin'et er kalibreret til **Next.js (App Router) + TypeScript + Tailwind + Supabase + Vercel**. Det er den stack reglerne forudsætter — især sikkerhedsreglerne om RLS, `auth.uid()` og Supabase-client.

Hvis du bruger en anden stack (Drizzle, Prisma, Firebase, Pages Router), bør du **ikke bruge plugin'et** — eller alternativt: kør `/start-project`, slet de regelfiler der ikke passer, og tilret resten manuelt.

## Installation

I Claude Code:

```
/plugin marketplace add RubbiChamp/vibecode-starter
/plugin install vibecode-starter@juncher
/reload-plugins
```

## Engangs-opsætning anbefales også

Før dit første projekt, sæt **Supabase MCP** op globalt på din computer. Det giver Claude Code adgang til at oprette tabeller, køre migrations og inspicere data direkte via din Supabase-konto. Se Rubens kursusvideo for den 2-minutters opsætning.

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

Sikkerhedsreglerne er destilleret fra det enorme arbejde Christoffer Ohlsen har lagt i [SafeVibe.Codes](https://safevibe.codes) — over 190 statiske og dynamiske checks for vibe-coded apps. Plugin'et importerer ingen kode derfra; det er ren viden, oversat til menneskeligt sprog.

## Licens

MIT.
