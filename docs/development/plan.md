# NistChemPy and NistChemData — development specification

**Date:** 26 November 2026.  
**Revision:** 27 November 2026 — branch model and final documentation stage.  
**Purpose:** a basis for one coordinated development cycle in the permanent integration branches (`dev`) of the two existing repositories, without intermediate public releases.  
**Status:** approved working plan; bootstrap is in progress, and functional implementation has not yet started. Detailed decisions within each stage remain subject to its acceptance.

## 1. Decision and context

Develop the existing NistChemPy: consolidate knowledge of HTML markup in `parsing/`, extend data extraction, and ensure that provenance is preserved in a verifiable manner. Retain NistChemData as a collection of local retrieval and processing scripts built on the library.

The first development outcome must be **a catalog of the entire target functionality with synthetic input examples and expected results**, including sections not yet implemented. This corpus serves simultaneously as a specification, coverage map, and basis for subsequent tasks.

Four objectives from the discussion are retained:

1. Maintainable access to WebBook that is resilient to changes in page formatting.
2. Broad coverage of scientific data, with mandatory source and rights information.
3. Reproducible construction of local datasets through NistChemData.
4. Preparation of evidence for a paper on software and reproducible scientific data retrieval; the intended publication format is a JCIM Application Note.

The main architectural constraint is that **each change must eliminate observed duplication, prevent a specific loss of information, or provide a necessary check**. Design for extensibility where the catalog already demonstrates distinct cases. A universal engine for unknown future sources is not required.

## 2. Verified baseline

The repeated audit was conducted against the following states of `main`:

| Repository | Commit | Existing capabilities |
|---|---|---|
| NistChemPy | `569c4ed951673ff09183df1fb19a10018e108e14` | Version 2.0.0; search, compound pages, 2D/3D MOL, retrieval of four types of JCAMP spectra, GC, dataclass records, JSON/JSONL, local index and CLI |
| NistChemData | `50f069a835bc9b3a9415b9946d61e4ec5ee22203` | Three download scripts and four processing scripts; ZIP archives, CSV manifests, resumption, legacy archive support, NistChemPy local index |

NistChemPy already has `parsing/` and `records/`. Transport, public objects, and indexing are also organized separately. This separation should be made into a working contract rather than rebuilt from scratch. The library's mapping contains 14 data families, but nine are represented by link discovery without specialized table extraction. [P1][p1] [P2][p2] [P3][p3]

Offline and network tests are already separated: the ordinary test run excludes `network`, and network CI is triggered manually. The dedicated HTML corpus contains eight files; additional small examples are embedded in the indexing tests. This is a useful foundation, but not yet a representative specification of WebBook. [P4][p4] [P5][p5]

### Findings added by the repeated audit

| Observation in the code or source | Consequence for the specification |
|---|---|
| HTML correction and BeautifulSoup handling reside in `requests.py`; result recognition partly in `search.py`; spectral link extraction in `compound.py` and NistChemData | Move specific markup operations into library parsers; existing public entry points delegate to them |
| An unrecognized HTTP 200 response can become a successful empty search; MOL/JDX responses are accepted without format validation | Negative fixtures and a distinction between absent data, failed requests, and unrecognized responses |
| The GC parser captures subsequent tables without respecting section boundaries; repeated retrieval appends data | Test section boundaries, multiple tables, and repeated retrieval |
| `retrieved_at` is defined, but the normal record converters do not populate it | Record retrieval time once and preserve it during export |
| The NistChemData GC archive labeled raw contains already parsed CSV files | Store source HTML separately from derived tables for reparsing |
| An archive without a manifest row can be considered complete when only some files are present | Fix resumption after interruption of the first multi-resource download |
| The index writes `status='complete'` after enrichment errors; the combined build replaces seeds and clears accumulated partial results | Track process completion, completeness of the selected scope, and correct resumption separately |
| The ethanol page has reaction continuation pages and separate measurement pages behind average values; the current `get_compound_data_refs()` takes the first link of a subsection | Include the necessary navigation paths in the catalog and retrieval operations, not just top-level sections |

