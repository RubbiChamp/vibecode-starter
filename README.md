# vibecode-starter

Et lille Claude Code-plugin der bootstrapper dit Next.js + Supabase-projekt med en gennemtænkt `CLAUDE.md`, en arkitektur-skitse og tre lokale regelfiler om sikkerhed, skalering og struktur.

Lavet af [Ruben Juncher](https://rubenjuncher.dk) til hans kursusmedlemmer.

## Hvad gør plugin'et

**Én ting:** Når du står i en tom projektmappe og kører `/start-project`, stiller Claude dig 3 spørgsmål om dit projekt og opretter derefter:

- `CLAUDE.md` med projekt-konteksten (Claude læser den automatisk hver session)
- `docs/ARCHITECTURE.md` med foreslået datamodel og RLS-strategi
- `docs/PROGRESS.md` — log over hvad der sker i projektet
- `docs/regler/sikkerhed.md` — destilleret fra fælles fælder på Next.js + Supabase
- `docs/regler/skalering.md` — pragmatiske regler, ingen premature optimization
- `docs/regler/struktur.md` — feature-baseret mappestruktur og basale konventioner
- `.gitignore`, `.env.local.example`, `.worktreeinclude`

Det er det. Plugin'et gør **intet andet** end at oprette de filer. Der er ingen globale skills, ingen baggrunds-magi, intet der kører i dine andre projekter.

## Hvorfor det her er smart

- **Du ejer reglerne.** Filerne ligger i dit projekt og committes til Git. Vil du ændre en regel, åbn filen og ret den.
- **Reglerne overlever plugin-fjernelse.** Hvis du afinstallerer plugin'et, ligger fundamentet stadig i dit projekt.
- **Ingen globale bivirkninger.** Andre projekter på din computer mærker intet til plugin'et.
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

## Brug

Naviger til en tom mappe og kør:

```
/start-project
```

Claude stiller 3 spørgsmål og opretter fundamentet. Tager 5-10 minutter.

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
