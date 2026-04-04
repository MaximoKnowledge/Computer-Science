```datacorejsx
return function View() {
  const allPapers = dc.useQuery('@page and #paper');

  // Filter state
  const [search, setSearch] = dc.useState("");
  const [stateFilter, setStateFilter] = dc.useState("all");
  const [categoryFilter, setCategoryFilter] = dc.useState("all");
  const [sortBy, setSortBy] = dc.useState("name");
  const [sortAsc, setSortAsc] = dc.useState(true);
  const [page, setPage] = dc.useState(0);
  const [qualityFilter, setQualityFilter] = dc.useState("all");
  const PAGE_SIZE = 15;
  
  const stateColors = {
    "read": "#4caf50",
    "to-read": "#2196f3",
    "reading": "#ff9800",
    "abandoned": "#9e9e9e",
  };
  
  const qualityColors = {
	"banger": "#e91e63",
	"good": "#4caf50",
	"meh": "#9e9e9e",
  };

  // Extract unique values for filter dropdowns
  const states = Object.keys(stateColors); 
  const qualities = Object.keys(qualityColors);
  const categories = [...new Set(allPapers.map(p => p.value("category")).filter(Boolean))];

  // Filter
  let papers = allPapers.filter(p => {
    const name = (p.value("name") || p.$name || "").toLowerCase();
    const tldr = (p.value("tldr") || "").toLowerCase();
    const q = search.toLowerCase();
    if (q && !name.includes(q) && !tldr.includes(q)) return false;
    if (stateFilter !== "all" && p.value("state") !== stateFilter) return false;
    if (categoryFilter !== "all" && p.value("category") !== categoryFilter) return false;
    if (qualityFilter !== "all" && String(p.value("quality") ?? "").trim() !== qualityFilter) return false;
    return true;
  });

  // Sort
  papers = [...papers].sort((a, b) => {
    const va = (a.value(sortBy) || a.$name || "").toString().toLowerCase();
    const vb = (b.value(sortBy) || b.$name || "").toString().toLowerCase();
    return sortAsc ? va.localeCompare(vb) : vb.localeCompare(va);
  });

  // Paginate
  const totalPages = Math.max(1, Math.ceil(papers.length / PAGE_SIZE));
  const safePage = Math.min(page, totalPages - 1);
  const visible = papers.slice(safePage * PAGE_SIZE, (safePage + 1) * PAGE_SIZE);

  // Sortable column header helper
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

  // Styles
  const pill = (color) => ({
    display: "inline-block",
    padding: "2px 8px",
    borderRadius: "12px",
    fontSize: "0.8em",
    backgroundColor: color,
    color: "white",
  });

  return (
    <div style={{ marginLeft: "-15%", marginRight: "-15%", padding: "0 8px" }}>
      {/* ── Toolbar ── */}
      <div style={{ display: "flex", gap: "8px", flexWrap: "wrap", marginBottom: "12px", alignItems: "center" }}>
        <input
          type="text"
          placeholder="Search name or TLDR..."
          value={search}
          onInput={e => { setSearch(e.target.value); setPage(0); }}
          style={{ flex: 1, minWidth: "180px", padding: "6px 10px", borderRadius: "6px", border: "1px solid var(--background-modifier-border)" }}
        />
        <select
          value={stateFilter}
          onChange={e => { setStateFilter(e.target.value); setPage(0); }}
          style={{ padding: "6px", borderRadius: "6px" }}
        >
          <option value="all">All states</option>
          {states.map(s => <option key={s} value={s}>{s}</option>)}
        </select>
        <select
          value={categoryFilter}
          onChange={e => { setCategoryFilter(e.target.value); setPage(0); }}
          style={{ padding: "6px", borderRadius: "6px" }}
        >
          <option value="all">All categories</option>
          {categories.map(c => <option key={c} value={c}>{c}</option>)}
        </select>
        <select
		  value={qualityFilter}
		  onChange={e => { setQualityFilter(e.target.value); setPage(0); }}
		  style={{ padding: "6px", borderRadius: "6px" }}
>		
		  <option value="all">All qualities</option>
		  {qualities.map(r => <option key={r} value={r}>{r}</option>)}
		</select>
		<span style={{ fontSize: "0.85em", opacity: 0.6 }}>
          {papers.length} paper{papers.length !== 1 ? "s" : ""}
        </span>
      </div>

      {/* ── Table ── */}
      <table style={{ width: "100%", borderCollapse: "collapse" }}>
        <thead>
          <tr style={{ borderBottom: "2px solid var(--background-modifier-border)" }}>
            <th style={{ textAlign: "left", padding: "6px" }}><SortHeader field="name" label="Paper" /></th>
            <th style={{ textAlign: "left", padding: "6px" }}>Link</th>
            <th style={{ textAlign: "left", padding: "6px" }}><SortHeader field="state" label="State" /></th>
            <th style={{ textAlign: "left", padding: "6px" }}><SortHeader field="quality" label="Quality" /></th>
            <th style={{ textAlign: "left", padding: "6px" }}><SortHeader field="category" label="Category" /></th>
            <th style={{ textAlign: "left", padding: "6px" }}>TLDR</th>
          </tr>
        </thead>
        <tbody>
          {visible.map(p => (
            <tr key={p.$path} style={{ borderBottom: "1px solid var(--background-modifier-border)" }}>
              <td style={{ padding: "6px" }}>
			  <a
			    className="internal-link"
			    href={p.$path}
			    data-href={p.$path}>							
			    {p.value("name") || p.$name || p.$path.split("/").pop().replace(".md", "")}
			    </a>
			  </td>
			  <td style={{ padding: "6px" }}>
				  {p.value("link") ? (
				<a href={p.value("link")} target="_blank" rel="noopener noreferrer" style={{ fontSize: "0.85em" }}>
				      arXiv ↗
				</a>
				  ) : "—"}
			  </td>
              <td style={{ padding: "6px" }}>
                <span style={pill(stateColors[p.value("state")] || "#666")}>
                  {p.value("state") || "—"}
                </span>
              </td>
              <td style={{ padding: "6px" }}>
				<span style={pill(qualityColors[p.value("quality")] || "#666")}>
					{p.value("quality") || "—"}
				</span>
			  </td>
              <td style={{ padding: "6px" }}>{p.value("category") || "—"}</td>
              <td style={{ padding: "6px", fontSize: "0.9em", opacity: 0.85 }}>
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

      {/* ── Pagination ── */}
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

