# Fetching

Recovery techniques when fetching from GitHub fails. Used by any
project that points an LLM at GitHub-hosted memory or protocols.

## raw.githubusercontent.com returns 404 for a file that exists

Almost always Fastly's cache (negative or stale). The file is on
origin; the CDN edge serving the request returned 404 (or older
content) at some prior moment and is still serving that response.
GitHub does not expose a public purge API for raw content.

Try, in order:

1. **Cache-busting query string.** `?nocache=N` or `?v=2` or
   `?t=<timestamp>` — raw ignores unknown query params on origin,
   but Fastly keys its cache by full URL including query, so this
   reliably misses the poisoned entry:
   `https://raw.githubusercontent.com/<owner>/<repo>/main/<path>?nocache=1`

2. **`refs/heads/main` URL form.** Different cache key than
   `.../main/<path>`, often unaffected by the same poisoned entry:
   `https://raw.githubusercontent.com/<owner>/<repo>/refs/heads/main/<path>`

3. **Commit-SHA URL form.** Find the latest commit touching the file
   and substitute the SHA for the branch name. Different cache key.

4. **`github.com` blob page + extract `rawLines`.** The blob page
   `https://github.com/<owner>/<repo>/blob/<branch>/<path>` is served
   by a different backend, and embeds the file content as a JSON
   array in the page HTML. Parse the page, extract `rawLines`, join
   with newlines. Reliable when raw is broken; survives even when
   raw 404s on every URL form.

5. **Wait it out.** GitHub's nominal cache TTL on raw is 5 minutes,
   but in practice some stuck entries persist for hours (reports of
   `max-age=86400` exist). If the request is not time-sensitive,
   waiting is cheap.

6. **Last resort: `raw.githack.com`.** Third-party caching proxy
   sitting on Cloudflare, with its own purge endpoint
   (`curl -X DELETE https://raw.githack.com/purge?url=<url>`).
   Different CDN, different cache state. Adds a third-party
   dependency; use only when the file is unreachable through every
   GitHub-native path above.

### Push-based bust does NOT reliably work

A push that modifies the file does **not** reliably invalidate its
Fastly cache entry. Confirmed in practice (2026-05-03): a real
one-line diff committed and pushed to `launch-block.md` did not
restore raw access for that URL — fresh sessions continued to see
404 on `.../main/launch-block.md` while `.../main/README.md`
(modified in adjacent commits) responded normally.

Do NOT recommend "operator pushes a contentful diff" as a recovery.
It may help; it may not; the cache is opaque to us. Prefer the
techniques above, all of which sidestep the stuck entry rather than
trying to evict it.

## Fetch tool refuses constructed URLs

Some fetch tools only follow URLs that already appeared in the
conversation. Claude's `web_fetch` on claude.ai is the reference
case. Constructing a child URL from a `ref` field in a parent JSON
will be rejected even if the URL is well-formed.

Workarounds:

1. **Seed URLs in the README.** A project following `project-memory.md`
   should list its key URLs in a "Direct entry URLs" section
   (Principle 9). The fetch tool sees those URLs in the README and
   then can follow them.

2. **Shell-out to a less-restricted tool.** If the fetch tool
   restricts URLs but a shell does not, the shell can fetch via
   `curl` or `git clone`. Combine with technique 4 above when raw
   itself is broken.

3. **Ask the operator to paste the URL.** Once the URL is in the
   conversation, the fetch tool will accept it.

## raw is reachable but tool persistently 404s in this session

Possible internal cache inside the fetch tool. To distinguish from
Fastly issues, ask the operator to test the URL in a fresh session.
If it works there but not here, the cache is local to the tool's
session. Workaround: switch to shell-based fetching for the rest of
the session, or use a cache-busting query string (technique 1 above)
which also evades same-session caches.

## Authentication required

If the repo is private, no anonymous fetch will work. The operator
must provide an authorized URL or paste the content directly.