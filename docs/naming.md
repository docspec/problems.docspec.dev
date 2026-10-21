# Naming problem types

A problem type is the `type` member of an RFC 9457 problem body. Its URI is a permanent identifier that integrators switch on in code. Choose names as if they can never change, because they cannot.

## Form

- The URI is `https://problems.docspec.dev/<name>`.
- `<name>` is lower-case ASCII letters, digits and hyphens only: `[a-z0-9-]+`. No underscores, dots, slashes, upper case or Unicode. The `404.html` fallback and the stub directories rely on this.
- Two to three words, hyphen-separated: `malformed-input`, `unsupported-content`.

## Meaning

- Name the **class of problem the caller has**, not the component that found it and not the message. `malformed-input`, not `parse-error` or `quick-xml-error`.
- Name what is wrong with the request or document, as a noun phrase. No verbs (`cannot-parse`), no status codes (`error-422`), no `-error` or `-failure` suffix: every type is a failure.
- Keep it format-neutral unless the problem genuinely exists for one format only. `malformed-input` covers Markdown, HTML and DOCX alike; a `docx-encrypted` type would be fair game because only DOCX can be encrypted.
- Chosen by **error kind**, never by message text. If telling two cases apart would require matching on the wording of a message, they are one type.

## Granularity

- A type earns its place only when it adds meaning beyond the HTTP status code. Where the status already says everything (400, 404, 405, 406, 415, 500), the type is `about:blank` and the title is the reason phrase. Do not add `not-found` or `unsupported-media-type` types.
- Prefer few, broad types that integrators can act on over many narrow ones they will collapse anyway. "The file is broken" versus "the file contains something we cannot convert" is the distinction users act on today.
- A type maps to exactly one status code. If a new case needs a different status, it is a different type.

## Stability

- Never rename or repurpose a type. To change meaning, add a new name and keep the old one documented as deprecated, still served at its URI.
- The name must make sense without its page. Treat the page as help, not as the definition.

## Current names

| Name | Status | Core error kinds |
| --- | --- | --- |
| `malformed-input` | 422 | `Parse`, `Json` |
| `unsupported-content` | 422 | `Other`, `Io`, any future kind |
