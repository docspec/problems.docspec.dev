# Adding a problem type

Two repositories change together. Land the `docspec` change first; its tests pin the URI, and this site must serve the URI by the time that release ships.

## 1. In `docspec/docspec`

All in `crates/docspec-http`:

1. `src/error.rs`: add a `PROBLEM_TYPE_<NAME>` constant under `PROBLEM_TYPE_BASE`, a variant on `InputProblem` (or a new enum if the type is not a 422), and the arm in `type_uri()`.
2. Map the error kind to the variant where errors are classified (`handlers/conversion/mod.rs`, `classify_conversion_error`, or the relevant handler). Mapping is by `docspec_core::Error` variant, never by message text.
3. Tests: the `type_uri()` value, the classification per error kind, and one integration test that asserts the exact problem body including `type` and `instance`.
4. `README.md`, "Problem types" table: add the row. That table is the source of truth for this site.
5. Commit as a breaking change if the shape of `HttpError` changes; as `feat(http)` if only a new type is added.

## 2. In this repository

1. `index.html`: add a `<section id="<name>">` in the same shape as the existing ones: heading with a self-link, then Type, Status, When, What to do. Keep the wording identical to the README row's "When" column; "What to do" is the one place to add advice for the caller.
2. `<name>/index.html`: copy an existing stub and replace the name in the three places it appears. This is what makes `https://problems.docspec.dev/<name>` answer 200 and land on the section (see `redirects.md`).
3. `docs/naming.md`, "Current names" table: add the row.
4. Commit, push to `main`; Pages deploys in about a minute. Verify with:

```sh
curl -sIL https://problems.docspec.dev/<name> | grep -c 'HTTP/2 200'   # 1
curl -s https://problems.docspec.dev/ | grep -c 'id="<name>"'  # 1
```

## Deprecating a type

Do not delete anything. Add "Deprecated since vX.Y: use `<other>`" to the section's "When" entry and leave the stub in place. The URI must keep resolving for as long as any released docspec version can emit it.
