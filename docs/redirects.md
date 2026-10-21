# How `/<name>` resolves

GitHub Pages serves static files only, so the type URIs are made to work with three static pieces, no server rules.

## The one page

`index.html` is the whole documentation. Every type is a `<section>` whose `id` is the type name, so `https://problems.docspec.dev/#<name>` scrolls to it. Keep it one page: the taxonomy is small by design (see `naming.md`), and one page is the easiest thing to keep in step with the README table.

## Per-type stubs (the path form)

`https://problems.docspec.dev/<name>` is the canonical URI in problem bodies. GitHub Pages maps that path to `<name>/index.html`, a stub that:

- answers **200**, so RFC 9457's "SHOULD provide documentation" holds even for clients that only check reachability;
- carries `<meta http-equiv="refresh" content="0; url=/#<name>">`, which sends browsers to the section without JavaScript;
- declares `<link rel="canonical" href="https://problems.docspec.dev/#<name>">`, so search engines index the page, not the stub;
- contains a plain link as the fallback for anything that ignores the refresh.

A fragment in a redirect target is legal and every browser honours it. Fragments are never sent to the server, which is why the stub must exist as a file: nothing on the server can see `#<name>`.

## Unknown paths

`404.html` is what Pages serves for any path with no file. It reads `location.pathname`, and if it is a single `[a-z0-9-]+` segment it does `location.replace("/#" + name)`, otherwise it goes to `/`. This needs JavaScript and returns status 404, so it is a convenience for typos and for types that exist in a newer docspec release than this site, not the mechanism the stubs provide.

## Optional: Cloudflare

If the DNS record is proxied through Cloudflare, a Redirect Rule can replace the stubs with a clean 301:

- Rules → Redirect Rules → wildcard pattern.
- Request URL: `https://problems.docspec.dev/*`
- Target: `https://problems.docspec.dev/#${1}`, status 301, query string not preserved.
- Filter: path is not `/` and does not start with `/assets/`.
- Requirements: proxied (orange-cloud) record and SSL mode Full, so the hop to GitHub Pages stays encrypted.

The stubs stay in the repository either way; they cost nothing and keep the URIs working if the rule is ever removed.

## DNS and HTTPS

- DNS: `problems.docspec.dev` CNAME → `docspec.github.io`.
- Pages: source `main`, path `/`, custom domain from `CNAME`. Once DNS resolves, GitHub provisions the certificate; then enable "Enforce HTTPS" in the repository's Pages settings (or `gh api -X PUT repos/docspec/problems.docspec.dev/pages -F https_enforced=true`).
- Verify: `curl -sI https://problems.docspec.dev/` returns 200 with `content-type: text/html`, and `curl -sI http://problems.docspec.dev/` returns a 301 to https.
