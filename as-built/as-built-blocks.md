# As-built blocks

Copy these into the `<!-- ASBUILT:CONTENT -->` slot of `template.html`. Compose only
the blocks the recap needs. **Use real names and real status** — derive every verdict
from the actual code, not from the plan's optimism.

Rules the template handles for you (do NOT hand-write them):
- Section **numbers** (`01`, `02`…) — leave `<span class="num"></span>` empty.
- The **recap index** (left rail) — auto-built from section `<h2>`s.
- **Comment buttons**, counts, the fix drawer, the plan toggle.

You only write: the hero, then one `<section id="...">` per part of the recap.
Every section needs a unique `id` and a `<div class="sec-head">` with an empty `.num`.

## Status vocabulary (use everywhere)

| Class | Pill text | Meaning |
|-------|-----------|---------|
| `ok`  | `✓ As planned` | Built, matches the plan |
| `dev` | `≈ Changed`    | Built, but differs from the plan (a deviation) |
| `gap` | `✗ Not built`  | Planned, missing or incomplete |
| `add` | `+ Added`      | Shipped, was not in the plan |

---

## Hero (always first, no `id`) — leads with the build verdict

```html
<header class="hero section">
  <div class="klabel">As-built · <span style="color:var(--text)">feat/SLUG</span> · checked against plan.md</div>
  <h1>Short Title<br>“what shipped”</h1>
  <p class="lede">One sentence on what actually works now, naming the <em>one gap or deviation</em> that matters most.</p>
  <div class="meta-grid">
    <div class="meta"><span class="ic">◳</span> <b>1</b> table shipped</div>
    <div class="meta"><span class="ic">⇄</span> <b>2/3</b> endpoints</div>
    <div class="meta"><span class="ic">⌘</span> <b>5</b> files changed</div>
    <div class="meta"><span class="ic">◷</span> commit <b>a1b2c3d</b></div>
  </div>
</header>
```

## Build scorecard (recommended section 01) — counts + ship verdict

One `.score` per status; numbers must add up to the planned-item count (+ adds).

```html
<section class="section" id="status">
  <div class="sec-head"><span class="num"></span><h2>Build status</h2><span class="rule"></span></div>
  <div class="panel">
    <div class="scorecard">
      <div class="score ok"><div class="big">6</div><div class="lbl">✓ as planned</div></div>
      <div class="score dev"><div class="big">1</div><div class="lbl">≈ changed</div></div>
      <div class="score gap"><div class="big">1</div><div class="lbl">✗ not built</div></div>
      <div class="score add"><div class="big">2</div><div class="lbl">+ added</div></div>
    </div>
    <div class="verdict caveat">
      <span class="big">Ships with caveats.</span>
      <span>Core flow works end to end. One planned item (<code>delete own comment</code>) is unbuilt and one decision changed — see below.</span>
    </div>
  </div>
</section>
```
Verdict variants: `ship` (all green), `caveat` (gaps/deviations exist), `block` (a required item is missing).

## Planned → As-built verification (the core block)

The heart of the recap. One `.vrow` per planned capability. Status pill, what the
plan said, what actually shipped. Use `gap`/`add`/`dev` modifier on the `.vrow` too.

```html
<section class="section" id="verify">
  <div class="sec-head"><span class="num"></span><h2>Planned → built</h2><span class="rule"></span></div>
  <div class="panel verify">

    <div class="vrow ok">
      <div class="st-cell"><span class="stat ok">✓ As planned</span></div>
      <div class="vc"><span class="cap">Planned</span><div class="item">Post a comment</div><div class="planned">Composer under the post; optimistic insert.</div></div>
      <div class="vc"><span class="cap">As-built</span><div class="built"><code>CommentBox.tsx</code> posts to <code>POST /api/comments</code>, optimistic add in place.</div></div>
    </div>

    <div class="vrow dev">
      <div class="st-cell"><span class="stat dev">≈ Changed</span></div>
      <div class="vc"><span class="cap">Planned</span><div class="item">Store newest-first</div><div class="planned">Order by <code>created_at desc</code>.</div></div>
      <div class="vc"><span class="cap">As-built</span><div class="built">Shipped oldest-first to match the thread reading order — see Deviations.</div></div>
    </div>

    <div class="vrow gap">
      <div class="st-cell"><span class="stat gap">✗ Not built</span></div>
      <div class="vc"><span class="cap">Planned</span><div class="item">Delete own comment</div><div class="planned"><code>DELETE /api/comments/:id</code> with owner check.</div></div>
      <div class="vc"><span class="cap">As-built</span><div class="built">Route stubbed, no UI. Deferred — see Follow-ups.</div></div>
    </div>

    <div class="vrow add">
      <div class="st-cell"><span class="stat add">+ Added</span></div>
      <div class="vc"><span class="cap">Planned</span><div class="planned" style="color:var(--faint)">— not in the plan —</div></div>
      <div class="vc"><span class="cap">As-built</span><div class="built">Empty state (“Be the first to comment”) — added during build.</div></div>
    </div>

  </div>
</section>
```

## Deviations & gaps — the "why" behind every non-green row

