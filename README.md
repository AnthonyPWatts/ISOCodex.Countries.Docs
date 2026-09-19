# ISOCodex.Countries

Strongly typed country codes, country and territory metadata, subdivisions, display names and explicit lookups for .NET.

This repository contains public documentation, runnable examples and the consumer issue tracker. The library implementation and development history are maintained in a separate private repository. Install the published packages from NuGet; clone this repository to run the examples against published NuGet packages.

[Full documentation and support](https://github.com/AnthonyPWatts/ISOCodex.Countries.Docs).

## Try it locally

With the .NET 10 SDK installed, clone this public repository and run:

```powershell
git clone https://github.com/AnthonyPWatts/ISOCodex.Countries.Docs.git
cd ISOCodex.Countries.Docs
dotnet run --project samples/CountryLookup.Console
```

[Browse all runnable examples](samples/README.md). They use published NuGet packages and need no private repository access.

## Install

Current release: **1.1.1**. [View on NuGet](https://www.nuget.org/packages/ISOCodex.Countries/1.1.1).

```powershell
dotnet add package ISOCodex.Countries --version 1.1.1
```

## What it provides

- 249 current country and territory entries, including alpha-2, alpha-3 and numeric identifiers.
- 2,739 selected CLDR-derived display names and 5,027 regular subdivision entries across 200 countries.
- Explicit alias handling, parse/lookup results and System.Text.Json converters.
- Packaged data with no hidden runtime network calls.

## Documentation

- [Consumer guide](https://github.com/AnthonyPWatts/ISOCodex.Countries.Docs/blob/main/docs/usage.md)
- [Framework compatibility and verification](https://github.com/AnthonyPWatts/ISOCodex.Countries.Docs/blob/main/docs/compatibility.md)
- [Release notes](https://github.com/AnthonyPWatts/ISOCodex.Countries.Docs/blob/main/CHANGELOG.md)
- [Questions, bug reports and feature requests](https://github.com/AnthonyPWatts/ISOCodex.Countries.Docs/issues)
- [Discussion](https://github.com/AnthonyPWatts/ISOCodex.Countries.Docs/discussions)

- [Currency documentation](https://github.com/AnthonyPWatts/ISOCodex.Currency.Docs)
- [Addressing documentation](https://github.com/AnthonyPWatts/ISOCodex.Addressing.Docs)

- [Data sources and limitations](https://github.com/AnthonyPWatts/ISOCodex.Countries.Docs/blob/main/docs/data-sources.md)
- [Third-party notices](https://github.com/AnthonyPWatts/ISOCodex.Countries.Docs/blob/main/THIRD-PARTY-NOTICES.md)

## Feedback

Include package versions, target framework, expected behaviour and a small reproducible consumer example when reporting a problem. Data corrections should include a reliable source and the date checked.

## Licence and source availability

The documentation and sample applications are provided under the [MIT licence](https://github.com/AnthonyPWatts/ISOCodex.Countries.Docs/blob/main/LICENSE). Published packages retain their declared licences. Private source hosting does not revoke rights already granted for earlier distributions. ISOCodex is not an official ISO product or endorsed by ISO.