Evidence: [P2][p2] [P3][p3] [P6][p6] [P7][p7] [P8][p8] [D1][d1] [D2][d2] [D3][d3] [W2][w2] [W3][w3].

These are properties established by static inspection of the implementation and observations of several pages, not claims about the frequency of failures across WebBook. The full pytest suite was not executed during this audit because `requests` was missing from the working Python environment. Establish the baseline test result at the beginning of development.

## 3. Development outcome and coverage boundaries

**Development outcome:** a library and coordinated scripts that extract the entire mandatory scope from the established WebBook coverage matrix, preserve scientific context and provenance, can be tested without network access, and allow saved local sources to be reprocessed.

**Practical project outcome:** a local retrieval covering as much of the discovered scope as possible, a report on coverage and gaps, and reproducible examples for research and the paper. Track completion of development, completion of a particular retrieval, and manuscript readiness separately.

### Mandatory development scope

| Group | What must be represented in the catalog and supported by the end of the development cycle |
|---|---|
| Identification and navigation | Compound pages, existing search types, empty/truncated results, formula browser and sitemap; stereoisomers and isotopologues as related records |
| Structures | 2D/3D MOL, links, and information about the provenance or computed nature of a structure where provided by the source |
| Spectra | IR, THz, MS, UV/Vis: listings, individual spectra, original files, metadata, links to additional images |
| GC | All identified large-format tables, measurement conditions, classification, comments and references |
| Thermochemistry | Gas and condensed phases: individual measurements, series, recommended/average values and their distinctions, parameters of published correlations |
| Phase transitions and Henry's law | Quantities, temperatures/ranges, units, conditions and correlation coefficients |
| Reactions and ions | Reaction thermochemistry, ion energetics and clustering: participants, stoichiometry/charge and phase designations in an available structured form, associated measurements |
| Energy levels and diatomic molecules | Tables, state and transition designations, constants, units, notes and literature |
| Information common to all groups | Sources, compilers, bibliography, footnotes, owner/origin/collection, rights notices and unknown values |

The 14 codes in the current library are an initial cross-check of the list of families, not proof of complete coverage of the site. In A0, each family is divided into **genuinely distinct variants**. Account for multiple sections on a single page, continuations, links to individual values, and additional final resources. [P1][p1] [W1][w1] [W2][w2] [W3][w3]

For spectra, preservation of the original JCAMP and the complete available metadata is mandatory, as is correct operation of the existing IR/MS processors. Numerical representations of other variants are added where needed for the agreed output. Unsupported encodings are preserved and flagged; a few examples do not justify claiming universal JCAMP support.

### Explicit exclusions from this development cycle

Limit mandatory implementation to compound-page data and the navigation needed to retrieve it. Inventory but defer the separate Fluid Systems and Group Additivity computational services, specialized experimental collections, and independent new search interfaces not needed for this retrieval. This is **a scope clarification of the ambiguous goal of covering the entire API**. Fluid Systems, for example, requires selecting a mode, a grid of conditions, units, and a standard state; it is a separate task. [W4][w4]

Also out of scope: access to subscription databases via external links, OCR/digitization of images, reproducing NIST's computational models, automated scientific reassessment of measurements, an ML benchmark, a web service, and public redistribution of the full retrieved dataset. Preserve links to and available descriptions of these resources.

A mandatory catalog entry cannot be closed as implemented merely because its source HTML is saved. The agreed fields require verified structured extraction. Any subsequently encountered unknown variant must remain visible as a coverage gap; it cannot silently be excluded from the denominator.

## 4. A0: specify the entire scope first

A0 covers **all mandatory groups**, not only the current capabilities of NistChemPy.

For each case, record:

