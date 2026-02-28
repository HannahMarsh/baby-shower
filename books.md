---
layout: splash
title: "Books Instead of Cards"
permalink: /books/
---

# Bring your favorite book!

Instead of a card, please bring a children's book and write a message inside the cover :)

We are building Baby Olivia’s very first library. To help avoid duplicates, you can see which books have already been chosen below.

If you’d like to add the book you’re bringing, please submit it here:

<div class="form-wrapper">
  <iframe 
    data-tally-src="https://tally.so/embed/EkPbDL?alignLeft=1&hideTitle=1&transparentBackground=1&dynamicHeight=1"
    loading="lazy"
    width="100%"
    height="100"
    frameborder="0"
    marginheight="0"
    marginwidth="0"
    title="RSVP">
  </iframe>
</div>

<script>
  var d=document,w="https://tally.so/widgets/embed.js",v=function(){
    if(typeof Tally!=="undefined"){Tally.loadEmbeds();}
    else d.querySelectorAll("iframe[data-tally-src]").forEach(function(e){e.src=e.dataset.tallySrc;});
  };
  if(d.querySelector('script[src="'+w+'"]')===null){
    var s=d.createElement("script");
    s.src=w;
    s.onload=v;
    s.onerror=v;
    d.body.appendChild(s);
  } else { v(); }
</script>




# Olivia's Library

After submitting your book, refresh this page to see it appear below.



<div class="books-widget">
  <div class="books-controls">

    <!-- 
    <div class="search-wrapper">
        <svg class="search-icon" viewBox="0 0 24 24" aria-hidden="true">
            <path d="M10 2a8 8 0 105.293 14.293l4.707 4.707 1.414-1.414-4.707-4.707A8 8 0 0010 2zm0 2a6 6 0 110 12 6 6 0 010-12z"/>
        </svg>

        <input
            id="bookSearch"
            class="book-search"
            type="search"
            placeholder="Search by title or author…"
            autocomplete="off"
        />
    </div> 
    -->

    <div class="search-wrapper">
        <i class="fa-solid fa-magnifying-glass"></i>
        <input id="bookSearch" class="book-search" type="search" placeholder="Search by title or author…" autocomplete="off" />
    </div>
    <div class="books-meta" id="booksMeta"></div>
  </div>

  <div id="booksGrid" class="books-grid">
    <div class="books-loading">Loading book list…</div>
  </div>
</div>

<script>
  // ✅ 1) Paste your published CSV URL here:
  const BOOKS_CSV_URL = "https://docs.google.com/spreadsheets/d/e/2PACX-1vQLK5X0ZumxqktaZCKzaF8LfR_hC2_n6SfBYJIr9i-fbb2wNEY_tbtjy_GtTFH-uTNxi5ar7Fk2iV-y/pub?gid=0&single=true&output=csv";

  // ---- CSV parsing that handles quoted commas ----
  function parseCSV(text) {
    const rows = [];
    let row = [];
    let cur = "";
    let inQuotes = false;

    for (let i = 0; i < text.length; i++) {
      const c = text[i];
      const next = text[i + 1];

      if (c === '"' && inQuotes && next === '"') { // escaped quote
        cur += '"';
        i++;
      } else if (c === '"') {
        inQuotes = !inQuotes;
      } else if (c === ',' && !inQuotes) {
        row.push(cur.trim());
        cur = "";
      } else if ((c === '\n' || c === '\r') && !inQuotes) {
        if (c === '\r' && next === '\n') i++; // handle CRLF
        row.push(cur.trim());
        cur = "";
        // avoid pushing empty last line
        if (row.some(cell => cell.length > 0)) rows.push(row);
        row = [];
      } else {
        cur += c;
      }
    }
    // last cell
    row.push(cur.trim());
    if (row.some(cell => cell.length > 0)) rows.push(row);

    return rows;
  }

  function normalize(s) {
    return (s || "").toString().toLowerCase().trim();
  }

  function escapeHTML(s) {
    return (s ?? "").toString()
      .replaceAll("&", "&amp;")
      .replaceAll("<", "&lt;")
      .replaceAll(">", "&gt;")
      .replaceAll('"', "&quot;")
      .replaceAll("'", "&#39;");
  }

  async function loadBooks() {
    const grid = document.getElementById("booksGrid");
    const meta = document.getElementById("booksMeta");
    const search = document.getElementById("bookSearch");

    try {
      const res = await fetch(BOOKS_CSV_URL, { cache: "no-store" });
      if (!res.ok) throw new Error("Could not fetch CSV");
      const csvText = await res.text();

      const rows = parseCSV(csvText);
      if (rows.length < 2) {
        grid.innerHTML = `<div class="books-empty">No books yet — be the first to add one 💕</div>`;
        meta.textContent = "";
        return;
      }

      // Header row: find column indices by name (flexible)
      const header = rows[0].map(h => normalize(h));
      const idxTitle  = header.findIndex(h => h === "title" || h.includes("book title"));
      const idxAuthor = header.findIndex(h => h === "author" || h.includes("author"));
      const idxFormat = header.findIndex(h => h === "format" || h.includes("format"));

      const items = rows.slice(1)
        .map(r => ({
          title:  r[idxTitle]  ?? "",
          author: r[idxAuthor] ?? "",
          format: r[idxFormat] ?? ""
        }))
        .filter(x => x.title && x.title.trim().length > 0);

      // Sort by title (just in case the sheet isn't already sorted)
      items.sort((a,b) => normalize(a.title).localeCompare(normalize(b.title)));

      function render(filterText = "") {
        const q = normalize(filterText);

        const filtered = q
          ? items.filter(it =>
              normalize(it.title).includes(q) ||
              normalize(it.author).includes(q) ||
              normalize(it.format).includes(q)
            )
          : items;

        meta.textContent = `${filtered.length} book${filtered.length === 1 ? "" : "s"} listed`;

        if (filtered.length === 0) {
          grid.innerHTML = `<div class="books-empty">No matches — try a different search.</div>`;
          return;
        }

        grid.innerHTML = filtered.map(it => `
          <div class="book-card">
            <div class="book-title">${escapeHTML(it.title)}</div>
            ${it.author ? `<div class="book-author">${escapeHTML(it.author)}</div>` : ""}
            ${it.format ? `<div class="book-tag">${escapeHTML(it.format)}</div>` : ""}
          </div>
        `).join("");
      }

      render();

      search.addEventListener("input", (e) => render(e.target.value));

    } catch (err) {
      grid.innerHTML = `
        <div class="books-error">
          Couldn’t load the live book list right now.
          <br/>Please try again in a minute 💗
        </div>`;
      console.error(err);
    }
  }

  loadBooks();
</script>

