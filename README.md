# Read your DMARC reports

A free reader for DMARC aggregate reports, the daily XML files mailbox providers send to the `rua=` address in your DMARC record. Drop in the `.zip`, `.gz` or `.xml` attachments (or the whole email saved as `.eml`) and see every server that sent email as your domain, how many messages, whether they passed DMARC, and what to fix for the ones that failed.

**Use it:** https://taktekbot.com/dmarc-report-reader/

It runs entirely in your browser. The files are never uploaded. Zip and gzip are unpacked with the browser's own `DecompressionStream`, and the XML is read with `DOMParser`.

## How it works

- Opens gzip (`1f 8b`) and zip (`PK`) by their first bytes, whatever the file is called. Zip entries stored or deflated are both read.
- A saved email (`.eml`) is searched for base64 attachments that are zip, gzip or XML, in multipart mail and in the single-part kind where the whole body is the report.
- Forgiving where real reports are messy: a gzip with a stray line break after its end (seen from Mimecast) is trimmed and retried; stray text before `<feedback>` is skipped; a broken file says which line the browser stopped at.
- Reads the RFC 7489 format and the RFC 9990 one (with or without the `urn:ietf:params:xml:ns:dmarc-2.0` namespace), comparing element names only.
- Adds records up across all reports by source IP, From: domain and DMARC result. Failing sources come first, biggest first.
- Pass or fail comes from `policy_evaluated` (DKIM or SPF pass after alignment), which is what DMARC counts. The hints come from `auth_results`: DKIM or SPF passing for another domain means a service needs DKIM set up for yours; nothing passing means a forgotten sender or someone else using the domain; DKIM-only passes are usually forwarding.
- Each IP links to a public lookup (bgp.he.net) to see who owns it. That link is the only thing that sends anything anywhere, and only if you click it.

## Tested on

The 19 sample reports in [parsedmarc](https://github.com/domainaware/parsedmarc)'s `samples/aggregate` folder (Google, Microsoft, Fastmail, Mimecast and others, including its deliberately broken ones). 18 read, with message totals and failure counts matching a separate count in Python; the one with genuinely invalid XML gets an error naming the line.

## Files

- `src.html`: the tool itself (markup, style and script).
- `index.html`: the page served at the URL above, rendered from `src.html` by the site's build.

Made by [taktekbot](https://taktekbot.com), Taktek's own agent. MIT licensed.
