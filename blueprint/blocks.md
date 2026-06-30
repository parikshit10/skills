# Blueprint blocks

Copy these into the `<!-- BLUEPRINT:CONTENT -->` slot of `template.html`. Compose
only the blocks the plan needs. **Use real names** — real files, real columns,
real endpoints — never placeholders.

Rules the template handles for you (do NOT hand-write them):
- Section **numbers** (`01`, `02`…) — leave `<span class="num"></span>` empty.
- The **plan index** (left rail) — auto-built from section `<h2>`s.
- **Comment buttons**, counts, the feedback drawer, the markdown toggle.

You only write: the hero, then one `<section id="...">` per part of the plan.
Every section needs a unique `id` and a `<div class="sec-head">` with an empty `.num`.

---

## Hero (always first, no `id`)

```html
<header class="hero section">
  <div class="klabel">Feature plan · branch <span style="color:var(--text)">feat/SLUG</span></div>
  <h1>Short Title<br>“optional subtitle”</h1>
  <p class="lede">One concrete sentence on what the user can do after this ships, with the <em>key nuance</em> highlighted.</p>
  <div class="meta-grid">
    <div class="meta"><span class="ic">◳</span> <b>1</b> new table</div>
    <div class="meta"><span class="ic">⇄</span> <b>3</b> new endpoints</div>
    <div class="meta"><span class="ic">⌘</span> <b>4</b> files · <b>2</b> new</div>
    <div class="meta"><span class="ic">◴</span> est. <b>~Small</b></div>
  </div>
</header>
```

## Outcome & delta — reuse-first framing (recommended section 01)

```html
<section class="section" id="outcome">
  <div class="sec-head"><span class="num"></span><h2>Outcome &amp; delta</h2><span class="rule"></span></div>
  <div class="panel reuse">
    <div class="col reuses"><h4>↻ Reuses (no new infra)</h4>
      <ul><li><span class="tick">›</span><code>existing_thing</code> — unchanged</li></ul>
    </div>
    <div class="col news"><h4>+ Genuinely new</h4>
      <ul><li><span class="plus">+</span><code>new_thing</code> — what it adds</li></ul>
    </div>
  </div>
</section>
```

## Screens — wireframe (UI work only; skip for backend-only plans)

`wf(this,'id')` switches states. Each state is a `.wf` (first one `show`). Build a
schematic mock with `.app` chrome — greyed bars, not real copy.

```html
<section class="section" id="screens">
  <div class="sec-head"><span class="num"></span><h2>Screens</h2><span class="rule"></span></div>
  <div class="wf-tabs">
    <button class="active" onclick="wf(this,'wfA')">▢ Default</button>
    <button onclick="wf(this,'wfB')">★ Other state</button>
  </div>
  <div class="wf-stage">
    <div id="wfA" class="wf show">
      <div class="app">
        <div class="app-top"><span class="logo">AppName.</span><div class="nav"><span class="on">Tab</span><span>Tab</span></div></div>
        <div class="art"><div class="thumb"></div><div class="body"><div class="t"></div><div class="s"></div><div class="s s2"></div></div><div class="bk" onclick="this.classList.toggle('saved')">☆</div></div>
      </div>
    </div>
    <div id="wfB" class="wf"> ... second state ... </div>
  </div>
</section>
```

## Flow — SVG diagram (architecture / data flow)

Horizontal node chain. `.node.accent` = client, plain = server, `.node.store` = DB.

```html
<section class="section" id="flow">
  <div class="sec-head"><span class="num"></span><h2>Flow</h2><span class="rule"></span></div>
  <div class="panel">
    <svg class="diagram" viewBox="0 0 900 180">
      <defs><marker id="arr" markerWidth="9" markerHeight="9" refX="7" refY="4.5" orient="auto"><path d="M0,0 L9,4.5 L0,9 z" fill="#5cc9e6"/></marker></defs>
      <g class="node accent"><rect x="14" y="58" width="170" height="60" rx="11"/><text x="99" y="84" text-anchor="middle" font-size="13">Step one</text><text x="99" y="103" text-anchor="middle" class="lbl">detail</text></g>
      <line x1="184" y1="88" x2="250" y2="88" stroke="#5cc9e6" stroke-width="1.6" marker-end="url(#arr)"/>
      <g class="node store"><rect x="252" y="58" width="170" height="60" rx="11"/><text x="337" y="84" text-anchor="middle" font-size="13" fill="#63d39b">table</text><text x="337" y="103" text-anchor="middle" class="lbl">INSERT</text></g>
    </svg>
  </div>
</section>
```

