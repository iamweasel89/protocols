# Fetching

Recovery techniques when fetching from GitHub fails. Used by any
project that points an LLM at GitHub-hosted memory or protocols.

## raw.githubusercontent.com returns 404 for a file that exists

Almost always Fastly's negative cache. The file is on origin; the CDN
edge serving the request returned 404 at some prior moment and is
still serving that response.

Try, in order:

1. **`refs/heads/main` URL form.** Different cache key than
   `.../main/<path>`, often unaffected by the same poisoned entry:
   `https://raw.githubusercontent.com/<owner>/<repo>/refs/heads/main/<path>`

2. **Commit-SHA URL form.** Find the latest commit touching the file
   and substitute the SHA for the branch name. Different cache key.

3. **`github.com` blob page + extract `rawLines`.** The blob page
   `https://github.com/<owner>/<repo>/blob/<branch>/<path>` is served
   by a different backend, and embeds the file content as a JSON
   array in the page HTML. Parse the page, extract `rawLines`, join
   with newlines.

4. **Ask the operator to bust the cache.** A push that modifies the
   file invalidates its cache entry. An empty commit
   (`git commit --allow-empty`) does NOT — purge is per changed file.
   The smallest reliable diff is a trailing newline.

## Fetch tool refuses constructed URLs

Some fetch tools only follow URLs that already appeared in the
conversation. Claude's `web_fetch` on claude.ai is the reference case.
Constructing a child URL from a `ref` field in a parent JSON will be
rejected even if the URL is well-formed.

Workarounds:

1. **Seed URLs in the README.** A project following `project-memory.md`
   should list its key URLs in a "Direct entry URLs" section
   (Principle 9). The fetch tool sees those URLs in the README and
   then can follow them.

2. **Shell-out to a less-restricted tool.** If the fetch tool
   restricts URLs but a shell does not, the shell can fetch via
   `curl` or `git clone`. Combine with technique 3 above when raw
   itself is broken.

3. **Ask the operator to paste the URL.** Once the URL is in the
   conversation, the fetch tool will accept it.

## raw is reachable but tool persistently 404s in this session

Possible internal cache inside the fetch tool. To distinguish from
Fastly issues, ask the operator to test the URL in a fresh session.
If it works there but not here, the cache is local to the tool's
session. Workaround: switch to shell-based fetching for the rest of
the session.

## Authentication required

If the repo is private, no anonymous fetch will work. The operator
must provide an authorized URL or paste the content directly.