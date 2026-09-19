# Consumer guide

## Quick Start

```csharp
using ISOCodex.Countries;

CountryAlpha2Code countryCode = CountryAlpha2Code.Parse("gb");
CountryInfo country = CountryRegistry.GetByAlpha2(countryCode);

Console.WriteLine(country.Alpha2);            // GB
Console.WriteLine(country.EnglishShortName);  // United Kingdom
```

For boundary input, prefer `TryParse` or `CountryRegistry.Lookup`:

```csharp
CountryCodeLookupResult result = CountryRegistry.Lookup("826");

if (result.Success)
{
    Console.WriteLine(result.Country!.EnglishShortName);
}
```

For invalid or unresolved boundary input, keep the failure reason with the row/request instead of collapsing everything into "not found":

```csharp
CountryCodeLookupResult result = CountryRegistry.Lookup("EU");

if (!result.Success)
{
    Console.WriteLine(result.FailureReason); // ReservedButNotCountry
    Console.WriteLine(result.NormalizedInput); // EU
}
```

## Choosing The Right Lookup API

Different values need different APIs. The package keeps these paths separate so consumers do not accidentally treat names, aliases, and identifiers as the same thing.

| Input shape | API | Example result |
|---|---|---|
| Canonical country code | `CountryRegistry.Lookup` | `GB`, `GBR`, `826` resolve to United Kingdom |
| Display name | `CountryNameRegistry.LookupDisplayName` | `DE` + `de` returns `Deutschland` |
| Common or deprecated alias | `CountryAliasRegistry.Lookup` | `Great Britain` resolves to `GB`; `UK` can resolve explicitly |
| Special alpha-2-shaped value | `CountryCodeElementRegistry.TryGetByAlpha2` | `EU` returns European Union metadata |
| Subdivision code | `CountrySubdivisionRegistry.Lookup` | `GB-ENG` resolves to England |

`CountryRegistry.Lookup` is deliberately code-oriented. It does not resolve `Britain`, `Germany`, `Deutschland`, or `UK` as a country unless the caller opts into an alias or display-name API.

## Syntax And Registry Membership

Syntax validation and country lookup answer different questions.

`CountryAlpha2Code.Parse` and `CountryAlpha2Code.TryParse` only validate and normalise the alpha-2 shape. They do not prove that the value is a current country known to this package. For example, `UK` is syntactically alpha-2, but the canonical current-country code for the United Kingdom is `GB`.

When consumer code needs deliverability or current-country semantics, validate the syntax and then check the country registry:

```csharp
if (!CountryAlpha2Code.TryParse(input, out CountryAlpha2Code code))
{
    // Not syntactically alpha-2.
}
else if (!CountryRegistry.TryGetByAlpha2(code, out CountryInfo? country))
{
    // Syntactically alpha-2, but not a current country known to the registry.
}
else
{
    // Current country metadata is available.
}
```

Use `CountryCodeElementRegistry` when an alpha-2-shaped value is meaningful but is not a current country entry, such as `EU`, `QO`, `XA`, `XB`, `XK`, or `ZZ`.

## Country Codes

The foundation API supports value objects for:

- `CountryAlpha2Code`, such as `GB`
- `CountryAlpha3Code`, such as `GBR`
- `CountryNumericCode`, such as `826` or `008`

Alpha codes are stored in canonical uppercase. Numeric codes are stored as three-character strings so leading zeroes are preserved.

Mixed lookup is deterministic: two ASCII letters are treated as alpha-2, three ASCII letters as alpha-3, and three digits as numeric. `UK` is recognised as syntactically valid alpha-2 input, but it is not silently treated as canonical `GB`.

`EU`, `QO`, `XA`, `XB`, `XK`, and `ZZ` are also syntactically valid alpha-2 shapes. They are not current country entries in this package. Mixed lookup returns `ReservedButNotCountry` for these known non-country or special-purpose codes, so import pipelines can distinguish them from invalid syntax and arbitrary unknown codes.

Subdivision-shaped input is outside country-code lookup. `CountryRegistry.Lookup("GB-ENG")` returns `Unsupported`; use `CountrySubdivisionRegistry.Lookup` for subdivision codes.

Typical mixed lookup outcomes:

| Input | Success | Failure reason | Notes |
|---|---:|---|---|
| `GB` | Yes | | Canonical alpha-2 country code |
| `gbr` | Yes | | Canonicalised alpha-3 country code |
| `008` | Yes | | Numeric code with leading zero preserved |
| `UK` | No | `Unknown` | Alias-like alpha-2 shape; use `CountryAliasRegistry` if desired |
| `EU` | No | `ReservedButNotCountry` | Known special code element |
| `GB-ENG` | No | `Unsupported` | Valid subdivision shape; use subdivision lookup |
| `1!` | No | `InvalidSyntax` | Not a supported country-code shape |

## Display Names

Country codes are identifiers. Display names are human-readable metadata.

The package includes a generated `CountryNameRegistry` backed by selected Unicode CLDR 48.2 locale files. Names are stored as Unicode, normalised to NFC, and preserved in their source script. No transliteration is performed, and right-to-left text is preserved rather than reversed.

```csharp
CountryAlpha2Code de = CountryAlpha2Code.Parse("DE");

string english = CountryNameRegistry.GetEnglishShortName(de);
// Germany

CountryNameRegistry.TryGetDisplayName(de, "de", out CountryDisplayName? german);
// Deutschland

CountryDisplayNameLookupResult portuguese = CountryNameRegistry.LookupDisplayName(de, "pt-BR");
Console.WriteLine(portuguese.UsedFallback);
Console.WriteLine(portuguese.ResolvedLanguageTag);
```

