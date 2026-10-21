# problems.docspec.dev

Documentation for the [RFC 9457](https://www.rfc-editor.org/rfc/rfc9457) problem `type` URIs returned by the [DocSpec](https://github.com/docspec/docspec) HTTP API, served at <https://problems.docspec.dev/> by Cloudflare Pages.

- `index.html` is the one page; each problem type is a section whose `id` is the type name.
- `<name>/index.html` makes `https://problems.docspec.dev/<name>` resolve with a 200 and send the reader to `/#<name>`.
- `404.html` sends any other path to the matching anchor, or to the root.
- `wrangler.toml` names the Pages project and sets the output directory to the repository root; there is no build step.
- `CNAME` is the GitHub Pages leftover; it is harmless on Cloudflare and can go once the DNS has moved.

Source of truth: the "Problem types" table in [`crates/docspec-http/README.md`](https://github.com/docspec/docspec/blob/main/crates/docspec-http/README.md#problem-types). When a type is added to `PROBLEM_TYPE_BASE` constants in `docspec-http`, add its section here and a `<name>/index.html` stub in the same change.

## Maintaining

- [`docs/naming.md`](docs/naming.md): how to name a problem type, and the current names.
- [`docs/adding-a-type.md`](docs/adding-a-type.md): the steps in `docspec/docspec` and here when a type is added or deprecated.
- [`docs/redirects.md`](docs/redirects.md): how `/<name>` resolves on static hosting, the 404 fallback, the optional Cloudflare rule, DNS and HTTPS.
