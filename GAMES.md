# PixelPlayground: HTML5 Games Integration Template

This template shows how to integrate many HTML5 games into PixelPlayground using links, embeds, and iframes. It includes example code, reputable open-source sources, and a roadmap for scaling to thousands of games and community submissions.

---

## Quick Start

Choose one of these approaches:
- Link to external game pages (fastest, lowest maintenance)
- Embed open-source games directly via iframe
- Host copies of permissively licensed games in this repository (respect licenses)
- Dynamically load games from a JSON catalog or external API

---

## 1) Link-only Catalog (simple and scalable)

Create a JSON file of game metadata and render it into a gallery.

Example: games.json
```json
[
  {"title": "2048 (open-source)", "url": "https://play2048.co/", "repo": "https://github.com/gabrielecirulli/2048", "license": "MIT"},
  {"title": "Hextris", "url": "https://hextris.io/", "repo": "https://github.com/Hextris/hextris", "license": "MIT"},
  {"title": "Clumsy Bird", "url": "https://ellisonleao.github.io/clumsy-bird/", "repo": "https://github.com/ellisonleao/clumsy-bird", "license": "MIT"}
]
```

Example gallery (games.html snippet):
```html
<section id="gallery" class="games-grid"></section>
<script>
async function loadGames() {
  const res = await fetch('games.json');
  const games = await res.json();
  const el = document.getElementById('gallery');
  el.innerHTML = games.map(g => `
    <article class="game-card">
      <h3>${g.title}</h3>
      <p><a href="${g.url}" target="_blank" rel="noopener noreferrer">Play</a></p>
      ${g.repo ? `<p><a href="${g.repo}" target="_blank" rel="noopener">Source</a></p>` : ''}
      ${g.license ? `<p>License: ${g.license}</p>` : ''}
    </article>
  `).join('');
}
loadGames();
</script>
```

Pros: very scalable, minimal cross-origin issues. Cons: navigates away to play.

---

## 2) Embed via Iframe (play in-page)

Use iframes to embed games hosted elsewhere. Always verify the host allows embedding and set appropriate sandboxing.

Example iframe card:
```html
<article class="game-embed">
  <h3>2048 (MIT)</h3>
  <iframe
    src="https://play2048.co/"
    loading="lazy"
    referrerpolicy="no-referrer"
    sandbox="allow-scripts allow-same-origin allow-pointer-lock allow-popups"
    width="480" height="640" style="border:0;">
  </iframe>
  <p><a href="https://github.com/gabrielecirulli/2048" target="_blank" rel="noopener">Source</a></p>
</article>
```

Security notes:
- Prefer sandbox on iframes; add permissions incrementally
- Use rel=noopener noreferrer on external links
- Only embed reputable, trusted domains

---

## 3) Host Open-source Games Here (licensed)

You can vendor game code into subfolders, respecting their licenses.

Structure:
```
/games/
  2048/          # fork or vendored copy (MIT)
  hextris/       # fork or vendored copy (MIT)
  clumsy-bird/   # MIT
index.html       # router/gallery
```

Embed locally hosted games:
```html
<iframe src="./games/2048/index.html" width="480" height="640" sandbox="allow-scripts allow-same-origin"></iframe>
```

Make sure to:
- Keep LICENSE and NOTICE files
- Attribute original authors in README
- Respect trademarks and asset licenses

---

## 4) Dynamic Catalog From URLs or APIs

You can scale to thousands of games by loading metadata from:
- A static JSON catalog hosted on GitHub Pages or a CDN
- Public APIs that index open-source web games
- Community-submitted URLs stored in a database or JSON

Example using a static JSON of many items and virtualized list:
```html
<script type="module">
import { virtualize } from 'https://cdn.skypack.dev/virtual-list@3';
const container = document.getElementById('gallery');
const res = await fetch('https://example.com/huge-games.json');
const games = await res.json();
const row = (i) => {
  const g = games[i];
  return `<div class="row">
    <span>${g.title}</span>
    <a href="${g.url}" target="_blank" rel="noopener">Play</a>
  </div>`;
};
container.innerHTML = '';
virtualize({ container, total: games.length, itemHeight: 56, renderItem: row });
</script>
```

Pagination and search example:
```html
<input id="q" placeholder="Search games..."/>
<div id="gallery"></div>
<script>
let all=[]; let page=0; const pageSize=50;
async function load() {
  const res = await fetch('games.json');
  all = await res.json();
  render();
}
function render() {
  const q = document.getElementById('q').value.toLowerCase();
  const filtered = all.filter(g => g.title.toLowerCase().includes(q));
  const slice = filtered.slice(0, (page+1)*pageSize);
  document.getElementById('gallery').innerHTML = slice.map(g => `
    <div class="game"><strong>${g.title}</strong> — <a href="${g.url}" target="_blank" rel="noopener">Play</a></div>
  `).join('');
}
q.addEventListener('input', () => { page=0; render(); });
window.addEventListener('scroll', () => {
  if (window.innerHeight + window.scrollY >= document.body.offsetHeight - 200) {
    page++; render();
  }
});
load();
</script>
```

---

## 5) Example Reputable Open-source HTML5 Games