- a stable `case_id`, family, and significant variant;
- the URL of the observed source, date of inspection, and structural features it demonstrates;
- synthetic inputs: HTML, JCAMP/MOL, and a linked set of responses for multistep retrieval;
- expected values, units, source associations, rights, and expected failure behavior;
- current implementation status, test, and development task.

One small machine-readable catalog, such as `tests/fixtures/cases.json`, the fixture files themselves, and expected JSON are sufficient. No separate database, website generator, or test description language is needed. The documentation's overview table should be based on this catalog.

A0 also includes end-to-end cases: a synthetic index, current and legacy manifests, a small ZIP archive, interruption after the first of two resources, resumption, export, and reprocessing. Prepare their inputs and expected results at the start; the code and executable scenario tests may be introduced in A1–A3. Do not introduce a separate scenario language for this purpose; ordinary pytest fixtures are sufficient.

**Sequence:** inspect a real example → independently write a minimal synthetic equivalent → manually verify the expected result → add it to the matrix. Do not generate the expected result mechanically from the current parser.

Dummy pages reproduce significant structures and associations but contain original names, numbers, references, and text. Do not copy entire pages while replacing only a few values. Do not include real measurement datasets, lengthy source notes, images, or branding. Explicitly state the authorship and distribution terms of the synthetic corpus.

The minimum for each distinct variant is a valid example and its expected result. Each family must also include applicable cases of missing optional information and damaged mandatory structure. Cover common variations—whitespace, line breaks, nested tags, attribute order—through parameterization without hundreds of copies. Include multiple sources/tables, numerical qualifiers, `rowspan/colspan`, superscripts/subscripts, both unit systems, and navigation between pages where observed.

**A0 deliverable:** the entire agreed catalog and a representative corpus, not an exhaustive model of every page on the site. Validate the fixture files and expected structures immediately. Activate parser tests for future capabilities when their implementation is introduced; keep `planned` cases visible in the report without widespread use of `xfail`. No mandatory `planned` cases may remain at the end of development.

Design the minimum provenance fields from Section 6 at this stage. The detailed numerical field layout of each family is established by its expected results; a separate Python class for each quantity is not required.

## 5. Architecture with minimal changes to existing code

| Existing component | Responsibility after the work |
|---|---|
| `requests.py` | HTTP, configuration, bounded retries, original response and retrieval time |
| `parsing/` | Markup recognition, section boundaries, links, tables, bibliography and conversion to agreed structures; no HTTP or file writing |
| `compound.py`, `search.py` | Public operations, selection of subsequent requests, result assembly and object updates |
| `records/` | Existing record types, a small common provenance component and necessary new table records; JSON/JSONL |
| `indexing/`, CLI | Candidate discovery, enrichment and local index; accurate accounting of selected scope, errors and resumption |
| NistChemData `scripts/` | Scope selection, sequential retrieval, archives, resumption, derived tables and run report |

A parser receives text/DOM and the page URL; it must not require a client, a live `requests.Response`, global state, or the current time. Existing functions accepting BeautifulSoup may be retained. Existing `NistResponse.text`, `.soup`, and public imports remain compatible thin entry points; actual markup processing is delegated to `parsing/`.

Existing `NistCompound.data_refs`, `CompoundRecord.data_refs`, and index columns retain one primary URL per family. A complete list of links is added as a separate view and used by the new traversal, including continuations and individual measurements. The existing field must not silently change from a string to a list. Internally, one complete collection and a derived compatibility view are sufficient; two independently mutable link collections are unnecessary.

Add new files as they become useful: common HTML helpers, parsing of search results, spectral links and the formula browser, then scientific table families. There is no need to lay out all potential modules in advance.

Shared functions are useful for five recurring operations:

1. Extracting a section up to the next heading of the appropriate level.
2. Reading text while preserving scientific notation and meaningful separators.
3. Resolving relative links and URL parameters through the standard `urllib.parse` module.
4. Reading observed table layouts while preserving headers, merged cells, references and notes.
5. Extracting bibliography and provenance statements together with their scope.

