# bundled-pm-change-ruby-4-0-10

## Feature exercised

Verifies that Mend SCA correctly detects Ruby dependencies when
the project targets Ruby 4.0.10, whose bundled Bundler version
(2.6.9) differs from whatever Mend's default `install-tool`
provisioning would otherwise choose. The probe exercises the
`scanSettings.versioning` pin mechanism for Bucket A PMs.

## Probe metadata

| Field                  | Value                                     |
|------------------------|-------------------------------------------|
| Pattern                | `bundled-pm-change-ruby-4-0-10`           |
| Category               | `bundled_pm_change`                       |
| Ruby version           | 4.0.10                                    |
| Bundler version        | 2.6.9 (bundled with Ruby 4.0.10)          |
| Schema version         | 1.2                                       |
| `pm_version_under_test`| `2.6.9`                                   |
| Generated at           | 2026-11-19T00:00:00+00:00                 |

## Dependency layout

```
bundled-pm-change-ruby-4-0-10 (root)
├── json 2.7.1          [main]
├── rack 3.1.7          [main]
├── rake 13.2.1         [main]
└── minitest 5.25.1     [test]
```

All four gems are direct dependencies with no transitive rubygems
dependencies. Each resolves from `https://rubygems.org/`.

## Expected dependency tree

- `json`, `rack`, `rake` appear in the main group with
  `source: "registry"`.
- `minitest` appears in the test group with `source: "registry"`.
- No transitive dependencies — all four gems are standalone.
- `project_metadata.bundler_version` is `"2.6.9"`.
- `project_metadata.ruby_version` is `"ruby 4.0.10p0"`.
- `project_metadata.platforms` is `["ruby", "x86_64-linux"]`.

## Mend config

Bucket A — default-emit with `bundler` and `ruby` pinned.

`.whitesource` `scanSettings.versioning`:
```json
{
  "bundler": "2.6.9",
  "ruby": "4.0.10"
}
```

`configMode` is `"AUTO"` (no `whitesource.config` ships with this
probe, so LOCAL would be incorrect per the whitesource-config.md
rule).

Both pins are exact versions sourced from the lockfile:
- `bundler` from `BUNDLED WITH 2.6.9`.
- `ruby` from `RUBY VERSION ruby 4.0.10p0` (dropping the `p0`
  suffix as `install-tool` expects only the numeric part).

## Mend failure modes to watch

1. `scanSettings.versioning` pin ignored — Mend installs a
   different Bundler version and may produce a different lockfile
   parse result if Bundler's lockfile format changed between
   versions.
2. `RUBY VERSION` or `BUNDLED WITH` sections not reflected in
   `project_metadata` — the downstream comparator checks both.
3. Lockfile format for Bundler 2.6.9 not recognized — Mend
   truncates the lockfile or emits 0 deps.
4. Fallback to manifest-only (pre-step off) parsing — loses
   all deps because there is no lockfile at scan time.

## Notes

Ruby 4.0.10 is a future release; Bundler 2.6.9 is the projected
bundled version based on the progression observed in earlier
Ruby 3.x patch series. Adjust `pm_version_under_test` and the
`.whitesource` pin if the actual release ships a different
Bundler version.

This probe is a **single probe** (not a pair) — see
`version-variations.md#version-lang-ruby4.0.10-bundled-bundler`
for the catalog cross-reference.
