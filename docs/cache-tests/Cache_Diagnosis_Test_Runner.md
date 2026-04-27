# Cache Diagnosis Test Runner
## Goal: Identify which exact cache layer is serving stale Guide content

**Time required:** ~30 minutes (mostly waiting between fetches for cache state to settle)  
**Outcome:** A definitive map of cache behavior for YOUR Anthropic account, so we can pick the right fix without guessing.

---

## PRE-TEST SETUP (do this once, ~5 minutes)

### Step 1: Push all 5 test files to GitHub

Copy these 5 files (all are in your downloads from this conversation) into your repo at `/docs/cache-tests/`:

| Source filename | Push to repo as |
|-----------------|-----------------|
| `test-A.md` | `docs/cache-tests/test-A.md` |
| `test-B.md` | `docs/cache-tests/test-B.md` |
| `test-C-v1.md` | `docs/cache-tests/test-C-v1.md` |
| `test-C-v2.md` | `docs/cache-tests/test-C-v2.md` |
| `test-D.md` | `docs/cache-tests/test-D.md` |

(Don't push `test-A-UPDATE.md` yet — it's used during Test 1 to update test-A.md.)

Commit and push to the `main` branch.

### Step 2: Verify in your browser

Open these 5 URLs in your browser (NOT here in chat). Make sure each one returns what you expect:

1. https://raw.githubusercontent.com/jsd4026/tableland-partners-copilot/main/docs/cache-tests/test-A.md → should show `TEST-A VERSION 1.0`
2. https://raw.githubusercontent.com/jsd4026/tableland-partners-copilot/main/docs/cache-tests/test-B.md → should show `TEST-B VERSION 1.0`
3. https://raw.githubusercontent.com/jsd4026/tableland-partners-copilot/main/docs/cache-tests/test-C-v1.md → should show `TEST-C VERSION 1.0`
4. https://raw.githubusercontent.com/jsd4026/tableland-partners-copilot/main/docs/cache-tests/test-C-v2.md → should show `TEST-C VERSION 2.0`
5. https://raw.githubusercontent.com/jsd4026/tableland-partners-copilot/main/docs/cache-tests/test-D.md → should show `TEST-D VERSION 1.0 INITIAL`

If all 5 look correct in your browser, ground truth is confirmed. **Wait 5 more minutes** before starting tests, to let GitHub's CDN settle globally.

### Step 3: Tell me you're ready

Reply with: "Tests ready, all 5 files verified in browser." I'll start Test 1.

---

## THE TESTS (run in order, don't skip)

### TEST 1 — Does query-string cache busting work?

**What we're testing:** Whether appending `?v=X` or `?t=X` to a URL forces a fresh fetch.

**Steps:**

**1.1** Paste this in chat after telling me you're ready:

```
Run Test 1.1: web_fetch this URL and report exactly what version line comes back: https://raw.githubusercontent.com/jsd4026/tableland-partners-copilot/main/docs/cache-tests/test-A.md
```

I'll fetch and report the version line.

**1.2** Now update the file on GitHub. Replace the contents of `docs/cache-tests/test-A.md` with the contents of `test-A-UPDATE.md` (which says `TEST-A VERSION 2.0`). Commit and push.

**1.3** Wait 2 minutes. Verify in your browser the URL now shows `TEST-A VERSION 2.0`.

**1.4** Paste this in chat:

```
Run Test 1.4: web_fetch this URL (no query string) and report the version: https://raw.githubusercontent.com/jsd4026/tableland-partners-copilot/main/docs/cache-tests/test-A.md
```

**1.5** Paste this in chat:

```
Run Test 1.5: web_fetch this URL (with v query string) and report the version: https://raw.githubusercontent.com/jsd4026/tableland-partners-copilot/main/docs/cache-tests/test-A.md?v=2
```

**1.6** Paste this in chat:

```
Run Test 1.6: web_fetch this URL (with t query string) and report the version: https://raw.githubusercontent.com/jsd4026/tableland-partners-copilot/main/docs/cache-tests/test-A.md?t=99887766
```

**Record results in the table at the bottom of this doc.**

---

### TEST 2 — Do version-stamped filenames work in same conversation?

**What we're testing:** Whether different filenames on the same domain are treated as separate fetches.

**Steps:**

**2.1** Paste this in chat:

```
Run Test 2.1: web_fetch and report the version line from: https://raw.githubusercontent.com/jsd4026/tableland-partners-copilot/main/docs/cache-tests/test-C-v1.md
```

**2.2** Paste this in chat:

```
Run Test 2.2: web_fetch and report the version line from: https://raw.githubusercontent.com/jsd4026/tableland-partners-copilot/main/docs/cache-tests/test-C-v2.md
```

Both should return their respective versions.

---

### TEST 3 — Does the cache cross conversations?

**What we're testing:** Whether the v1.0 result from Test 1 stays cached when accessed from a NEW conversation in the same project.

**Steps:**

**3.1** Open a NEW conversation in this same project (the AI Essentials Roundtable project). Don't change Project Instructions or anything — just start a new chat.

**3.2** Paste this in the new conversation:

```
Run a cache test for me. Use web_fetch on this URL and report exactly what version line comes back, no other commentary:

https://raw.githubusercontent.com/jsd4026/tableland-partners-copilot/main/docs/cache-tests/test-A.md
```

**3.3** Note the result. Come back to THIS conversation and tell me what version came back.

---

### TEST 4 — Does the cache cross projects?

**What we're testing:** Whether a brand new project (which has never fetched this URL) returns fresh content or stale.

**Steps:**

**4.1** Create a brand new throwaway project in your Claude account. Don't add any files. Don't paste any project instructions. Just create it bare and name it "Cache Test Throwaway."

**4.2** Start a conversation in that throwaway project and paste:

```
Run a cache test for me. Use web_fetch on this URL and report exactly what version line comes back, no other commentary:

https://raw.githubusercontent.com/jsd4026/tableland-partners-copilot/main/docs/cache-tests/test-A.md
```

**4.3** Note the result. Come back here and tell me what version came back.

---

### TEST 5 — Does a never-before-fetched URL return fresh content?

**What we're testing:** Whether a URL Anthropic has truly never seen does a fresh fetch.

This test must run in the throwaway project from Test 4 (so it's a maximally fresh context).

**Steps:**

**5.1** In the throwaway project, in a new conversation (or the same one is fine), paste:

```
Use web_fetch on this URL and report exactly what version line comes back: https://raw.githubusercontent.com/jsd4026/tableland-partners-copilot/main/docs/cache-tests/test-D.md
```

**5.2** Note the result. Come back here and tell me what came back.

---

## RESULT TABLE — fill this in as you go

Copy this into your reply when you've run all the tests:

```
TEST 1 RESULTS:
  1.1 (initial fetch, no query string):                 [version line]
  1.4 (after GitHub update, no query string):           [version line]
  1.5 (after GitHub update, ?v=2):                      [version line]
  1.6 (after GitHub update, ?t=99887766):               [version line]

TEST 2 RESULTS:
  2.1 (test-C-v1.md):                                   [version line]
  2.2 (test-C-v2.md):                                   [version line]

TEST 3 RESULTS:
  3.2 (test-A.md from new conversation, same project):  [version line]

TEST 4 RESULTS:
  4.2 (test-A.md from brand new project):               [version line]

TEST 5 RESULTS:
  5.1 (test-D.md from throwaway project):               [version line]
```

---

## What I'll do with the results

After you paste the result table, I'll diagnose exactly which cache layer is the culprit and design a fix that requires ZERO member action. The fix will:

- Live entirely on the GitHub side (and possibly a Cloudflare Worker)
- Auto-version the URL programmatically as you push Guide updates
- Require nothing from you except your normal Guide commit workflow
- Require nothing from members ever

The diagnosis dictates which mechanism we use:

- **If query strings work (Test 1.5 or 1.6 returns v2.0):** Simple GitHub Actions fix. You commit Guide.md, the action computes a content hash, the URL gets `?v={hash}` automatically.
- **If only filename changes work (Test 2 succeeds, Test 1 fails):** GitHub Actions auto-renames the file to include version, members reference a stable `current.json` pointer.
- **If even filename changes fail (Test 5 returns wrong content):** We deploy a Cloudflare Worker that proxies the file and serves it with cache-defeating headers + a unique URL path per request.

We won't know which path until we have results. That's why the tests matter.

---

## Cleanup after testing

Once we've identified the fix and you're confident it works, you can delete the entire `/docs/cache-tests/` directory from your repo. Or leave it — it's a tiny diagnostic artifact and useful if Anthropic changes their cache behavior in the future and you need to re-test.