Extract common code once repetition across distinct cases has been confirmed. Interpretation of reaction enthalpies, Antoine coefficients, spectral states, and reactions remains specialized. Identical HTML does not imply identical scientific meaning.

For new tabular data, start with a simple table container: family, table type, data, conditions/metadata, sources, and diagnostics. Agreed field names for each family are defined by its fixtures. Retain pandas and dataclasses. Introduce a separate type when the existing container would lose meaning or require incompatible fields.

**Do not introduce without a separately demonstrated need:** plugins/provider registries, a general site AST, ORM, a database, a universal unit system, task queues, asynchronous traversal, browser automation, automatic guessing of changed structures, or a legal decision engine. Do not impose an arbitrary target number of files or classes.

## 6. Data, provenance and rights

Preserve three distinguishable levels: original resources; extracted information without loss of content; derived normalized data. Normalization must not replace the source.

| Information group | Minimum content |
|---|---|
| Identity | WebBook ID, family, spectrum/table index or local identifier associated with the original resource |
| Retrieval | Requested and final URL, UTC retrieval time, hash of the saved response body, encoding where necessary |
| Scientific source | Bibliographic records and links from rows/series to them; collection, origin, owner, compiler where specified |
| Rights | Observed notice/link to terms, its scope and source; whether rights information is known; explicit absence or conflict of information |
| Processing | Schema and NistChemPy versions; commit for a development build; NistChemData version/commit and transformation parameters for derived exports |

Retain `source_url` and `retrieved_at`. Missing information in legacy archives and manually created records remains unknown. Re-export time does not become retrieval time. Do not invent NIST provenance for an arbitrary user-provided dictionary passed through a writer.

Preserve numerical fields together with units, temperature/pressure and other context, uncertainty, range or qualifier, method, phase, and notes. A value that cannot be parsed confidently remains available in its original form with a diagnostic. Missing uncertainty does not mean zero; an average does not replace individual measurements; records from distinct sources are not deduplicated solely because their numerical values agree. Do not implement evaluation of every published correlation merely to extract its coefficients.

Owner, origin, article reference, and license are distinct pieces of information. A real spectrum page provides separate owner, origin, and collection rights information; NIST separately describes the treatment of SRD and other materials. The library therefore preserves evidence and explicit unknown values rather than inferring unrestricted use from public page access or the MIT license of the code. [W5][w5] [W6][w6]

A page-wide notice cannot unconditionally be treated as the sole license of every row. For mixed sources, preserve the association with the table/row/spectrum; multiple applicable notices must not overwrite one another. Distinguish extracted information from an explicitly added rights assessment.

JSON/JSONL must carry these fields. CSV/SDF and combined tables receive a stable record identifier and a link back to the source. If the export format itself cannot preserve the complete association, an accompanying JSON/JSONL file is mandatory. One global provenance statement for the entire CSV is insufficient. Do not use the current row number as the only key. Acceptance example: three rows of one table, two with different references/rights notices and one with unknown rights; after filtering, reordering, and re-export, the associations remain correct. Shared notices may be stored once and referenced by identifier.

## 7. Errors and tests

### Distinguishing outcomes

Confirmed absence of data, unsupported variants, damaged/unexpected responses, network failures, and partial results have distinguishable outcomes. A common result wrapper for all existing methods is not required: consistent diagnostics, the existing exception hierarchy, and statuses in the batch report are sufficient.

An object operation must not leave partially appended state without an explicit indication. Repeating a successful retrieval must not create duplicates. Batch retrieval may continue after an individual item fails, but must finish with a report and a nonzero exit code if unresolved errors remain.

### Three levels of testing

| Level | What it checks | Role |
|---|---|---|
| Offline | Fixtures → expected values; public operations with substituted responses; export and resumption | Mandatory CI and acceptance criterion |
| Live source compatibility checks | A small set of real URLs → expected families, meaningful features and availability of required navigation paths | Separate manual and periodic workflow with warnings; does not block ordinary PRs |
| Network scenarios | Real search → retrieval → parsing → record | Separate optional technical integration run; limited acceptance run before final delivery |

