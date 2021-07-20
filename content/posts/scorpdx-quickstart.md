---
title: "scorPDX Quickstart"
date: 2021-07-20T14:18:28-05:00
draft: true
---

# scorPDX Quickstart

[scorpdx](https://github.com/scorpdx) is a collection of tools and libraries that work with Paradox Interactive games. These are helpful for both Paradox players and developers who create mods and extensions for Paradox games.

Discussion, technical support, and developer chit-chat is available on [Discord](https://discord.gg/FkRPyz6kcD).

## For Players

### ⚒️ironmunge

Savescum in ironman, explore multiple timelines, and never lose a savegame again

[⚒️ironmunge](https://github.com/scorpdx/ironmunge) is a general purpose save manager for Paradox Interactive titles. It automatically manages multiple timelines that you can switch between and explore during a campaign. It currently supports Crusader Kings II (released) and Crusader Kings 3 (to be released Soon™️).

#### [Download on Github](https://github.com/scorpdx/ironmunge/releases)

### PortraitBuilder

Create your own CK2 avatar

![Chinese Empress Unicorn](https://portrait.ga/portrait?dna=00000000aa0&properties=p000000000000000000000000000000000j000&base=horsegfx_female&clothing=horsegfx_female&government=ChineseImperial&titlerank=Emperor)

#### [Download on Github](https://github.com/scorpdx/portraitbuilder/releases)

## For Developers

|Tool|Blurb|
|---------|-------------------|
|[LibCK3](#libck3)|Parse binary (ironman) CK3 saves and turn them into JSON|
|[LibCK3.Tokens](#libck3)|Binary (ironman) token definitions for every CK3 patch|
|[ck3json](#ck3json)|Parse plaintext or binary CK3 saves and turn them into JSON|
|[ck2json](#ck2json)|Parse plaintext CK2 saves and turn them into JSON|
|[PortraitBuilder](#portraitbuilder-dev)|Generate CK2 portraits programmatically|
|[Chronicler](#chronicler)|Extract and format chronicle data from CK2 saves|

### LibCK3

[LibCK3](https://github.com/scorpdx/LibCK3) is a .NET library for reading and parsing Crusader Kings 3 savegames in the binary (Ironman) format. It's actively maintained and updated with new tokens (through [LibCK3.Tokens](https://github.com/scorpdx/LibCK3.Tokens)) every patch.

LibCK3's parser returns an [RFC 8259](https://datatracker.ietf.org/doc/html/rfc8259) compliant JSON string representing a CK3 savegame, allowing developers to view and parse any CK3 save with off-the-shelf JSON tools.

### ck3json

[ck3json](https://github.com/scorpdx/ck3json) is a console application to convert plaintext format Crusader Kings 3 saves into JSON format, as well as melting binary (ironman) saves into plaintext format. It contains a Parsing Expression Grammar definition that can be helpful for developers need to extract data from CK3.

The binary tokens used for melting are not actively updated, but can be extracted from [LibCK3.Tokens](#libck3). The PEG definition should continue to work, barring major changes to the CK3 savegame format.

### ck2json

[ck2json](https://github.com/scorpdx/ck2json) is a console application to convert plaintext format Crusader Kings II saves into JSON format. It contains a Parsing Expression Grammar definition that can be helpful for developers who need to extract data from CK2.

### PortraitBuilder {#portraitbuilder-dev}

[PortraitBuilder](https://github.com/scorpdx/portraitbuilder) is a headless Crusader Kings II portrait generator. It can be used as a standalone desktop application, or can be deployed with a package of graphic assets to support CK2-style portrait rendering in server or library scenarios.

Portrait links can also be used to share custom character portraits using a hosted portrait service.

![Handsome Norman Emperor](https://portrait.ga/portrait?dna=wofssnplqfk&properties=ea0a000i00000d000000000000000000000000&base=normangfx_male&government=feudal&titlerank=emperor)

The above portrait was dynamically rendered from this query: `portrait?dna=wofssnplqfk&properties=ea0a000i00000d000000000000000000000000&base=normangfx_male&government=feudal&titlerank=emperor`

PortraitBuilder was forked from [Romain Quinio's Winforms portrait builder](https://github.com/rquinio/PortraitBuilder) and rewritten with SkiaSharp to render into a buffer rather than using Windows GDI+. 

### Chronicler

[Chronicler](https://github.com/scorpdx/chronicler) is a .NET library for parsing and extracting Chronicles from a Crusader Kings II _JSON_ save. When combined with [ck2json](#ck2json), it can be used to generate timelines and other flavor content from a CK2 save.

#### [View a demo timeline](https://jzebedee.bitbucket.io/)