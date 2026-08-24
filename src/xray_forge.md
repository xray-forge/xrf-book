# X-Ray Forge

X-Ray Forge (XRF) is a collection of development projects for [X-Ray 16](https://github.com/OpenXRay/xray-16) engine. It
provides a TypeScript scripting layer and the tooling around it for building, testing, and maintaining mods.

## What XRF includes

- A TypeScript rewrite of the OpenXRay scripting layer, compiled to Lua.
- Build-time tools for scripts, configs, UI forms, translations, and resources.
- A TypeScript SDK for the OpenXRay engine API.
- The XRF CLI for building projects and working with a local game installation.
- XRF Tools: a command-line toolset and a desktop development application.
- This book and the SDK API reference.

## Start here

- Setting up or building an XRF project? Read [Installation](./INSTALLATION.md), then
  [Building](./xrf/building/building.md).
- Writing or changing scripts? Start with the [Script engine](./script_engine/script_engine.md) and
  [Schemes](./script_engine/schemes/schemes.md).
- Looking for a command? See the [XRF CLI](./xrf/cli/cli.md) or [XRF Tools CLI](./tools/cli/cli.md).
- Working with game-data files? See [XRF Tools](./tools/tools.md).
- Investigating runtime behavior? Read [Game engine](./game_engine/game_engine.md) and
  [Debugging](./debugging/debugging.md).

## Repositories

### Core development

- [XRF engine](https://github.com/xray-forge/stalker-xrf-engine) — TypeScript scripting layer, build pipeline, and CLI.
- [X-Ray 16 TypeScript SDK](https://github.com/xray-forge/stalker-xrf-xray16-sdk) — TypeScript API for OpenXRay.
- [XRF tools](https://github.com/xray-forge/stalker-xrf-tools) — command-line tools and desktop application.
- [XRF tools end-to-end tests](https://github.com/xray-forge/xrf-tools-e2e) — end-to-end tests for the XRF Tools CLI.
- [XRF book](https://github.com/xray-forge/stalker-xrf-book) — this documentation.

### Distribution

- [XRF binaries](https://github.com/xray-forge/stalker-xrf-bin) — packaged engine and tool binaries.

### Resource packs

- [Base assets](https://gitlab.com/xray-forge/xrf-resources-base)
- [Extended assets](https://gitlab.com/xray-forge/xrf-resources-extended)
- [English locale](https://gitlab.com/xray-forge/xrf-resources-locale-en)
- [Ukrainian locale](https://gitlab.com/xray-forge/xrf-resources-locale-ukr)
- [Russian locale](https://gitlab.com/xray-forge/xrf-resources-locale-ru)