Live source compatibility checks compare significant features, not the entire DOM, exact row counts, or unchanged chemical values. They do not establish the equivalence of the dummies to the entire site. Distinguish source unavailability, structural drift, and expected content changes. Fixtures must not be rewritten automatically.

A CI warning does not authorize exporting known incorrect data: confirmed parsing errors in mandatory cases must be fixed before acceptance. Site unavailability is recorded separately and does not cause offline tests to fail.

Mandatory regressions: unrecognized HTML with HTTP 200; HTML instead of MOL/JDX; incorrect/unrelated `Index`; preservation of RequestConfig; GC section boundaries; repeated calls; multiple URLs for one family; reaction continuations; individual values; multiple bibliographic references; rights from different sources; interruption and resumption; a missing file in a completed archive; identical captions of different GC tables; a partial index.

Use pytest and the existing markers. Technically prevent network access in ordinary tests. Check common formatting changes using a bounded set of parameterized transformations. Broad fuzzing, mutation-testing infrastructure, and arbitrary code coverage percentages are not part of this development cycle.

## 8. NistChemData and local reproducibility

Retain the scripts, ZIP archives, and familiar derived formats. Eliminate duplicate link extraction and resource retrieval logic: these belong in NistChemPy. Library operations must be able to use the URL/ID from an existing index without making an unnecessary compound-page request for each file.

Add a reproducible retrieval path through the library for every new mandatory family. If one common tabular data script is sufficient, do not create a downloader for every physical quantity.

Necessary changes:

- Save the original bodies of all responses used, including GC HTML, compound pages, listings, continuations, and individual-measurement pages, where the result, set of child resources, or provenance depends on them. Record metadata for each response; label derived CSV files as processed.
- Identify resources by their source, not solely by table caption or compound name.
- Account for the expected set of resources before declaring an item complete; retain the state of interrupted retrieval.
- For the new format, verify file existence, hashes, and minimum validity. Read old archives without adequate manifests as legacy archives with unknown completeness; reuse verified files but do not declare the whole set complete.
- Preserve the original metadata of reused resources. A changed source on a later visit creates a new version of the local input and is explicitly reflected in the report.
- Version the manifest format; do not append new columns beneath an old CSV header.
- Record selected IDs/families, limits, the index and its hash, versions of both repositories, parameters, and output checksums.
- Preserve the complete original set of spectral metadata. The existing IR origin/owner/source reference fields are useful; a fixed column projection must not discard other information. [D4][d4]
- Keep scientific filtering, sample-state normalization, selection of the first spectrum, and similar decisions as explicitly named transformations, preserving inputs and parameters.

**Reproducibility boundary:** the saved index is an explicit input to NistChemData. Starting from that input, preserve all used sources and associations necessary to repeat retrieval within the selected run and its subsequent processing. Rebuilding the index from the entire live site is not a requirement for offline dataset reproduction; record the index's discovery strategy, limitations, and errors separately.

**Reproducibility criterion:** identical saved inputs, settings and versions produce equivalent scientific records and provenance associations without network access. When validating resumption after interruption following the first of two resources, account for the expected resource set, original retrieval times, and rights. A new retrieval from a changing site need not produce the same dataset; ZIP timestamps alone do not constitute a scientific discrepancy.

"All data" means the entire planned and discovered scope of a particular run. The report identifies selected records, discovered resources, successfully retrieved and parsed resources, confirmed absence, unsupported variants, errors, and discovery limitations. One completed traversal does not establish global WebBook completeness. Full personal retrieval follows a small pilot and takes account of the source's terms; its success is evaluated using this report.

## 9. Work plan and verifiable deliverables

The stages are internal checkpoints in a single development cycle. They do not imply intermediate releases.

