# Understøttede sprog

Chloros tilbyder fuld understøttelse af brugergrænsefladen på **38 sprog verden over**, hvilket gør det tilgængeligt for brugere over hele kloden. Du kan skifte sprog med det samme både i desktop-brugergrænsefladen og i CLI.

Chloros understøtter følgende sprog:

| # | Sprog | Navn på originalsproget | CLI-kode |
|---|----------|-------------|----------|
| 1 | 🇺🇸 Engelsk | English | `en` |
| 2 | 🇪🇸 Spansk | Español | `es` |
| 3 | 🇵🇹 Portugisisk | Português | `pt` |
| 4 | 🇫🇷 Fransk | Français | `fr` |
| 5 | 🇩🇪 Tysk | Deutsch | `de` |
| 6 | 🇮🇹 Italiensk | Italiano | `it` |
| 7 | 🇯🇵 Japansk | 日本語 | `ja` |
| 8 | 🇰🇷 Koreansk | 한국어 | `ko` |
| 9 | 🇨🇳 Kinesisk (forenklet) | 简体中文 | `zh` |
| 10 | 🇹🇼 Kinesisk (traditionel) | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 Russisk | Русский | `ru` |
| 12 | 🇳🇱 Hollandsk | Nederlands | `nl` |
| 13 | 🇸🇦 Arabisk | العربية | `ar` |
| 14 | 🇵🇱 Polsk | Polski | `pl` |
| 15 | 🇹🇷 Tyrkisk | Türkçe | `tr` |
| 16 | 🇮🇳 Hindi | हिंदी | `hi` |
| 17 | 🇮🇩 Indonesisk | Bahasa Indonesia | `id` |
| 18 | 🇻🇳 Vietnamesisk | Tiếng Việt | `vi` |
| 19 | 🇹🇭 Thailandske | ไทย | `th` |
| 20 | 🇸🇪 Svenske | Svenska | `sv` |
| 21 | 🇩🇰 Dansk | Dansk | `da` |
| 22 | 🇳🇴 Norsk | Norsk | `no` |
| 23 | 🇫🇮 Finsk | Suomi | `fi` |
| 24 | 🇬🇷 Græsk | Ελληνικά | `el` |
| 25 | 🇨🇿 Tjekkisk | Čeština | `cs` |
| 26 | 🇭🇺 Ungarsk | Magyar | `hu` |
| 27 | 🇷🇴 Rumænsk | Română | `ro` |
| 28 | 🇺🇦 Ukrainsk | Українська | `uk` |
| 29 | 🇧🇷 Brasiliansk portugisisk | Português Brasileiro | `pt-BR` |
| 30 | 🇭🇰 Kantonesisk | 粵語 | `zh-HK` |
| 31 | 🇲🇾 Malaysisk | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 Slovakisk | Slovenčina | `sk` |
| 33 | 🇧🇬 Bulgarsk | Български | `bg` |
| 34 | 🇭🇷 Kroatisk | Hrvatski | `hr` |
| 35 | 🇱🇹 Litauisk | Lietuvių | `lt` |
| 36 | 🇱🇻 Lettisk | Latviešu | `lv` |
| 37 | 🇪🇪 Estisk | Eesti | `et` |
| 38 | 🇸🇮 Slovensk | Slovenščina | `sl` |

## Sådan skifter du sprog

### I Chloros Desktop

1. Åbn programindstillingerne
2. Gå til menuen for sprogvalg
3. Vælg dit foretrukne sprog fra listen
4. Brugergrænsefladen opdateres med det samme

### I Chloros CLI

Brug kommandoen `language` til at se det eller ændre sproget i brugergrænsefladen:

```bash
# View current language
chloros-cli language

# Change to Spanish
chloros-cli language es

# Change to Chinese (Simplified)
chloros-cli language zh

# Change to Brazilian Portuguese
chloros-cli language pt-BR

# List all available languages
chloros-cli language --list
```

Se [CLI-dokumentationen](CLI.md) for flere detaljer.

## Dækning

Alle 38 sprog understøttes fuldt ud i:

* **Chloros Desktop** – Komplet oversættelse af brugergrænsefladen
* **Chloros CLI** – Kommandolinjegrænseflade og outputmeddelelser

Python SDK API og dens [referencedokumentation](reference/sdk-reference.md) leveres på engelsk.

Sprogunderstøttelsen sikrer, at brugere over hele verden kan arbejde effektivt på deres modersmål uden hindringer.
