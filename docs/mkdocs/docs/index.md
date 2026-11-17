<div align="center">
  <img src="https://hanhy06.github.io/embellish-chat/assets/logo.svg" alt="Embellish Chat" width="460">
</div>

<div align="center">
  <a href="https://modrinth.com/mod/embellish-chat">
    <img alt="Download" src="https://img.shields.io/modrinth/dt/embellish-chat?label=Modrinth&logo=modrinth&style=flat-square">
  </a>
  <a href="https://discord.gg/nsca3Na9K8">
    <img alt="Discord" src="https://img.shields.io/badge/Discord-5865F2?logo=discord&logoColor=white&style=flat-square">
  </a>
  <a href="https://hanhy06.github.io/embellish-chat/">
    <img alt="Wiki" src="https://img.shields.io/badge/Wiki-blueviolet?logo=github&style=flat-square">
  </a>
</div>

# Embellish Chat

Embellish Chat expands Minecraft chat with rich formatting, clickable text, hover text, item, inventory, and ender chest
showcases, icons, and configurable mentions. Most features are server-side, so players can use the enhanced chat
experience without installing the mod on their client.

!!! tip "Start here"
    Server owners should begin with [Configuration](embellish-chat/Configuration.md). Players who only need syntax can use
    the tables below.

## What It Adds

| Area        | Description                                                                                                          |
|-------------|----------------------------------------------------------------------------------------------------------------------|
| Style       | Markdown-like formatting, colors, gradients, fonts, links, hover text, click actions, and icons.                     |
| Mention     | Configurable notifications for players, nearby players, teams, groups, worlds, channels, permissions, and selectors. |
| Showcase    | Share view-only item, inventory, or ender chest screens in chat.                                                     |
| Admin Tools | Reload configs, test regular expressions, and manage mod feature bans.                                               |

## Recommended Reading Path

| Goal                                    | Page                                             |
|-----------------------------------------|--------------------------------------------------|
| Install and understand the config files | [Configuration](embellish-chat/Configuration.md) |
| Use player and admin commands           | [Commands](embellish-chat/Commands.md)           |
| Create or edit chat styles              | [Style System](style/StyleSystem.md)             |
| Create or edit mentions                 | [Mention System](mention/MentionSystem.md)       |
| Check mod compatibility                 | [Compatibility](embellish-chat/Compatibility.md) |

## Default Chat Syntax

| Feature       | Syntax             | Example                       |
|---------------|--------------------|-------------------------------|
| Bold          | `**Text**`         | `**Important**`               |
| Italic        | `_Text_`           | `_Whisper_`                   |
| Underline     | `__Text__`         | `__Title__`                   |
| Strikethrough | `~~Text~~`         | `~~Canceled~~`                |
| Obfuscated    | `\|\|Text\|\|`     | `\|\|Secret\|\|`              |
| Hex color     | `[Text]<#Hex>`     | `[Sky]<#00AAFF>`              |
| Preset color  | `[Text]<Preset>`   | `[Warning]<red>`              |
| Gradient      | `[Text]<#Hex#Hex>` | `[Fire]<#FFFF00#FF0000>`      |
| Rainbow       | `[Text]<RAINBOW>`  | `[Magic]<RAINBOW>`            |
| Link          | `[Text](URL)`      | `[Docs](https://example.com)` |
| Font          | `[Text]{Font ID}`  | `[Rune]{minecraft:alt}`       |
| Item          | `[i]`              | `Look at my [i]`              |
| Inventory     | `[inv]`            | `Check my [inv]`              |
| Ender chest   | `[end]`            | `Check my [end]`              |
| Icon          | `:Icon:`           | `:heart:`                     |

## Default Mention Syntax

| Target          | Example                       | Result                                                 |
|-----------------|-------------------------------|--------------------------------------------------------|
| Player          | `@Steve`                      | Mentions one player by name or supported nickname.     |
| Everyone        | `@everyone`                   | Mentions all online players.                           |
| Nearby players  | `@here`                       | Mentions players near the sender.                      |
| Team            | `@team(red)`                  | Mentions players in the scoreboard team named `red`.   |
| LuckPerms group | `@group(admin)`               | Mentions players in the LuckPerms group named `admin`. |
| World           | `@world(minecraft:overworld)` | Mentions players in the selected world.                |

!!! note "Configuration decides the final behavior"
    The tables show the built-in defaults. Server owners can change patterns, permissions, colors, sounds, cooldowns, and
    targeting rules in the config files.

## Links

| Resource    | Link                                                                      |
|-------------|---------------------------------------------------------------------------|
| Download    | [Modrinth](https://modrinth.com/mod/embellish-chat)                       |
| CurseForge  | [CurseForge](https://www.curseforge.com/minecraft/mc-mods/embellish-chat) |
| Source Code | [GitHub](https://github.com/hanhy06/embellish-chat)                       |
| Issues      | [Bug Tracker](https://github.com/hanhy06/embellish-chat/issues)           |