- 2048 — MIT — https://github.com/gabrielecirulli/2048 — https://play2048.co/
- Hextris — MIT — https://github.com/Hextris/hextris — https://hextris.io/
- Clumsy Bird — MIT — https://github.com/ellisonleao/clumsy-bird — https://ellisonleao.github.io/clumsy-bird/
- Browser NES — MIT — https://github.com/bfirsh/jsnes — demo: https://jsnes.org/
- Emularity/Emularity-based Arcade — GPL/MIT mix — https://github.com/db48x/emularity

Always review each repo’s LICENSE before embedding or hosting.

---

## 6) games.html Boilerplate (drop-in page)

Create a games.html page for your site:
```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>PixelPlayground — Games</title>
  <style>
    body { font-family: system-ui, sans-serif; margin: 0; padding: 1rem; }
    .toolbar { display:flex; gap:.5rem; align-items:center; flex-wrap:wrap; }
    .games-grid { display:grid; grid-template-columns: repeat(auto-fill, minmax(240px, 1fr)); gap: 12px; margin-top:1rem; }
    .game-card, .game-embed { border:1px solid #ddd; border-radius:8px; padding:12px; background:#fff; }
    iframe { width:100%; aspect-ratio: 3 / 4; background:#000; }
  </style>
</head>
<body>
  <header class="toolbar">
    <h1 style="margin:0;">PixelPlayground</h1>
    <input id="q" placeholder="Search games..." style="flex:1; min-width:200px;" />
    <button id="mode-link">Link Mode</button>
    <button id="mode-embed">Embed Mode</button>
  </header>
  <section id="gallery" class="games-grid"></section>

  <script>
  const catalogUrl = 'games.json'; // or a remote JSON URL
  let games = [];
  let mode = 'link';

  async function boot() {
    try {
      const res = await fetch(catalogUrl);
      games = await res.json();
      render();
    } catch (e) {
      console.error('Failed to load catalog', e);
      document.getElementById('gallery').innerHTML = '<p>Failed to load catalog.</p>';
    }
  }

  function render() {
    const q = document.getElementById('q').value?.toLowerCase() || '';
    const filtered = games.filter(g => !q || g.title.toLowerCase().includes(q));
    const html = filtered.map(g => mode === 'embed' && g.embedUrl ? `
      <article class="game-embed">
        <h3>${g.title}</h3>
        <iframe src="${g.embedUrl}" loading="lazy" sandbox="allow-scripts allow-same-origin allow-pointer-lock allow-popups" referrerpolicy="no-referrer"></iframe>
        ${g.repo ? `<p><a href="${g.repo}" target="_blank" rel="noopener">Source</a></p>` : ''}
      </article>
    ` : `
      <article class="game-card">
        <h3>${g.title}</h3>
        <p><a href="${g.url}" target="_blank" rel="noopener">Play</a></p>
        ${g.repo ? `<p><a href="${g.repo}" target="_blank" rel="noopener">Source</a></p>` : ''}
      </article>
    `).join('');
    document.getElementById('gallery').innerHTML = html || '<p>No games found.</p>';
  }

  document.getElementById('q').addEventListener('input', render);
  document.getElementById('mode-link').addEventListener('click', () => { mode='link'; render(); });
  document.getElementById('mode-embed').addEventListener('click', () => { mode='embed'; render(); });

  boot();
  </script>
</body>
</html>
```

Add games.json entries with both url (for link mode) and embedUrl (if the host allows embedding):
```json
[
  { "title": "2048", "url": "https://play2048.co/", "embedUrl": "https://play2048.co/", "repo": "https://github.com/gabrielecirulli/2048", "license": "MIT" }
]
```

---

## 7) Roadmap: Scaling and Community Submissions

- Catalog at scale
  - Store metadata in games.json or split into paged files games-1.json, games-2.json, ...
  - Add fields: id, title, url, embedUrl, repo, license, tags, author, addedAt
  - Implement search, tags, pagination, virtualization, and caching
- Submission workflow
  - Add a CONTRIBUTING.md with a PR template for new games
  - Validate submissions via GitHub Actions (JSON schema, URL checks, embedding policy)
  - Auto-generate a README section from the catalog
- API integration
  - Create a small serverless function or worker that proxies/validates external catalogs
  - Consider larger indexes or APIs that list open-source HTML5 games
- Moderation and safety
  - Only allow HTTPS, trusted domains, and clean licensing
  - Enforce iframe sandbox and CSP on hosted pages
- Performance
  - Lazy-load embeds, images, and metadata; pre-render link cards
  - Use pagination or infinite scroll for thousands of items

---

## 8) License and Attribution

- Respect each game’s license; keep LICENSE files and notices when hosting
- Attribute authors in the catalog and UI
- Provide a way to remove content upon request if licensing changes

---

## 9) Maintenance Checklist

- [ ] Keep games.json updated
- [ ] Periodically recheck URLs and embedding headers (X-Frame-Options/CSP)
- [ ] Update dependencies and sandbox permissions as needed
- [ ] Monitor performance with large catalogs

---

Happy hacking! Contributions welcome via PRs adding entries to games.json or improving games.html.
