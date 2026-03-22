# Understøttede sprog

Chloros tilbyder fuld understøttelse af brugergrænsefladen på **38 sprog verden over**, hvilket gør det tilgængeligt for brugere over hele kloden. Du kan skifte sprog med det samme på alle grænseflader: Desktop, Browser, CLI og Python SDK.

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
| 19 | 🇹🇭 Thai | ไทย | `th` |
| 20 | 🇸🇪 Svensk | Svenska | `sv` |
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
| 31 | 🇲🇾 Malay | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 Slovakisk | Slovenčina | `sk` |
| 33 | 🇧🇬 Bulgarsk | Български | `bg` |
| 34 | 🇭🇷 Kroatisk | Hrvatski | `hr` |
| 35 | 🇱🇹 Litauisk | Lietuvių | `lt` |
| 36 | 🇱🇻 Lettisk | Latviešu | `lv` |
| 37 | 🇪🇪 Estisk | Eesti | `et` |
| 38 | 🇸🇮 Slovensk | Slovenščina | `sl` |

## Sådan ændrer du sprog

### I Chloros Desktop/Browser

1. Åbn programindstillingerne
2. Gå til sprogvalgmenuen
3. Vælg dit foretrukne sprog fra listen
4. Brugergrænsefladen opdateres øjeblikkeligt

### I Chloros CLI

Brug kommandoen `language` til at se eller ændre sproget i CLI-grænsefladen:

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

### I Chloros Python SDK

Indstil sprogparameteren, når du initialiserer SDK, for at få meddelelser og output på dit foretrukne sprog.

## Dækning

Alle 38 sprog understøttes fuldt ud på:

* **Chloros Desktop** - Komplet oversættelse af GUI
* **Chloros Browser** - Webgrænseflade på alle sprog
* **Chloros CLI** - Kommandolinjegrænseflade og outputmeddelelser
* **Chloros Python SDK** - API-meddelelser og dokumentation

Sprogunderstøttelse sikrer, at brugere over hele verden kan arbejde effektivt på deres modersmål uden barrierer.
