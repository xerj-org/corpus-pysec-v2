# corpus-pysec-v2 — text-record mirror of the PyPI OSV advisory database

One plain-text record per advisory of the [PyPA advisory
database](https://github.com/pypa/advisory-database) (the OSV `PYSEC-*`
database for the PyPI ecosystem), re-laid-out for lexical (BM25-style)
retrieval. Built 2026-10-05 from upstream commit
`3a0bc38e965704f08cad7d269b7012c4a25c1abe`.

Generated one-file-per-advisory from the upstream YAML
(`vulns/<package>/PYSEC-YYYY-NNNN.yaml` → `records/PYSEC-YYYY-NNNN.txt`,
flattened: the package name moves into the record content). No content
was added or removed; the advisory content is only re-laid-out — the
`details` prose in particular is kept verbatim and in full, with nothing
collapsed.

The mirror exists because raw OSV YAML is shape-hostile to lexical
retrieval: most advisories carry only terse fields, so the few with long
prose `details` fields dominate every BM25 query's top-k. One
self-describing record per advisory, each carrying its own identity
terms (id, package, aliases, versions, references) densely, removes that
failure mode.

## Record shape

Each `records/PYSEC-YYYY-NNNN.txt` contains, in order:

1. `PYSEC-YYYY-NNNN <ecosystem>/<package> — <summary>`, where `<summary>`
   is the advisory's `summary` field if present, otherwise the first 100
   characters of `details` with whitespace runs collapsed to single
   spaces.
2. `package: <name>` — from `affected[0].package.name`.
3. `ecosystem: PyPI` — the actual `affected[].package.ecosystem` value
   (all 7,722 affected entries in this database are `PyPI`).
4. `versions affected:` followed by one indented line per range entry
   from every `affected[]` entry: OSV events rendered human-readable
   (`introduced` → `>= v`, `fixed` → `< v`, `last_affected` → `<= v`),
   comma-separated, with the range type appended (`(ECOSYSTEM)`, or
   `(GIT: <repo url>)` for commit ranges). An `affected[]` entry with no
   ranges prints its explicit `versions` list instead.
5. `aliases: GHSA-xxxx-xxxx-xxxx, CVE-YYYY-NNNN` — every alias,
   comma-separated. These terms are what queries carry.
6. `fixed versions: ...` — from a top-level `fixed[]` field if present.
   The upstream schema has no such field (0 of 7,721 advisories), so
   this line never occurs here; fix points appear inside the range
   rendering above as `< <fixed version>` bounds.
7. `references:` followed by each reference URL on its own indented
   line.
8. A blank line, then the full `details` prose, verbatim.

A field absent from an advisory is skipped silently (e.g. 25 advisories
have no aliases; 1 has no references; 20 have an `affected[]` entry with
neither ranges nor versions, so they carry no `versions affected:`
block).

Not re-laid-out into the records (the advisories' other upstream
fields): `modified`, `published`, `withdrawn`, `severity`, `credits`,
`related`, `database_specific`, and the `purl` /
`ecosystem_specific` sub-fields of `affected[]` entries. Everything
listed above is rendered from the upstream values unchanged.

## Provenance and licence

- Upstream: https://github.com/pypa/advisory-database (7,721 advisory
  YAMLs at the pinned commit).
- Licence: **Creative Commons Attribution 4.0 International
  (CC-BY-4.0)**, verified by reading the `LICENSE` file at the pinned
  commit in the source clone — see [LICENSE](./LICENSE) in this
  repository for the attribution notice and the verbatim licence text.
- Build: 2026-10-05. Deterministic — identical input produces identical
  bytes; the mapping is strictly file → file, nothing sorted or merged.
