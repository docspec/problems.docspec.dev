# How `/<name>` resolves

Cloudflare Pages serves this repository as static files, so the type URIs are made to work with three static pieces that work on any static host, no server rules.

## The one page

`index.html` is the whole documentation. Every type is a `<section>` whose `id` is the type name, so `https://problems.docspec.dev/#<name>` scrolls to it. Keep it one page: the taxonomy is small by design (see `naming.md`), and one page is the easiest thing to keep in step with the README table.

## Per-type stubs (the path form)

`https://problems.docspec.dev/<name>` is the canonical URI in problem bodies. Pages maps that path to `<name>/index.html` (via a 308 to the trailing-slash form), a stub that:

- answers **200**, so RFC 9457's "SHOULD provide documentation" holds even for clients that only check reachability;
- carries `<meta http-equiv="refresh" content="0; url=/#<name>">`, which sends browsers to the section without JavaScript;
- declares `<link rel="canonical" href="https://problems.docspec.dev/#<name>">`, so search engines index the page, not the stub;
- contains a plain link as the fallback for anything that ignores the refresh.

A fragment in a redirect target is legal and every browser honours it. Fragments are never sent to the server, which is why the stub must exist as a file: nothing on the server can see `#<name>`.

## Unknown paths

`404.html` at the root is what Pages serves for any path with no file. It reads `location.pathname`, and if it is a single `[a-z0-9-]+` segment it does `location.replace("/#" + name)`, otherwise it goes to `/`. This needs JavaScript and returns status 404, so it is a convenience for typos and for types that exist in a newer docspec release than this site, not the mechanism the stubs provide.

## Optional: Redirect Rule

A zone Redirect Rule can replace the stubs with a clean 301 before the request reaches Pages:

- Rules → Redirect Rules → wildcard pattern.
- Request URL: `https://problems.docspec.dev/*`
- Target: `https://problems.docspec.dev/#${1}`, status 301, query string not preserved.
- Filter: path is not `/` and does not start with `/assets/`.

The stubs stay in the repository either way; they cost nothing and keep the URIs working if the rule is ever removed.

## Hosting, DNS and HTTPS

- Pages project `problems-docspec-dev`, connected to `github.com/docspec/problems.docspec.dev`, production branch `main`, build command empty, build output directory `/` (also in `wrangler.toml`). Every push to `main` deploys; other branches get preview URLs.
- First-time setup, from a machine logged in to Cloudflare:

  ```sh
  npx wrangler login
  npx wrangler pages project create problems-docspec-dev --production-branch main
  npx wrangler pages deploy            # manual deploy; connect the Git repo in the dashboard for automatic ones
  ```

- Custom domain: Pages project → Custom domains → add `problems.docspec.dev`. Because the `docspec.dev` zone is on Cloudflare, the CNAME to `problems-docspec-dev.pages.dev` and the certificate are created for you; HTTPS is enforced by default. If the zone still has the GitHub Pages CNAME to `docspec.github.io`, remove it first.
- Verify: `curl -sI https://problems.docspec.dev/` returns 200 with `content-type: text/html`, `curl -sIL https://problems.docspec.dev/malformed-input | grep -c 'HTTP/2 200'` is 1, and `curl -sI https://problems.docspec.dev/no-such-type | head -1` is 404.
