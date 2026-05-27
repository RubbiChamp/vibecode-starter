# vibecode-starter

Et lille Claude Code-plugin der lægger en kort sikkerheds- og strukturkøreplan ind i dit projekt — de lavt hængende frugter Claude følger når den bygger.

Lavet af [Ruben Juncher](https://rubenjuncher.dk) til hans kursusmedlemmer.

## Hvad plugin'et gør

**Én ting:** Når du står i en projektmappe og kører `/start-project`, opretter Claude tre filer:

- `CLAUDE.md` — auto-læses af Claude Code i hver session og peger på reglerne
- `docs/regler/sikkerhed.md` — 8 sikkerhedsregler (de lavt hængende frugter)
- `docs/regler/struktur.md` — basal struktur og konventioner

Det er det. Ingen spørgsmål. Ingen globale skills. Ingen baggrunds-magi. Ingen pre-launch-checks. Ingen kodestil-bibel.

## Hvorfor det her er smart

- **Du ejer reglerne.** Filerne ligger i dit projekt og committes til Git. Vil du ændre en regel, åbn filen og ret den.
- **Reglerne overlever plugin-fjernelse.** Hvis du afinstallerer plugin'et, ligger fundamentet stadig i dit projekt.
- **Ingen globale bivirkninger.** Andre projekter på din computer mærker intet til plugin'et.
- **Rules-first.** Reglerne lægges ned FØR Claude skriver første linje kode — så den allerførste kode er allerede compliant.

## Det reglerne dækker

Sikkerheds-katastroferne for non-developers på Next.js + Supabase:

1. RLS på alle tabeller fra dag ét
2. `service_role`-nøglen aldrig client-side eller `NEXT_PUBLIC_`
3. Ingen hardcoded API-keys, `.env.local` i `.gitignore`
4. `auth.getUser()` ikke `getSession()` server-side
5. Roller og permissions tjekkes aldrig fra client state
6. Ownership-tjek når `service_role` bruges
7. Zod-validering på server actions
8. Webhooks: signature-verifikation med rå body (`req.text()`)

Plus tre småting der ofte glemmes: Supabase Storage policies, `dangerouslySetInnerHTML` med DOMPurify, og at undlade at returnere rå DB-fejl til client.

## Anbefalet workflow

```
1. Brainstorm idé i almindelig Claude → få en PRD ud
2. Lav en tom mappe og læg PRD.md ind
3. Åbn Claude Code i mappen
4. Kør /start-project (fundamentet lægges)
5. Sig til Claude: "Byg projektet fra PRD.md og udfyld CLAUDE.md"
6. Tag det derfra — Claude spørger undervejs om at oprette Supabase-projekt,
   GitHub-repo osv.
```

## Stack-anbefaling

Plugin'et er kalibreret til **Next.js (App Router) + TypeScript + Tailwind + Supabase + Vercel**. Hvis du bruger en anden stack, så åbn `docs/regler/`-filerne og tilret dem — eller drop plugin'et.

## Installation

I Claude Code:

```
/plugin marketplace add RubbiChamp/vibecode-starter
/plugin install vibecode-starter@juncher
/reload-plugins
```

## Brug

I en projektmappe (gerne med en `PRD.md`):

```
/start-project
```

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
