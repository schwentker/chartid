# Contributing to ChartID

Thanks for your interest in contributing to `space.chartid.*`. This is an open AT Protocol Lexicon standard — contributions from astrologers, developers, and protocol designers are welcome.

---

## How to contribute

1. **Open an issue first** — describe the proposed change and why. Breaking schema changes require an RFC comment period before merge.
2. **Fork and branch** — create a feature branch off `main`.
3. **Make changes** — follow the guidelines below.
4. **Open a PR** — include rationale, any affected lexicon files, and README/PRIVACY updates as needed.

---

## Schema versioning

Changes follow semantic versioning:

| Type | Version bump | Examples |
|------|-------------|---------|
| Patch | `1.0.x` | Description improvements, `knownValues` additions |
| Minor | `1.x.0` | New optional fields |
| Major | `x.0.0` | Required field additions, type changes, field removals |

Breaking changes (`schemaVersion` bump to `2.0.0`) require a new lexicon file. Old records must remain valid under old readers.

---

## Lexicon style rules

- **No floats.** All degree values are decimal degree strings (`"254.378500"`). ATProto Lexicon does not have a float type; strings are explicit and lossless.
- **`knownValues` not `enum`.** Prefer `knownValues` over closed `enum` so future values don't break old validators.
- **Explicit house systems and ayanamsas.** Never infer from context. Always declare `houseSystem` and `ayanamsa` explicitly.
- **`unknown` as extension point.** Use `"type": "unknown"` for extension fields (`extensions`, `vargaCharts`, `dashaTree`). Tradition-specific extras that aren't yet in spec belong there.
- **`key: "self"` for singletons.** Records of which there should only be one per user (like `settings`) use `key: "self"`.

---

## Privacy review checklist

Before proposing any new field, run through this checklist:

- [ ] Does this field contain or directly derive PII (name, exact birth time, location, identifier)?
- [ ] If yes: is there a privacy/encryption mechanism documented in PRIVACY.md?
- [ ] Could an app misuse this field to track users across contexts?
- [ ] Is the field actually necessary, or can it be optional with a documented default?
- [ ] Does the field description mention privacy implications where relevant?

**Example:** `exactBirthTime` is useful, but should default to `unknown` via `birthTimeTrust` and support encryption via `encryptedData`. A field that embeds a real name without a visibility control would require a privacy review before acceptance.

---

## Testing & validation

After editing lexicon JSON:

1. Validate all JSON is well-formed (`jq . lexicons/space/chartid/*.json`)
2. Check that required fields are defined and types match ATProto Lexicon spec
3. Test `natalChart` with `visibility: "private"` + encrypted `encryptedData` payload
4. Test `chartVerification` links correctly to a `natalChart` via `chartUri`
5. Test `settings` reads as singleton (only one per user via `key: "self"`)

---

## License

Lexicon definitions: CC0 — no rights reserved. Use freely. By contributing, you agree your contributions are released under CC0.