```html
<section class="section" id="deltas">
  <div class="sec-head"><span class="num"></span><h2>Deviations &amp; gaps</h2><span class="rule"></span></div>
  <div class="panel">
    <div class="dev-card dev">
      <p class="dt"><span class="stat dev">≈ Changed</span> Comment ordering</p>
      <p>Plan said newest-first; shipped oldest-first so replies read top-to-bottom.</p>
      <p class="why"><b>Why:</b> matches every other thread in the app. Cheap to flip if you disagree.</p>
    </div>
    <div class="dev-card gap">
      <p class="dt"><span class="stat gap">✗ Not built</span> Delete own comment</p>
      <p>Endpoint exists but returns 501; no button wired up.</p>
      <p class="why"><b>Why:</b> ran out of scope this pass. Owner-check logic is the only unknown left.</p>
    </div>
  </div>
</section>
```

## Schema as shipped — final shape (badge `ship`; mark unplanned columns `.added`)

```html
<section class="section" id="data">
  <div class="sec-head"><span class="num"></span><h2>Schema — as shipped</h2><span class="rule"></span></div>
  <div class="panel schema">
    <div class="tname">▦ comments <span class="badge ship">Shipped</span></div>
    <div class="col-row"><span class="cn pk">id</span><span class="ct">uuid · pk</span><span class="cd">default gen_random_uuid()</span></div>
    <div class="col-row"><span class="cn">post_id</span><span class="ct">uuid · fk</span><span class="cd">→ posts.id</span></div>
    <div class="col-row added"><span class="cn">deleted_at</span><span class="ct">timestamptz</span><span class="cd">added — soft delete, not in plan</span></div>
  </div>
</section>
```

## API as shipped — what each route actually returns

Use the verbs `post green`, `get cyan`, `del red`, `put amber`. Note 501/missing in `.ep-desc`.

```html
<section class="section" id="api">
  <div class="sec-head"><span class="num"></span><h2>API — as shipped</h2><span class="rule"></span></div>
  <div class="panel" style="padding:16px">
    <div class="endpoint open" onclick="epToggle(this)">
      <div class="ep-head"><span class="verb post">POST</span><span class="ep-path">/api/comments</span><span class="ep-desc">live</span><span class="chev">›</span></div>
      <div class="ep-body"><pre><span class="cm"># returns the full row, author joined (changed from plan)</span>
{ <span class="k">"id"</span>: <span class="v">"uuid"</span>, <span class="k">"author"</span>: {…} }</pre></div>
    </div>
    <div class="endpoint" onclick="epToggle(this)">
      <div class="ep-head"><span class="verb del">DELETE</span><span class="ep-path">/api/comments/:id</span><span class="ep-desc" style="color:var(--red)">501 — stubbed</span><span class="chev">›</span></div>
      <div class="ep-body"><pre><span class="cm"># planned, not implemented</span></pre></div>
    </div>
  </div>
</section>
```

## Files actually changed — real git diff (tags: new / mod / unplanned)

```html
<section class="section" id="files">
  <div class="sec-head"><span class="num"></span><h2>Files changed</h2><span class="rule"></span></div>
  <div class="panel tree">
    <div class="row"><span class="dir">app/api/comments/</span></div>
    <div class="row"><span class="ind">└─</span><span class="fn">route.ts</span><span class="tag new">new</span></div>
    <div class="row"><span class="dir">app/components/</span></div>
    <div class="row"><span class="ind">└─</span><span class="fn">CommentBox.tsx</span><span class="tag new">new</span></div>
    <div class="row"><span class="ind">└─</span><span class="fn">EmptyState.tsx</span><span class="tag unplanned">unplanned</span></div>
  </div>
</section>
```

## Annotated code — the SHIPPED code, with what to watch

`.hl` highlights the line that deviated or carries risk.

```html
<section class="section" id="code">
  <div class="sec-head"><span class="num"></span><h2>Key code — as shipped</h2><span class="rule"></span></div>
  <div class="code-wrap">
<pre class="code"><span class="ln">1</span><span class="kw">export async function</span> <span class="fn">addComment</span>(body) {
<span class="hl"><span class="ln">2</span>  setComments(c => [...c, optimistic]) <span class="cm">// optimistic, as planned</span></span>
<span class="ln">3</span>}</pre>
    <div class="annos">
      <div class="anno"><div class="tagn"><span class="b">A</span> line 2</div><p>Matches the plan — but no rollback on failure yet. Candidate follow-up.</p></div>
    </div>
  </div>
</section>
```

## Follow-ups & remaining debt (recommended last section)

What's deferred, stubbed, or newly owed. Reuses the question card, no picker.

```html
<section class="section" id="followups">
  <div class="sec-head"><span class="num"></span><h2>Follow-ups</h2><span class="rule"></span></div>
  <div class="q">
    <p class="qq">Wire up delete (owner check + button)</p>
    <div class="rec">carried over from the plan · ~Small</div>
  </div>
  <div class="q">
    <p class="qq">Optimistic rollback on POST failure</p>
    <div class="rec">new debt found during build</div>
  </div>
</section>
```

## Plain note (use anywhere inside a `.panel`)

```html
<div class="panel"><p class="note">A short paragraph. <b>Bold</b> for emphasis.</p></div>
```
