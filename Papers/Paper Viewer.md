```datacorejsx
return function View() {
  const allPapers = dc.useQuery('@page and #paper');

  const [search, setSearch] = dc.useState("");
  const [stateFilter, setStateFilter] = dc.useState("all");
  const [tagsFilter, setTagsFilter] = dc.useState("all");
  const [sortBy, setSortBy] = dc.useState("title");
  const [sortAsc, setSortAsc] = dc.useState(true);
  const [page, setPage] = dc.useState(0);
  const [qualityFilter, setQualityFilter] = dc.useState("all");
  const PAGE_SIZE = 15;

  const stateColors = {
    "read": "#4caf50",
    "skimmed": "#2acaea",
    "to-read": "#2196f3",
    "reading": "#ff9800",
    "abandoned": "#9e9e9e",
  };

  const qualityColors = {
    "banger": "#e91e63",
    "good": "#4caf50",
    "meh": "#9e9e9e",
  };

  const tagColorCache = {};
  const tagColor = (tag) => {
    if (tagColorCache[tag]) return tagColorCache[tag];
    let h = 0;
    for (let i = 0; i < tag.length; i++) h = tag.charCodeAt(i) + ((h << 5) - h);
    const hue = ((h % 360) + 360) % 360;
    tagColorCache[tag] = `hsl(${hue}, 55%, 45%)`;
    return tagColorCache[tag];
  };

  const states = Object.keys(stateColors);
  const qualities = Object.keys(qualityColors);

  const categories = [...new Set(
    allPapers.flatMap(p => {
      const t = p.value("tags");
      if (Array.isArray(t)) return t;
      if (t) return [t];
      return [];
    })
  )].sort();

  const getTags = (p) => {
    const t = p.value("tags");
    if (Array.isArray(t)) return [...t].sort();
    if (t) return [t];
    return [];
  };

  let papers = allPapers.filter(p => {
    const title = (p.value("title") || p.$title || "").toLowerCase();
    const tldr = (p.value("tldr") || "").toLowerCase();
    const q = search.toLowerCase();
    if (q && !title.includes(q) && !tldr.includes(q)) return false;
    if (stateFilter !== "all" && p.value("state") !== stateFilter) return false;
    if (tagsFilter !== "all" && !getTags(p).includes(tagsFilter)) return false;
    if (qualityFilter !== "all" && String(p.value("quality") ?? "").trim() !== qualityFilter) return false;
    return true;
  });

  papers = [...papers].sort((a, b) => {
	  let va, vb;
	  if (sortBy === "title") {
	    va = (a.value("title") || a.$title || a.$path.split("/").pop().replace(".md", "")).toLowerCase();
	    vb = (b.value("title") || b.$title || b.$path.split("/").pop().replace(".md", "")).toLowerCase();
	  } else {
	    va = (a.value(sortBy) || "").toString().toLowerCase();
	    vb = (b.value(sortBy) || "").toString().toLowerCase();
	  }
	  return sortAsc ? va.localeCompare(vb) : vb.localeCompare(va);
	});

  const totalPages = Math.max(1, Math.ceil(papers.length / PAGE_SIZE));
  const safePage = Math.min(page, totalPages - 1);
  const visible = papers.slice(safePage * PAGE_SIZE, (safePage + 1) * PAGE_SIZE);

  const SortHeader = ({ field, label }) => {
    const active = sortBy === field;
    return (
      <span
        onClick={() => {
          if (active) setSortAsc(!sortAsc);
          else { setSortBy(field); setSortAsc(true); }
        }}
        style={{ cursor: "pointer", fontWeight: active ? "bold" : "normal" }}
      >
        {label} {active ? (sortAsc ? "▲" : "▼") : ""}
      </span>
    );
  };

  const pill = (color) => ({
    display: "inline-block",
    padding: "2px 8px",
    borderRadius: "12px",
    fontSize: "0.8em",
    backgroundColor: color,
    color: "white",
  });

  const colWidths = {
    paper: "22%",
    link: "7%",
    state: "9%",
    quality: "9%",
    tags: "14%",
    tldr: "39%",
  };

  return (
    <div style={{ marginLeft: "-15%", marginRight: "-15%", padding: "0 8px" }}>
      <div style={{ display: "flex", gap: "8px", flexWrap: "wrap", marginBottom: "12px", alignItems: "center" }}>
        <input
          type="text"
          placeholder="Search title or TLDR..."
          value={search}
          onInput={e => { setSearch(e.target.value); setPage(0); }}
          style={{ flex: 1, minWidth: "180px", padding: "6px 10px", borderRadius: "6px", border: "1px solid var(--background-modifier-border)" }}
        />
        <select value={stateFilter} onChange={e => { setStateFilter(e.target.value); setPage(0); }} style={{ padding: "6px", borderRadius: "6px" }}>
          <option value="all">All states</option>
          {states.map(s => <option key={s} value={s}>{s}</option>)}
        </select>
        <select value={tagsFilter} onChange={e => { setTagsFilter(e.target.value); setPage(0); }} style={{ padding: "6px", borderRadius: "6px" }}>
          <option value="all">All categories</option>
          {categories.map(c => <option key={c} value={c}>{c}</option>)}
        </select>
        <select value={qualityFilter} onChange={e => { setQualityFilter(e.target.value); setPage(0); }} style={{ padding: "6px", borderRadius: "6px" }}>
          <option value="all">All qualities</option>
          {qualities.map(r => <option key={r} value={r}>{r}</option>)}
        </select>
        <span style={{ fontSize: "0.85em", opacity: 0.6 }}>
          {papers.length} paper{papers.length !== 1 ? "s" : ""}
        </span>
      </div>

      <table style={{ width: "100%", borderCollapse: "collapse", tableLayout: "fixed" }}>
        <thead>
          <tr style={{ borderBottom: "2px solid var(--background-modifier-border)" }}>
            <th style={{ textAlign: "left", padding: "6px", width: colWidths.paper }}><SortHeader field="title" label="Paper" /></th>
            <th style={{ textAlign: "left", padding: "6px", width: colWidths.link }}>Link</th>
            <th style={{ textAlign: "left", padding: "6px", width: colWidths.state }}><SortHeader field="state" label="State" /></th>
            <th style={{ textAlign: "left", padding: "6px", width: colWidths.quality }}><SortHeader field="quality" label="Quality" /></th>
            <th style={{ textAlign: "left", padding: "6px", width: colWidths.tags }}><SortHeader field="tags" label="Tags" /></th>
            <th style={{ textAlign: "left", padding: "6px", width: colWidths.tldr }}>TLDR</th>
          </tr>
        </thead>
        <tbody>
          {visible.map(p => (
            <tr key={p.$path} style={{ borderBottom: "1px solid var(--background-modifier-border)" }}>
              <td style={{ padding: "6px", wordBreak: "break-word" }}>
                <a className="internal-link" href={p.$path} data-href={p.$path}>
                  {p.value("title") || p.$title || p.$path.split("/").pop().replace(".md", "")}
                </a>
              </td>
              <td style={{ padding: "6px" }}>
                {p.value("link") ? (
                  <a href={p.value("link")} target="_blank" rel="noopener noreferrer" style={{ fontSize: "0.85em" }}>arXiv ↗</a>
                ) : "—"}
              </td>
              <td style={{ padding: "6px" }}>
                <span style={pill(stateColors[p.value("state")] || "#666")}>{p.value("state") || "—"}</span>
              </td>
              <td style={{ padding: "6px" }}>
                <span style={pill(qualityColors[p.value("quality")] || "#666")}>{p.value("quality") || "—"}</span>
              </td>
              <td style={{ padding: "6px", wordBreak: "break-word" }}>
                {getTags(p).length > 0 ? (
                  <div style={{ display: "flex", flexWrap: "wrap", gap: "4px" }}>
                    {getTags(p).map(tag => (
                      <span key={tag} style={{
                        display: "inline-block",
                        padding: "2px 8px",
                        borderRadius: "12px",
                        fontSize: "0.75em",
                        backgroundColor: tagColor(tag),
                        color: "white",
                        whiteSpace: "nowrap",
                      }}>{tag}</span>
                    ))}
                  </div>
                ) : "—"}
              </td>
              <td style={{ padding: "6px", fontSize: "0.9em", opacity: 0.85, wordBreak: "break-word" }}>
                {p.value("tldr") || "—"}
              </td>
            </tr>
          ))}
          {visible.length === 0 && (
            <tr><td colSpan={6} style={{ padding: "20px", textAlign: "center", opacity: 0.5 }}>
              No papers match your filters
            </td></tr>
          )}
        </tbody>
      </table>

      {totalPages > 1 && (
        <div style={{ display: "flex", justifyContent: "center", gap: "8px", marginTop: "12px", alignItems: "center" }}>
          <button onClick={() => setPage(Math.max(0, safePage - 1))} disabled={safePage === 0}>←</button>
          <span style={{ fontSize: "0.85em" }}>{safePage + 1} / {totalPages}</span>
          <button onClick={() => setPage(Math.min(totalPages - 1, safePage + 1))} disabled={safePage >= totalPages - 1}>→</button>
        </div>
      )}
    </div>
  );
}
```

