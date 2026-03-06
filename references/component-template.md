# Component Template

Wklej poniższy kod jako artefakt .jsx, podstawiając:
- `DATA_PLACEHOLDER` → wyekstrahowany JSON {nodes, links}
- `TITLE_PLACEHOLDER` → tytuł lub temat dokumentu (krótki string)

```jsx
import { useState, useEffect, useRef, useCallback } from "react";
import * as d3 from "d3";

const DATA = DATA_PLACEHOLDER;

const TITLE = "TITLE_PLACEHOLDER";

const TYPE_CONFIG = {
  teza:           { color: "#e2c97e", bg: "#2a2310", label: "Teza" },
  fakt:           { color: "#7eb8e2", bg: "#0f1e2a", label: "Fakt" },
  dane:           { color: "#7ee2b8", bg: "#0f2a1e", label: "Dane" },
  trend:          { color: "#a0e27e", bg: "#162a0f", label: "Trend" },
  wniosek_autora: { color: "#b87ee2", bg: "#1e0f2a", label: "Wniosek" },
  sprzeczność:    { color: "#e27e7e", bg: "#2a0f0f", label: "Sprzeczność" },
  luka:           { color: "#e2a07e", bg: "#2a1a0f", label: "Luka" },
};

const LINK_CONFIG = {
  wyjaśnia:   { color: "#7eb8e2", dash: "none" },
  poprzedza:  { color: "#7ee2b8", dash: "none" },
  wynika_z:   { color: "#b87ee2", dash: "none" },
  kontrastuje:{ color: "#e27e7e", dash: "6,3" },
  potwierdza: { color: "#7ee2b8", dash: "none" },
  podważa:    { color: "#e27e7e", dash: "4,2" },
};

const PEWNOSC_OPACITY = { wysoka: 1, średnia: 0.65, niska: 0.35 };

export default function GraphVisualizer() {
  const svgRef = useRef(null);
  const [selected, setSelected] = useState(null);
  const [filter, setFilter] = useState(null);

  const handleNodeClick = useCallback((node) => {
    setSelected(prev => prev?.id === node.id ? null : node);
  }, []);

  useEffect(() => {
    const el = svgRef.current;
    if (!el) return;
    const W = el.clientWidth || 800;
    const H = el.clientHeight || 600;
    d3.select(el).selectAll("*").remove();
    const svg = d3.select(el).attr("width", W).attr("height", H);

    const defs = svg.append("defs");
    Object.entries(LINK_CONFIG).forEach(([rel, cfg]) => {
      defs.append("marker")
        .attr("id", `arrow-${rel}`)
        .attr("viewBox", "0 -5 10 10")
        .attr("refX", 28).attr("refY", 0)
        .attr("markerWidth", 6).attr("markerHeight", 6)
        .attr("orient", "auto")
        .append("path").attr("d", "M0,-5L10,0L0,5")
        .attr("fill", cfg.color).attr("opacity", 0.8);
    });

    const g = svg.append("g");
    svg.call(d3.zoom().scaleExtent([0.25, 3]).on("zoom", e => g.attr("transform", e.transform)));

    const nodes = DATA.nodes.map(d => ({ ...d }));
    const links = DATA.links.map(d => ({ ...d }));

    const sim = d3.forceSimulation(nodes)
      .force("link", d3.forceLink(links).id(d => d.id).distance(140).strength(0.6))
      .force("charge", d3.forceManyBody().strength(-420))
      .force("center", d3.forceCenter(W / 2, H / 2))
      .force("collision", d3.forceCollide(58));

    const linkGroup = g.append("g");
    const linkEl = linkGroup.selectAll("line").data(links).enter().append("line")
      .attr("stroke", d => LINK_CONFIG[d.relacja]?.color || "#555")
      .attr("stroke-width", 1.5)
      .attr("stroke-opacity", 0.65)
      .attr("stroke-dasharray", d => LINK_CONFIG[d.relacja]?.dash || "none")
      .attr("marker-end", d => `url(#arrow-${d.relacja})`);

    const linkLabelEl = linkGroup.selectAll("text").data(links).enter().append("text")
      .attr("font-size", "9px").attr("font-family", "monospace")
      .attr("fill", d => LINK_CONFIG[d.relacja]?.color || "#888")
      .attr("opacity", 0.65).attr("text-anchor", "middle")
      .text(d => d.relacja);

    const nodeGroup = g.append("g");
    const nodeEl = nodeGroup.selectAll("g").data(nodes).enter().append("g")
      .attr("cursor", "pointer")
      .call(d3.drag()
        .on("start", (e, d) => { if (!e.active) sim.alphaTarget(0.3).restart(); d.fx = d.x; d.fy = d.y; })
        .on("drag", (e, d) => { d.fx = e.x; d.fy = e.y; })
        .on("end", (e, d) => { if (!e.active) sim.alphaTarget(0); d.fx = null; d.fy = null; })
      )
      .on("click", (e, d) => { e.stopPropagation(); handleNodeClick(d); });

    nodeEl.append("circle").attr("r", 36)
      .attr("fill", d => TYPE_CONFIG[d.typ]?.bg || "#111")
      .attr("stroke", d => TYPE_CONFIG[d.typ]?.color || "#555")
      .attr("stroke-width", d => d.pewność === "wysoka" ? 2 : 1)
      .attr("stroke-dasharray", d => d.pewność === "niska" ? "4,2" : "none")
      .attr("opacity", d => PEWNOSC_OPACITY[d.pewność] || 1);

    nodeEl.append("text").attr("y", -20).attr("text-anchor", "middle")
      .attr("font-size", "8px").attr("font-family", "monospace")
      .attr("letter-spacing", "0.05em")
      .attr("fill", d => TYPE_CONFIG[d.typ]?.color || "#aaa").attr("opacity", 0.9)
      .text(d => (TYPE_CONFIG[d.typ]?.label || d.typ).toUpperCase());

    nodeEl.append("text").attr("y", -8).attr("text-anchor", "middle")
      .attr("font-size", "11px").attr("font-weight", "700")
      .attr("font-family", "monospace")
      .attr("fill", d => TYPE_CONFIG[d.typ]?.color || "#fff")
      .text(d => d.id);

    nodeEl.each(function(d) {
      const words = d.label.split(" ");
      const lines = []; let line = [];
      words.forEach(w => {
        line.push(w);
        if (line.join(" ").length > 12) { lines.push(line.join(" ")); line = []; }
      });
      if (line.length) lines.push(line.join(" "));
      lines.slice(0, 2).forEach((ln, i) => {
        d3.select(this).append("text")
          .attr("y", 6 + i * 11).attr("text-anchor", "middle")
          .attr("font-size", "9px").attr("font-family", "sans-serif")
          .attr("fill", "#ccc").text(ln);
      });
    });

    svg.on("click", () => setSelected(null));

    sim.on("tick", () => {
      linkEl.attr("x1", d => d.source.x).attr("y1", d => d.source.y)
        .attr("x2", d => d.target.x).attr("y2", d => d.target.y);
      linkLabelEl
        .attr("x", d => (d.source.x + d.target.x) / 2)
        .attr("y", d => (d.source.y + d.target.y) / 2 - 4);
      nodeEl.attr("transform", d => `translate(${d.x},${d.y})`);
    });

    return () => sim.stop();
  }, [handleNodeClick]);

  useEffect(() => {
    if (!svgRef.current) return;
    const svg = d3.select(svgRef.current);
    svg.selectAll("g g").attr("opacity", d => {
      if (!filter) return 1;
      return d.typ === filter ? 1 : 0.1;
    });
  }, [filter]);

  const typeCounts = DATA.nodes.reduce((acc, n) => { acc[n.typ] = (acc[n.typ] || 0) + 1; return acc; }, {});

  return (
    <div style={{ background: "#0a0a0f", minHeight: "100vh", color: "#ccc", fontFamily: "monospace", display: "flex", flexDirection: "column" }}>
      <div style={{ padding: "16px 24px", borderBottom: "1px solid #1e1e2e", display: "flex", alignItems: "center", justifyContent: "space-between", background: "#0d0d18" }}>
        <div>
          <div style={{ fontSize: "10px", color: "#444", letterSpacing: "0.2em", marginBottom: 4 }}>STRUCTURE OF THOUGHT · ANALIZA</div>
          <div style={{ fontSize: "17px", color: "#e2c97e", fontWeight: 700, letterSpacing: "0.04em" }}>{TITLE}</div>
        </div>
        <div style={{ fontSize: "10px", color: "#444", textAlign: "right" }}>
          <div>{DATA.nodes.length} węzłów · {DATA.links.length} relacji</div>
          <div style={{ marginTop: 4, color: "#2a2a3a" }}>drag · scroll · click</div>
        </div>
      </div>

      <div style={{ display: "flex", flex: 1, overflow: "hidden" }}>
        <div style={{ flex: 1, position: "relative" }}>
          <svg ref={svgRef} style={{ width: "100%", height: "calc(100vh - 61px)" }} />
        </div>

        <div style={{ width: 256, borderLeft: "1px solid #1e1e2e", background: "#0d0d18", display: "flex", flexDirection: "column", overflow: "hidden" }}>
          <div style={{ padding: "14px 16px", borderBottom: "1px solid #161624" }}>
            <div style={{ fontSize: "9px", color: "#3a3a5a", letterSpacing: "0.15em", marginBottom: 10 }}>TYPY · filtruj</div>
            {Object.entries(typeCounts).map(([typ, count]) => {
              const cfg = TYPE_CONFIG[typ] || { color: "#777", label: typ };
              const active = filter === typ;
              return (
                <div key={typ} onClick={() => setFilter(active ? null : typ)} style={{ display: "flex", alignItems: "center", gap: 8, padding: "5px 8px", marginBottom: 2, borderRadius: 4, cursor: "pointer", background: active ? cfg.bg : "transparent", border: active ? `1px solid ${cfg.color}` : "1px solid transparent", transition: "all 0.15s" }}>
                  <div style={{ width: 8, height: 8, borderRadius: "50%", background: cfg.color, flexShrink: 0 }} />
                  <span style={{ fontSize: "10px", color: active ? cfg.color : "#555", flex: 1 }}>{cfg.label}</span>
                  <span style={{ fontSize: "10px", color: "#2a2a3a" }}>{count}</span>
                </div>
              );
            })}
          </div>

          <div style={{ padding: "12px 16px", borderBottom: "1px solid #161624" }}>
            <div style={{ fontSize: "9px", color: "#3a3a5a", letterSpacing: "0.15em", marginBottom: 8 }}>RELACJE</div>
            {Object.entries(LINK_CONFIG).map(([rel, cfg]) => (
              <div key={rel} style={{ display: "flex", alignItems: "center", gap: 8, marginBottom: 4 }}>
                <div style={{ width: 18, height: 0, flexShrink: 0, borderTop: cfg.dash !== "none" ? `1.5px dashed ${cfg.color}` : `1.5px solid ${cfg.color}`, opacity: 0.8 }} />
                <span style={{ fontSize: "9px", color: "#444" }}>{rel}</span>
              </div>
            ))}
          </div>

          <div style={{ flex: 1, padding: "14px 16px", overflowY: "auto" }}>
            {selected ? (
              <>
                <div style={{ fontSize: "9px", color: "#3a3a5a", letterSpacing: "0.15em", marginBottom: 8 }}>WĘZEŁ</div>
                <div style={{ background: TYPE_CONFIG[selected.typ]?.bg || "#111", border: `1px solid ${TYPE_CONFIG[selected.typ]?.color || "#333"}`, borderRadius: 6, padding: 12, marginBottom: 10 }}>
                  <div style={{ fontSize: "9px", color: TYPE_CONFIG[selected.typ]?.color, marginBottom: 4 }}>
                    {selected.id} · {(TYPE_CONFIG[selected.typ]?.label || selected.typ).toUpperCase()}
                  </div>
                  <div style={{ fontSize: "12px", color: "#ddd", fontWeight: 600, lineHeight: 1.4, marginBottom: 8 }}>{selected.label}</div>
                  <div style={{ fontSize: "10px", color: "#666", lineHeight: 1.6, borderLeft: `2px solid ${TYPE_CONFIG[selected.typ]?.color || "#333"}`, paddingLeft: 8, fontFamily: "sans-serif", fontStyle: "italic" }}>
                    {selected.cytat}
                  </div>
                </div>
                <div style={{ display: "flex", gap: 6 }}>
                  <div style={{ flex: 1, background: "#111", border: "1px solid #1a1a28", borderRadius: 4, padding: "6px 8px", fontSize: "9px", color: "#444", textAlign: "center" }}>
                    pewność<br />
                    <span style={{ color: selected.pewność === "wysoka" ? "#7ee2b8" : selected.pewność === "średnia" ? "#e2c97e" : "#e27e7e", fontWeight: 700, fontSize: "11px" }}>
                      {selected.pewność}
                    </span>
                  </div>
                  <div style={{ flex: 1, background: "#111", border: "1px solid #1a1a28", borderRadius: 4, padding: "6px 8px", fontSize: "9px", color: "#444", textAlign: "center" }}>
                    relacje<br />
                    <span style={{ color: "#7eb8e2", fontWeight: 700, fontSize: "11px" }}>
                      {DATA.links.filter(l => l.source === selected.id || l.target === selected.id).length}
                    </span>
                  </div>
                </div>
              </>
            ) : (
              <div style={{ fontSize: "10px", color: "#2a2a3a", lineHeight: 2 }}>
                Kliknij węzeł<br />aby zobaczyć<br />cytat źródłowy.
              </div>
            )}
          </div>
        </div>
      </div>
    </div>
  );
}
```