| Stage | Work | Completion condition |
|---|---|---|
| A0 — catalog and reference cases | Record the baseline pytest result; inventory the entire mandatory scope; create dummies, expected results and minimum provenance schema | Every mandatory variant has an example, expected output and status; new observations become concrete tasks; scope limits are explicit |
| A1 — isolation and existing contracts | Move HTML operations into `parsing/`; implement common provenance fields; fix false successes, configuration, link selection and repeated GC retrieval | Existing public scenarios pass offline; parsers do not depend on network access; provenance survives through export |
| A2 — scientific coverage | Implement all mandatory families and navigation paths in small tasks according to their reference cases | Each variant has a parser, test, preserved scientific context and usage example; no mandatory gaps remain |
| A3 — index and batch retrieval | Fix index completeness accounting/resumption; coordinate NistChemData; original files, manifests, processing and all new families | The synthetic sequence index → retrieval → interruption → resumption → export works without duplicates or losses |
| A4 — functional acceptance | Check live source compatibility, a small real pilot, independent comparison, compatibility of the two repositories, and the build | Functional criteria in Section 10 are met; final versions of both repositories are compatible; scope and contracts are ready for final documentation |
| A5 — documentation and final acceptance | After A4, migrate NistChemPy documentation to Zensical, update the README, navigation, guides, API and examples; create an independent landing page with dark/light modes, using pyvoro2 as a reference; organize working materials | Documentation reflects the implemented functionality, has been built and checked; working materials are integrated or archived; joint criteria in Section 10 are met; release PR is ready |
| After acceptance | Full personal retrieval, coverage report, research examples and paper material | The resulting dataset has verifiable composition and provenance; limitations or errors are explicitly reported |

A reasonable order within A2 is: strengthen compound pages/spectra/GC; gas and condensed-phase thermochemistry; phase transitions and Henry's law; reactions, ion energetics and clustering; energy levels and diatomic molecules. Refine the order after A0 based on observed structural commonality. Complex families are included in the reference cases from the beginning so that early decisions take them into account.

Working areas: `tests/fixtures/`, `tests/test_parsing_*`, `tests/online/`; `requests.py`, `compound.py`, `search.py`, `parsing/`, `records/`; focused changes to `indexing/builder.py`, `discovery.py`, `enrichment.py`; then NistChemData `scripts/` and a new offline test suite. Update technical descriptions of implemented behavior, limitations, and tests alongside the corresponding capability; do not leave known errors in affected examples or API descriptions. Keep the existing site and Sphinx until A5. Restructure the documentation, migrate to Zensical, and design the landing page only after functional acceptance, not during A0–A4.

Each implementation task follows a short cycle: independent expected result → test of specific behavior → minimal implementation → validation of affected scenarios → small commit. Define detailed signatures of new scientific parsers based on A0; specifying them in advance for variants that have not yet been investigated would be premature.

## 10. Acceptance criteria

- All mandatory catalog cases are implemented, have active offline tests, and are tested through the user-facing path; saving raw text does not substitute for extracting the agreed fields.
- A deterministic offline test run has completed with network access prohibited. Test the declared minimum Python version for NistChemPy (`>=3.9`) and a selected current supported version; a change to the minimum must not be disguised as refactoring.
- Existing public imports, calling conventions, and the meaning of existing results are preserved. New fields must not silently shift positional arguments of dataclass constructors. Documented changes to erroneous behavior are tested separately.
- JSON/JSONL and tabular/SDF exports preserve the required link to the source and known/unknown rights information. Disabling raw text in exports does not remove provenance.
- Repeated retrieval and resumption do not lose resources, create unexplained duplicates, or relabel a partial result as complete.
- The entire NistChemData path can be reproduced without network access on the saved synthetic corpus; reprocessing does not change original retrieval times.
- A small live pilot is checked independently of parser code: values, units, conditions, references, and completeness of navigation paths. Each mandatory family requires at least one verified real example. A dated manual check from A0 counts if subsequent contract changes have not invalidated it and no confirmed discrepancy exists. Current source unavailability is recorded separately; if a real example has never been checked, the case remains an incomplete external validation.
- The report contains the coverage denominator and all unresolved cases. Confirmed failures of mandatory extraction are not dismissed as warnings.
- README/API documentation/examples describe new capabilities and limitations; the build is checked to ensure it contains no local retrievals or original real pages. Retain the existing `tools/check_package_artifacts.py`. [P9][p9]

