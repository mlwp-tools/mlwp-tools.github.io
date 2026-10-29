---
title: "Roadmap"
description: "Release plan for mlwp-data-specs, mlwp-data-loaders and mxalign: what goes into each release, in what order, and what is still open."
date-modified: 2026-10-29
---

::: {.callout-note}
This is a working draft and changes as pull requests land. Issue and PR numbers link to GitHub: `specs#36` is PR 36 in mlwp-data-specs, `loaders#27` is PR 27 in mlwp-data-loaders.
:::

The three core packages depend on each other in one direction: mlwp-data-specs ← mlwp-data-loaders ← mxalign (mxalign also imports mlwp-data-specs directly). Releases go in that order, so each package is released only after what it depends on. Below, "specs" and "loaders" are short for mlwp-data-specs and mlwp-data-loaders.

## Current state

| Package | Latest tag | Versioning | Notes |
|-----------|------|----------|------------------------------|
| [mlwp-data-specs](https://github.com/mlwp-tools/mlwp-data-specs) | v0.1.0 | setuptools-scm (tag-based) | Space, Time and Uncertainty traits, and the validator. `main` has the checks restructure ([specs#36], merged 2026-10-29); [specs#38] and [specs#35] are open ([step 3](#step-3)). |
| [mlwp-data-loaders](https://github.com/mlwp-tools/mlwp-data-loaders) | none | setuptools-scm (tag-based) | Loaders for anemoi, HARP and IFS data; the PyPI workflow exists. All four loader rewrites are merged on `main` (2026-10-28); see [step 1](#step-1). |
| [mxalign](https://github.com/mlwp-tools/mxalign) | v0.1.0 | static `version = "0.1.0"` in `pyproject.toml` | Loading and validation fully moved to loaders and specs on `main` ([mxalign#27], [mxalign#47]); still pins specs and loaders by git. |

## Release sequence

| Step | Release | Content | Depends on |
|---|-------------|----------------------------------------|----------|
| 1 | **mlwp-data-loaders v0.1.0** | First release. All bundled loaders (anemoi, HARP, IFS) are rewritten so their dask graphs stay small ([loaders#24], [loaders#25], [loaders#27], [loaders#30]); the IFS forecast loader came in [loaders#12]. Also `loader` as a python callable ([loaders#28], closes [loaders#14]), and the trait-axis conventions of ADR-001 ([loaders#29]) with their implementation. No ensemble changes beyond what [loaders#27] already brings ([item 1](#item-1)). | specs v0.1.0 |
| 2 | **mxalign v0.2.0** | Separation release: loading and validation outsourced to loaders and specs ([mxalign#27], already on `main`), leftovers removed in [mxalign#47]. Git pins replaced with released versions ([step 2](#step-2)). | loaders v0.1.0, specs v0.1.0 |
| 3 | **mlwp-data-specs v0.2.0** | Ensemble support: `Uncertainty` member and quantile conventions ([specs#34], [specs#35]). Also the checks restructure ([specs#36], merged) and optional `mlwp_*_trait` global attributes ([specs#38]). | none |
| 4 | **mlwp-data-loaders v0.2.0** | Loaders return and validate ensemble datasets. | specs v0.2.0 |
| 5 | **mxalign v0.3.0** | Alignment refactor ([mxalign#30]), ensemble alignment ([mxalign#45]) and the MkDocs site ([mxalign#43]). | loaders v0.2.0, specs v0.2.0 |
| 6 | **mlwp-data-loaders v0.3.0** (plus specs and mxalign as needed) | Grid-mapping and projection information ([loaders#11], [mxalign#31]). Likely a specs change first, then loaders, then an mxalign bump. | decided then |

Why split mxalign into two releases: the separation is a breaking, user-visible change (import paths change, the loaders are gone) that is already merged and independent of ensembles. Releasing it alone keeps the migration note small and makes the later ensemble release purely additive.

## Status

### Step 1: mlwp-data-loaders v0.1.0 {#step-1}

Done, merged on `main`:

- [loaders#24] anemoi-datasets, [loaders#25] anemoi-inference, [loaders#27] IFS forecast and [loaders#30] HARP obstable: lazy loaders whose dask graphs don't grow with the number of files or the length of the record.
- `loader` can be a python callable ([loaders#28]).

Still to do before tagging:

- **ADR-001, one conventions helper per trait axis** ([loaders#29], ADR only, open). It decides that every bundled loader ends with `apply_time_conventions`, `apply_space_conventions` and `apply_uncertainty_conventions` from a new `mlwp_data_loaders.conventions` package, so all loaders give the same coordinates. These are float64 latitude and longitude, longitudes in `[-180, 180)`, `datetime64[ns]` times, sorted unique time labels, `(member, time, other, space)` dimension order and the member conventions of [specs#35]. Merge [loaders#29] first, or together with the implementation.
- **The implementation PR** is written but not opened yet. It waited for [loaders#24], [loaders#25] and [loaders#27]; now that those and [loaders#30] are merged, it gets rebuilt on `main` and opened.
- The implementation changes loader output, which is why it belongs in the first release rather than in a patch right after: anemoi-datasets longitudes 0–360 → `[-180, 180)` and float32 → float64, HARP `altitude` → `elevation`, `reference_time` sorted, and duplicate forecasts raise. These need migration lines in the CHANGELOG. On mxalign `main` only the notebook outputs are affected ([step 2](#step-2)).
- **Write trait attributes as plain strings** (added 2026-10-29). Every loader on `main` sets the `mlwp_*_trait` attributes to Enum members (`ds.attrs[TIME_TRAIT_ATTR] = Time.FORECAST`), and so do the conventions helpers. Enum members in attrs break saving to disk: `to_netcdf(engine="netcdf4")` writes `'Time.FORECAST'`, which fails validation when read back, and `engine="h5netcdf"` raises a `TypeError`. They also fail [specs#38] as it stands ([step 3](#step-3)). Fix: write `.value` in the conventions helpers. mxalign already writes `.value`.
- CHANGELOG: `main` already has a `## [0.1.0]` section from [loaders#19], but no tag exists. At release, merge `[Unreleased]` into it (or rename it) so v0.1.0 has one set of notes.

Not blocking: [loaders#26] (anemoi-inference reopens remote files for every read) and [loaders#23] (zarr pin, [item 4](#item-4)).

What the rewrites leave for mxalign: step 2 needs mxalign's loaders pin and lock bumped (below), and step 5 needs [mxalign#30] to pass `variables` through to the loaders again, as mxalign `main` already does.

### Step 2: mxalign v0.2.0 {#step-2}

[mxalign#27] moved loading and validation out, and [mxalign#47] (merged 2026-10-28) removed the leftovers. `src/mxalign/loaders/` is gone, and `Runner.load_datasets` calls `mlwp_data_loaders.load_and_validate_dataset` and passes `variables:` through. The CHANGELOG's `[Unreleased]` section has the migration note. `properties/` is no longer tracked. `src/mxalign/utils/traits.py` stays: it isn't a leftover. `accessors/time.py` and `interpolations/base.py` use it to re-validate and set a trait that alignment changes.

Left for step 2, all waiting on loaders v0.1.0:

- `[tool.uv.sources]` still pins specs to rev `059f382` and loaders to the default branch. The `uv.lock` resolves the loaders to `3fc53f3` ([loaders#6]), so mxalign's CI runs none of the loader rewrites yet. Replace both pins with released versions and re-lock with uv 0.13.5. [specs#36] doesn't affect mxalign: it only imports `mlwp_data_specs.api` and `mlwp_data_specs.specs.traits.*`, not the moved `checks` modules.
- The version is still static (`version = "0.1.0"`). Switch to setuptools-scm before tagging ([versioning rules](#versioning-rules)).
- Re-run `examples/introduction.ipynb`. Its saved outputs still show the old HARP `altitude` coordinate (ADR-001 renames it `elevation`). No mxalign code on `main` uses `altitude` or 0–360 longitudes.

### Step 3: mlwp-data-specs v0.2.0 {#step-3}

Merge order is [specs#36] → [specs#38] → [specs#35], all in v0.2.0 (decided 2026-10-29).

- **[specs#36]**: merged 2026-10-29 (squash, `29746f1`). It moves `checks/traits/_common.py` and `checks/metadata/coords.py` into `checks/coords/` and renames `axis=` to `trait_axis=`.
- **[specs#38]**: in review ([review of 2026-10-29](https://github.com/mlwp-tools/mlwp-data-specs/pull/38#pullrequestreview-5695380288)). It needs a fix before merging: attributes set to Enum members fail validation (table below). Also, an invalid attribute is only reported when that trait is passed explicitly, and the branch still carries the pre-squash [specs#36] commit, so it needs a rebase onto `main`.
- **[specs#35]**: conflicts with `main` since [specs#36]. It modifies the two files [specs#36] deleted, so its check changes move to `checks/coords/{uncertainty,dimensions,names}.py` with `trait_axis=`. Rebase it after [specs#38], which also edits `specs/traits/uncertainty.py` and `CHANGELOG.md`, and keep its `VERSION = "0.2.0-draft"` over the `0.1.1-draft` of [specs#38].

Compatibility with loaders v0.1.0 ([item 2](#item-2)): the loaders `main` test suite, run against each specs change on 2026-10-29.

| specs | loaders `main` |
|---|---|
| `main`, with [specs#36] | 49 passed |
| [specs#35] | 49 passed |
| [specs#38] as it stands | 12 failed, 37 passed |
| [specs#38] with `StrEnum` trait enums | 49 passed |

[specs#38] fails because its new check compares `str(value)`, and `str()` of a `(str, Enum)` member is `'Time.FORECAST'`, not `'forecast'`. Either fix on its own keeps the `<0.3` bound safe: `StrEnum` in specs (suggested in the [specs#38] review), or loaders writing `.value` ([step 1](#step-1)). Both are planned; the `StrEnum` change also covers third-party loaders that store Enum members.

## Versioning rules

- SemVer, tag-driven: tag `vX.Y.Z` on `main` of each repo, and the release workflow publishes to PyPI. Before 1.0, a minor bump (0.1 → 0.2) may contain breaking changes; patch bumps are fixes only.
- specs and loaders use setuptools-scm, so the tag *is* the version. mxalign still has a static version; switch it to setuptools-scm too, as specs and loaders did, so no manual bump is needed.
- Each release: move `[Unreleased]` in `CHANGELOG.md` to the new version, then tag.
- Dependency constraints in `pyproject.toml`: loaders requires `mlwp-data-specs>=0.1,<0.3` while releasing step 1 (so it works with both), then `>=0.2,<0.3` at step 4. mxalign v0.2.0 requires `>=0.1,<0.2` of both; v0.3.0 requires `mlwp-data-loaders>=0.2,<0.3` and `mlwp-data-specs>=0.2,<0.3`.
- Before each tag, replace git pins in `[tool.uv.sources]` (mxalign pins specs to rev `059f382` and loaders to the default branch) with released versions and re-lock with uv 0.13.5.

## Things to decide or watch

1. []{#item-1}**Loaders v0.1.0 vs ensemble**: step 1 must not include any ensemble changes, so the dask-graph work branches from `main` and ensemble work waits for step 4. *Update 2026-10-28:* [loaders#27], now on `main`, already returns IFS ensembles with `member` first and the [specs#35] member conventions (`apply_member_conventions`). ADR-001 keeps them, renamed to `apply_uncertainty_conventions`. Decide whether v0.1.0 ships this member handling, with step 4 narrowing to validating ensembles against specs v0.2.0, or whether it is held back.
2. []{#item-2}**specs v0.2.0 and loaders v0.1.0**: loaders v0.1.0 depends on specs v0.1.0. Check that the v0.2.0 changes to specs are backwards compatible for single-member data, otherwise loaders v0.1.x needs an upper bound `<0.2`. *Update 2026-10-29:* compatible once [specs#38] is fixed; see the table in [step 3](#step-3).
3. **Breaking change in mxalign**: v0.2.0 removes loading and validation from mxalign (import paths change). Document the migration in the changelog. *Done in [mxalign#47].*
4. []{#item-4}**zarr/xarray pin**: `zarr<3` blocks xarray ≥ 2026.9 ([mxalign#46], [loaders#23]). Fix before step 2 if possible, or otherwise release with the `xarray<2026.9` cap and note it.
5. **mxalign PR stack**: *Update 2026-10-29:* [mxalign#44] is merged. [mxalign#45] (ensemble alignment) and [mxalign#43] (docs) are both based on [mxalign#30] (alignment refactor), so merge [mxalign#30] first, then the other two, then tag v0.3.0.
6. **Old `feat/ifs-fcst-loader` branch in mxalign**: superseded by the IFS forecast loader in [loaders#12], and now deleted on GitHub. Check that its fix for removing the `member` coordinate from deterministic forecasts is covered by the ensemble work in step 4.
7. **Open specs issues to settle before specs v0.2.0**: [specs#37] (control member), [specs#1] (CF overlap) and [specs#39]. [specs#39]: the `lead_time` units check conflicts with xarray's timedelta encoding. It also proposes the ADR-001 conventions as SHOULDs in the specs. Once settled, `LEAD_TIME_UNITS` in the loaders' `conventions/temporal.py` moves to `encoding`. The open specs PRs are in [step 3](#step-3).
8. **Loaders issues to triage for v0.1.0**: [loaders#17] (large dask graph warning, closed with the rewrites), [loaders#28] (loader as python callable, closes [loaders#14]; merged), [loaders#29] (ADR-001, see [step 1](#step-1)), [loaders#13] (Intake driver), [loaders#23] (zarr pin) and [loaders#26] (remote handle cache, a follow-up).
9. **Stale mxalign work**: `ens-dev` ([mxalign#29], list of changes), [mxalign#26] (use-case optimization, draft) and [mxalign#1] (separate concerns) overlap with the refactor. Decide what is superseded and close or fold it in before step 5, so the changelog is honest.
10. **Dependabot PRs**: merge or close them before tagging so the release CI actions are current. *Done 2026-10-29:* none open in specs, loaders or mxalign.
11. **Docs and migration**: the MkDocs site ([mxalign#43]) should document v0.3.0 (step 5). Add a short "migrating from mxalign 0.1" page, since loading and validation move out.
12. **Release mechanics**: the first PyPI release of loaders needs the trusted-publisher setup verified (the workflow came in [loaders#21]). Consider TestPyPI dry runs, and tag mxalign with the same process (its release workflow already triggers on a GitHub release).

[specs#1]: https://github.com/mlwp-tools/mlwp-data-specs/issues/1
[specs#34]: https://github.com/mlwp-tools/mlwp-data-specs/issues/34
[specs#35]: https://github.com/mlwp-tools/mlwp-data-specs/pull/35
[specs#36]: https://github.com/mlwp-tools/mlwp-data-specs/pull/36
[specs#37]: https://github.com/mlwp-tools/mlwp-data-specs/issues/37
[specs#38]: https://github.com/mlwp-tools/mlwp-data-specs/pull/38
[specs#39]: https://github.com/mlwp-tools/mlwp-data-specs/issues/39
[loaders#6]: https://github.com/mlwp-tools/mlwp-data-loaders/pull/6
[loaders#11]: https://github.com/mlwp-tools/mlwp-data-loaders/issues/11
[loaders#12]: https://github.com/mlwp-tools/mlwp-data-loaders/pull/12
[loaders#13]: https://github.com/mlwp-tools/mlwp-data-loaders/pull/13
[loaders#14]: https://github.com/mlwp-tools/mlwp-data-loaders/issues/14
[loaders#17]: https://github.com/mlwp-tools/mlwp-data-loaders/issues/17
[loaders#19]: https://github.com/mlwp-tools/mlwp-data-loaders/pull/19
[loaders#21]: https://github.com/mlwp-tools/mlwp-data-loaders/pull/21
[loaders#23]: https://github.com/mlwp-tools/mlwp-data-loaders/issues/23
[loaders#24]: https://github.com/mlwp-tools/mlwp-data-loaders/pull/24
[loaders#25]: https://github.com/mlwp-tools/mlwp-data-loaders/pull/25
[loaders#26]: https://github.com/mlwp-tools/mlwp-data-loaders/issues/26
[loaders#27]: https://github.com/mlwp-tools/mlwp-data-loaders/pull/27
[loaders#28]: https://github.com/mlwp-tools/mlwp-data-loaders/pull/28
[loaders#29]: https://github.com/mlwp-tools/mlwp-data-loaders/pull/29
[loaders#30]: https://github.com/mlwp-tools/mlwp-data-loaders/pull/30
[mxalign#1]: https://github.com/mlwp-tools/mxalign/issues/1
[mxalign#26]: https://github.com/mlwp-tools/mxalign/pull/26
[mxalign#27]: https://github.com/mlwp-tools/mxalign/pull/27
[mxalign#29]: https://github.com/mlwp-tools/mxalign/issues/29
[mxalign#30]: https://github.com/mlwp-tools/mxalign/pull/30
[mxalign#31]: https://github.com/mlwp-tools/mxalign/issues/31
[mxalign#43]: https://github.com/mlwp-tools/mxalign/pull/43
[mxalign#44]: https://github.com/mlwp-tools/mxalign/pull/44
[mxalign#45]: https://github.com/mlwp-tools/mxalign/pull/45
[mxalign#46]: https://github.com/mlwp-tools/mxalign/issues/46
[mxalign#47]: https://github.com/mlwp-tools/mxalign/pull/47
