# ChartID — `space.chartid.*`

An open AT Protocol Lexicon for astrological charts as self-sovereign identity records.

**Steward domain:** `chartid.space`  
**Lexicon namespace:** `space.chartid`  
**Status:** RFC / Pre-release — open for comment

---

## What this is

Every AT Protocol user has a DID. Their astrological chart — if they want one — should live in their own repo, portable across the entire Atmosphere, not locked inside any single app.

`space.chartid.*` defines the data model. No app owns the records. No app owns the standard.

---

## Protocol architecture

### Records live in user repos

A natal chart record at `space.chartid.natalChart` in a user's repo is:

- Signed by their repo key
- Readable by any Atmosphere app with Lexicon support
- Deletable by the user at any time
- Not migrated when they switch PDS — it's their data

A record looks like:

```json
{
  "$type": "space.chartid.natalChart",
  "schemaVersion": "1.0.0",
  "traditions": ["western", "jyotish"],
  "birthDatetime": "1990-04-15T14:32:00Z",
  "birthPlace": {
    "placeName": "Oakland, CA, USA",
    "coordinate": { "latitude": "37.804363", "longitude": "-122.271111" },
    "timezone": "America/Los_Angeles"
  },
  "timeConfidence": "exact",
  "roddenRating": "AA",
  "western": { ... },
  "jyotish": { ... },
  "createdAt": "2025-01-01T00:00:00Z"
}
```

### DID-level identity

`subjectDID` optionally links the chart to an AT identity. A person can point their own DID to their own chart — or keep them separate. Consent is explicit, not assumed.

`attestedBy` is an array of astrologer DIDs. Multiple practitioners can attest to chart accuracy without creating a separate attestation service. Each attesting DID is expected to publish a corresponding attestation record of their own.

### Multi-tradition by design

`traditions` is an array. A single record can carry both Western tropical and Jyotish sidereal data:

```json
{
  "traditions": ["western", "jyotish"],
  "western": { "houseSystem": "Placidus", "planets": [...] },
  "jyotish": { "ayanamsa": "Lahiri", "ayanamsaDegrees": "24.112300", "planets": [...], "vargaCharts": [...] }
}
```

Apps that only understand one tradition ignore the other. The record serves both without forking.

### Unknown birth times are valid records

`birthDatetime` is optional. `timeConfidence` signals the reliability of the time:

- `exact` — birth certificate / hospital record
- `approximate` — family memory, close estimate
- `unknown` — date known, time not
- `rectified` — time derived by rectification (see `rectificationNote`)

A chart with `timeConfidence: "unknown"` is a valid, useful partial record — rising sign and houses are absent, planets are present.

---

## Lexicon files

| File | Description |
|------|-------------|
| `space.chartid.defs` | Shared types: `bodyPosition`, `houseCusp`, `westernData`, `jyotishData`, `vargaChart`, `dashaTree`, etc. |
| `space.chartid.natalChart` | Birth chart. `key: "tid"` — users can hold multiple (Western, Jyotish, rectified versions). |
| `space.chartid.eventChart` | Moment in time: eclipses, launches, IPOs. `eventDatetime` required. |
| `space.chartid.entityChart` | Organization, country, institution. `entityName` required. |
| `space.chartid.horaryChart` | Horary / Prashna: a sincere question at a moment. `questionDatetime` required. |
| `space.chartid.electionalChart` | Elected / Muhurta: chosen auspicious timing. `electionalDatetime` required. |

---

## Design decisions

**No floats in ATProto Lexicon.** All degree values are decimal degree strings (`"254.378500"`). Explicit, lossless, no float precision surprises.

**`schemaVersion: "1.0.0"` is a const field.** Breaking schema changes will bump to 2.0.0 with a new lexicon file, not in-place mutation. Old records stay valid under old readers.

**`extensions: unknown` at top level and within each tradition block.** Tradition-specific fields not yet in spec go here without breaking existing record validation. This is how Hellenistic lots, Uranian midpoints, and shadbala scores get added incrementally.

**`traditions` is an array, not an enum.** One record, multiple simultaneous traditions. Compare western and Jyotish placements in a single query.

**`houseSystem` and `ayanamsa` are explicit fields**, never inferred from context or packed into a planet position string. A chart without a declared house system is ambiguous. These fields make it unambiguous.

---

## Adoption

### Implementing this Lexicon

1. Declare `space.chartid.natalChart` as a supported collection in your PDS config
2. Point to `lexicons/` in this repo for schema validation
3. Write records via `com.atproto.repo.createRecord` with `collection: "space.chartid.natalChart"`

### Live site

[chartid.space](https://chartid.space) — protocol overview, demo `space.chartid.natalChart` record rendered as a North Indian Jyotish chart, full Lexicon namespace listing. Hosted on Cloudflare Pages from this repo (`index.html`).

### First working implementation

[Artiji](https://artiji.xyz) — hyperpersonalized astrological greeting cards — is the first app writing `space.chartid.natalChart` records. Charts generated at card creation can be saved to the user's ATProto repo via "Save chart to ATProto" at card creation.

### Reference calculator

[jyotish-mcp](https://sandboxlabs.ai/jyotish-calc) — Swiss Ephemeris backed TypeScript MCP server producing Jyotish chart data in ChartID format. Lahiri ayanamsa. Sidereal ascendant math. Vimshottari dasha calculation.

---

## RFC

Open discussion: [`bluesky-social/atproto` Discussions](https://github.com/bluesky-social/atproto/discussions)

Feedback on: namespace choice, field naming, extension model, multi-tradition record vs separate records per tradition.

---

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md).

Schema changes follow semantic versioning:  
- Patch: clarifications, description improvements  
- Minor: new optional fields, new `knownValues` entries  
- Major: required field additions, type changes, field removals

All changes via PR with rationale. Breaking changes require RFC comment period before merge.

---

## License

Lexicon definitions: CC0 — no rights reserved. Use freely.