Fallback is visible by design:

1. exact requested language tag, such as `pt-BR`;
2. parent language tag, such as `pt`;
3. English display name from generated CLDR data;
4. existing `CountryInfo.EnglishShortName`.

The lookup result tells you what was requested, what was actually returned, and whether fallback was used.

Unicode names round-trip as normal .NET strings:

```csharp
CountryNameRegistry.TryGetDisplayName(CountryAlpha2Code.Parse("JP"), "ja", out var japanese);
Console.WriteLine(japanese?.Name); // 日本
```

Endonym coverage is deliberately source-limited. The package exposes `GetEndonyms` and `TryGetPrimaryEndonym`, but it does not add a `ToLocal()` shortcut because many countries have multiple local names, languages, and scripts.

## Alias And Special-Code Lookup

Alias lookup is explicit and opt-in. `CountryRegistry.Lookup` remains canonical-code-focused and does not resolve names or aliases.

```csharp
CountryAliasLookupResult alias = CountryAliasRegistry.Lookup("Great Britain");

if (alias.Success)
{
    Console.WriteLine(alias.Country!.Alpha2); // GB
}
```

Ambiguous aliases are reported rather than guessed:

```csharp
CountryAliasLookupResult alias = CountryAliasRegistry.Lookup("AN");

if (alias.Ambiguous)
{
    foreach (CountryInfo candidate in alias.Candidates)
    {
        Console.WriteLine(candidate.Alpha2);
    }
}
```

Known special country-code-shaped elements are exposed separately:

```csharp
if (CountryCodeElementRegistry.TryGetByAlpha2("EU", out var eu))
{
    Console.WriteLine(eu.DisplayName); // European Union
    Console.WriteLine(eu.Kind);        // RegionGrouping
}
```

## Persistence Guidance

For most applications, store canonical alpha-2 codes such as `GB`.

Store alpha-3 or numeric codes only when an integration requires them.

Do not store display names as identifiers.

If preserving what a user originally typed matters, store the original input separately from the canonical code.

## Subdivision Codes

`CountrySubdivisionCode` models ISO 3166-2-style codes such as `GB-ENG`, `US-CA`, and `IE-D`.

Syntax validity does not mean the subdivision is known. Use `CountrySubdivisionRegistry.TryGetByCode` when you need to check against the CLDR-derived seed registry.

```csharp
CountrySubdivisionLookupResult subdivision = CountrySubdivisionRegistry.Lookup("GB-ENG");

if (subdivision.Success)
{
    Console.WriteLine(subdivision.Subdivision!.EnglishName); // England
}
```

## JSON

Converters are included for the value objects:

```csharp
using System.Text.Json;
using ISOCodex.Countries;

var options = new JsonSerializerOptions();
options.Converters.Add(new CountryAlpha2CodeJsonConverter());
options.Converters.Add(new CountryAlpha3CodeJsonConverter());
options.Converters.Add(new CountryNumericCodeJsonConverter());
options.Converters.Add(new CountrySubdivisionCodeJsonConverter());

string json = JsonSerializer.Serialize(CountryAlpha2Code.Parse("gb"), options);
CountryAlpha2Code code = JsonSerializer.Deserialize<CountryAlpha2Code>("\"GB\"", options);
```

Converters serialise to canonical strings and reject invalid values during deserialisation.

Default value objects serialise as empty strings. A default value object is not a meaningful canonical country code; treat it as an uninitialised value rather than persisted domain data.

## CSV Import Validation

For import pipelines, keep row-level errors rather than throwing for normal invalid input:

```csharp
foreach (string input in countryColumnValues)
{
    CountryCodeLookupResult result = CountryRegistry.Lookup(input);

    if (!result.Success)
    {
        // Store result.FailureReason against the row.
        // InvalidSyntax, Unknown, and ReservedButNotCountry are distinct cases.
        continue;
    }

    CountryAlpha2Code canonicalCode = result.Country!.Alpha2;
}
```


## Country Dropdowns

For simple dropdowns, use `CountryRegistry.All` and store `CountryInfo.Alpha2` as the canonical value. Display names are useful labels, not identifiers.

```csharp
var options = CountryRegistry.All
    .OrderBy(country => country.EnglishShortName)
    .Select(country => new { Value = country.Alpha2.Value, Label = country.EnglishShortName });
```

## Data Sources And Limitations

The country seed data is derived from Unicode CLDR 48.2 and includes 249 current ISO-style country and territory entries with alpha-2, alpha-3, numeric, and English display-name metadata. Country display names are CLDR-derived labels, not official government names or geopolitical authority.

The display-name registry currently includes 2,739 generated country display names across selected CLDR locales. Localised names may use non-Latin scripts and right-to-left writing systems.

Special code elements such as `EU`, `QO`, `XA`, `XB`, `XK`, and `ZZ` are generated from CLDR display metadata but are not equivalent to current countries. Deprecated territory aliases such as `UK` are available only through `CountryAliasRegistry`.

Subdivision seed data is derived from Unicode CLDR 48.2 and includes 5,027 regular subdivision entries across 200 countries. It provides code and English display-name lookup. Subdivision type metadata is `Unknown` unless a specific reviewed overlay exists.

This package is not a redistributed official ISO dataset.

Current data version:

```csharp
Console.WriteLine(CountryDataVersion.Identifier);
Console.WriteLine(CountryDataVersion.CheckedOn);
Console.WriteLine(CountryDataVersion.Description);
```

No runtime code makes hidden network calls. The runtime package uses checked-in compiled seed data rather than loose external files.

Data-source policy and limitations are documented in [data sources](data-sources.md).