Final A5 acceptance additionally requires:

- The NistChemPy site builds with Zensical without retrieving WebBook data over the network and without mandatory notebook execution; all documented imports and examples are checked by applicable tests.
- The separate GitHub/PyPI README and site landing page make consistent scientific claims. The landing page has its own design, light and dark themes, and clear entry points into the documentation; it need not literally copy pyvoro2's visual system.
- Navigation and user/API documentation are cleared of outdated and duplicate material. Existing public paths are preserved or intentional redirects are provided; working plans and necessary decisions are integrated or archived.
- Documentation preview builds are available for PR review; production deployment runs only from `main`. Until A5, the current Sphinx site remains operational.

Code completion does not require journal acceptance of the paper, a successful NIST response to a letter that has not been sent, or an error-free network traversal of arbitrary size. Readiness of a particular retrieval is assessed separately from readiness of the tool.

## 11. Organization of the development cycle

Permanent branch model following pyvoro2:

| Branch | Role |
|---|---|
| `main` | Stable working version, default branch, source of public documentation |
| `dev` | Protected permanent integration branch, initially based on the current `main` |
| Working branches | Small tasks based on the current `dev`, e.g. `work/a0-fixtures`; changes are accepted into `dev` only through PRs |

Both integration branches are protected against direct writes, deletion, and force pushes, and require PRs and a mandatory deterministic CI check. The basic rulesets follow pyvoro2, including immutability of existing `v*` tags. The required `offline` check, with up-to-date branches required, is enabled after a fresh successful CI result; network checks are not required status checks. Squash merging is preferred for ordinary working PRs. Before the first release PR, agree on how to synchronize the history of the permanent `main` and `dev` branches after release, accounting for linear history and the force-push prohibition. Do not silently reset protected `dev` or publish intermediate versions.

Apply the same model when work on NistChemData begins; do not merge the repositories for this purpose. Record the pair of Py/Data commits for joint runs, not merely the name of a moving branch.

Before A0, complete a separate technical bootstrap: create `dev` from the verified HEAD of `main`, configure branch and tag protection, establish working documents and necessary deterministic checks. Place this specification in `docs/development/plan.md` through a working branch and PR to `dev`. The proposed minimum working section consists of `README.md` (entry point and links to tasks) and `plan.md` (this plan). Develop the source and reference-case catalog in A0; add ADRs only for significant long-term decisions not adequately described in existing contracts. GitHub issues and PRs track concrete work; do not duplicate their statuses in Markdown. Technical workflow settings and precise A0 criteria may be supplemented in a separate PR. Do not include the working section in the user-facing site yet; in A5 preserve the historical plan and place current scientific and technical materials in appropriate Zensical sections.

After bootstrap, establish the corpus and reference cases, then implement the functionality. A new finding in a mandatory family first adds a case and expected result to the catalog. A substantial new capability outside the agreed scope is recorded separately and does not expand the development cycle silently. There are no intermediate releases or PyPI publications.

## 12. What was retained, changed and added