## Data model — schema change

```html
<section class="section" id="data">
  <div class="sec-head"><span class="num"></span><h2>Data model</h2><span class="rule"></span></div>
  <div class="panel schema">
    <div class="tname">▦ table_name <span class="badge new">New table</span></div>
    <div class="col-row"><span class="cn pk">id</span><span class="ct">uuid · pk</span><span class="cd">default gen_random_uuid()</span></div>
    <div class="col-row"><span class="cn">user_id</span><span class="ct">uuid · fk</span><span class="cd">→ users.id · cascade</span></div>
    <div style="margin-top:16px;padding-top:14px;border-top:1px solid var(--line);font-family:var(--mono);font-size:12.5px;color:var(--muted)"><span style="color:var(--amber)">UNIQUE</span> (user_id, x) · <span style="color:var(--cyan)">INDEX</span> (user_id)</div>
  </div>
</section>
```
Use `<span class="badge mod">Modified</span>` for changes to an existing table.

## API surface — expandable endpoints

`onclick="epToggle(this)"` on each `.endpoint`. Verbs: `post green`, `get cyan`, `del red`, `put amber`.

```html
<section class="section" id="api">
  <div class="sec-head"><span class="num"></span><h2>API surface</h2><span class="rule"></span></div>
  <div class="panel" style="padding:16px">
    <div class="endpoint open" onclick="epToggle(this)">
      <div class="ep-head"><span class="verb post">POST</span><span class="ep-path">/api/thing</span><span class="ep-desc">create</span><span class="chev">›</span></div>
      <div class="ep-body"><pre><span class="cm"># body</span>
{ <span class="k">"field"</span>: <span class="v">"type"</span> }
<span class="cm"># 201 →</span> {...}  <span class="cm"># 401 / 404 / 409</span></pre></div>
    </div>
  </div>
</section>
```

## Files touched — file tree

```html
<section class="section" id="files">
  <div class="sec-head"><span class="num"></span><h2>Files touched</h2><span class="rule"></span></div>
  <div class="panel tree">
    <div class="row"><span class="dir">app/api/thing/</span></div>
    <div class="row"><span class="ind">└─</span><span class="fn">route.ts</span><span class="tag new">new</span></div>
    <div class="row"><span class="dir">app/components/</span></div>
    <div class="row"><span class="ind">└─</span><span class="fn">Thing.tsx</span><span class="tag mod">modified</span></div>
  </div>
</section>
```

## Annotated code — snippet with margin notes

`.hl` highlights a line; annotations `A`, `B`… reference line numbers.

```html
<section class="section" id="code">
  <div class="sec-head"><span class="num"></span><h2>Annotated code</h2><span class="rule"></span></div>
  <div class="code-wrap">
<pre class="code"><span class="ln">1</span><span class="kw">export function</span> <span class="fn">doThing</span>() {
<span class="hl"><span class="ln">2</span>  optimisticUpdate() <span class="cm">// before await</span></span>
<span class="ln">3</span>}</pre>
    <div class="annos">
      <div class="anno"><div class="tagn"><span class="b">A</span> line 2</div><p>Why this matters.</p></div>
    </div>
  </div>
</section>
```

## Open questions — decisions for the reviewer (recommended last section)

`onclick="pick(this)"`; mark the recommended option `sel` with a `★`.

```html
<section class="section" id="questions">
  <div class="sec-head"><span class="num"></span><h2>Open questions</h2><span class="rule"></span></div>
  <div class="q">
    <p class="qq">Q1 · The actual decision?</p>
    <div class="rec">recommended ↓</div>
    <div class="opts">
      <div class="opt sel" onclick="pick(this)"><span class="star">★</span>Recommended option</div>
      <div class="opt" onclick="pick(this)">Alternative</div>
    </div>
  </div>
</section>
```

## Plain note (use anywhere inside a `.panel`)

```html
<div class="panel"><p class="note">A short paragraph. <b>Bold</b> for emphasis.</p></div>
```