| Decision | Relative to the previous plan | Rationale |
|---|---|---|
| Separation of NistChemPy/NistChemData, principal public objects, pandas/dataclasses, JSONL, index, CLI | Retained | Already addresses most organizational needs |
| Dummies for the entire target functionality at the start | Retained and clarified | This is the development specification; the corpus is not limited to existing parsers |
| HTML isolation and shared helpers | Retained | Eliminates specific duplication and allows parsing to be tested without HTTP |
| Provenance and rights | Moved earlier | Fields are needed in A0 expected results and the first working export |
| Universal model of all measurements, common wrapper for every result, predetermined module tree | Simplified | Introduce a shared container/helper according to observed need; scientific distinctions remain explicit |
| Checking dummies against live pages | Clarified | Check meaningful features and availability of navigation paths; this is not a complete proof of equivalence |
| Warnings about live failures | Clarified | CI remains network-independent, but confirmed data corruption prevents acceptance |
| Continuations, individual measurements, multiple URLs per family | Added | Section names do not establish extraction completeness |
| Fix resumption, index completeness, file identity, and preservation of original GC HTML | Added | The repeated audit identified concrete paths to missing data and inability to reparse |
| Schema versions, commit pair, hashes, export-to-source associations | Added in minimal form | Allows the composition of the local dataset to be explained and processing reproduced |
| Permanent protected `main` and `dev`; PRs from working branches; no intermediate releases | Clarified on 27 November | Aligns the workflow with pyvoro2 and preserves the stable version until acceptance |
| Final Zensical documentation with a separate landing page and dark/light modes | Added on 27 November | Allows documentation to be reorganized once, after the functional contracts have stabilized |

## Audit sources

All code links are pinned to the inspected commits. NIST pages were inspected on 26 November 2026; this was a sample-based structural review, not a completed A0.

[p1]: https://github.com/muCommons/NistChemPy/blob/569c4ed951673ff09183df1fb19a10018e108e14/src/nistchempy/parsing/compound.py
[p2]: https://github.com/muCommons/NistChemPy/blob/569c4ed951673ff09183df1fb19a10018e108e14/src/nistchempy/compound.py
[p3]: https://github.com/muCommons/NistChemPy/blob/569c4ed951673ff09183df1fb19a10018e108e14/src/nistchempy/search.py
[p4]: https://github.com/muCommons/NistChemPy/tree/569c4ed951673ff09183df1fb19a10018e108e14/tests
[p5]: https://github.com/muCommons/NistChemPy/blob/569c4ed951673ff09183df1fb19a10018e108e14/.github/workflows/tests.yml
[p6]: https://github.com/muCommons/NistChemPy/blob/569c4ed951673ff09183df1fb19a10018e108e14/src/nistchempy/requests.py
[p7]: https://github.com/muCommons/NistChemPy/blob/569c4ed951673ff09183df1fb19a10018e108e14/src/nistchempy/records/base.py
[p8]: https://github.com/muCommons/NistChemPy/blob/569c4ed951673ff09183df1fb19a10018e108e14/src/nistchempy/indexing/builder.py
[p9]: https://github.com/muCommons/NistChemPy/blob/569c4ed951673ff09183df1fb19a10018e108e14/tools/check_package_artifacts.py
[d1]: https://github.com/muCommons/NistChemData/blob/50f069a835bc9b3a9415b9946d61e4ec5ee22203/scripts/common.py
[d2]: https://github.com/muCommons/NistChemData/blob/50f069a835bc9b3a9415b9946d61e4ec5ee22203/scripts/download_gas_chromatography.py
[d3]: https://github.com/muCommons/NistChemData/blob/50f069a835bc9b3a9415b9946d61e4ec5ee22203/scripts/download_spectra.py
[d4]: https://github.com/muCommons/NistChemData/blob/50f069a835bc9b3a9415b9946d61e4ec5ee22203/scripts/process_ir_spectra.py
[w1]: https://webbook.nist.gov/chemistry/guide/
[w2]: https://webbook.nist.gov/cgi/cbook.cgi?ID=C64175&Mask=27
[w3]: https://webbook.nist.gov/cgi/cbook.cgi?ID=C64175&Mask=8
[w4]: https://webbook.nist.gov/chemistry/fluid/
[w5]: https://webbook.nist.gov/cgi/inchi?ID=C19287457&Index=0&Type=IR-SPEC
[w6]: https://www.nist.gov/open/copyright-fair-use-and-licensing-statements-srd-data-software-and-technical-series-publications
