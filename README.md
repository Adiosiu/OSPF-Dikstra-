<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Simulasi Dijkstra dengan Graf Visual</title>
  <script src="https://unpkg.com/cytoscape@3.21.1/dist/cytoscape.min.js"></script>
  <style>
    body { font-family: sans-serif; }
    select, button { margin: 5px; padding: 5px; }
    #output { margin-top: 15px; white-space: pre-wrap; }
    #cy { width: 100%; height: 600px; border: 1px solid #ccc; margin-top: 20px; }
  </style>
</head>
<body>
  <h2>Simulasi Jalur Tercepat (Dijkstra) - Visual Graf</h2>
  <label>Dari:
    <select id="from"></select>
  </label>
  <label>Ke:
    <select id="to"></select>
  </label>
  <button onclick="runDijkstra()">Cari Jalur Tercepat</button>
  <button onclick="runViaSwitch()">Jalur Biasa</button>
  <div id="output"></div>
  <div id="cy"></div>

  <script>
    const edges = [
      { s: 'L1-R1', t: 'L1-R2', c: 3 }, { s: 'L1-R1', t: 'SW-L1', c: 5 },
      { s: 'L1-R2', t: 'L1-R3', c: 5 }, { s: 'L1-R2', t: 'SW-L1', c: 8 },
      { s: 'L1-R3', t: 'L1-R4', c: 7 }, { s: 'L1-R3', t: 'SW-L1', c: 2 },
      { s: 'L1-R4', t: 'SW-L1', c: 3 },
      { s: 'L2-R1', t: 'L2-R2', c: 3 }, { s: 'L2-R1', t: 'SW-L2', c: 1 },
      { s: 'L2-R2', t: 'L2-R3', c: 5 }, { s: 'L2-R2', t: 'SW-L2', c: 5 },
      { s: 'L2-R3', t: 'L2-R4', c: 8 }, { s: 'L2-R3', t: 'SW-L2', c: 3 },
      { s: 'L2-R4', t: 'SW-L2', c: 2 },
      { s: 'L3-R1', t: 'L3-R2', c: 4 }, { s: 'L3-R1', t: 'SW-L3', c: 7 },
      { s: 'L3-R2', t: 'L3-R3', c: 4 }, { s: 'L3-R2', t: 'SW-L3', c: 2 },
      { s: 'L3-R3', t: 'L3-R4', c: 7 }, { s: 'L3-R3', t: 'SW-L3', c: 7 },
      { s: 'L3-R4', t: 'SW-L3', c: 6 },
      { s: 'SW-L1', t: 'SW-L2', c: 8 }, { s: 'SW-L2', t: 'SW-L3', c: 8 }
    ];

    const nodes = new Set();
    edges.forEach(e => { nodes.add(e.s); nodes.add(e.t); });

    const fromSelect = document.getElementById("from");
    const toSelect = document.getElementById("to");
    nodes.forEach(n => {
      if (!n.startsWith("SW-")) {
        const opt1 = new Option(n, n);
        const opt2 = new Option(n, n);
        fromSelect.add(opt1);
        toSelect.add(opt2);
      }
    });

    const positions = {
      'SW-L1': { x: 200, y: 100 },
      'L1-R1': { x: 100, y: 180 }, 'L1-R2': { x: 180, y: 180 }, 'L1-R3': { x: 260, y: 180 }, 'L1-R4': { x: 340, y: 180 },
      'SW-L2': { x: 600, y: 100 },
      'L2-R1': { x: 500, y: 180 }, 'L2-R2': { x: 580, y: 180 }, 'L2-R3': { x: 660, y: 180 }, 'L2-R4': { x: 740, y: 180 },
      'SW-L3': { x: 1000, y: 100 },
      'L3-R1': { x: 900, y: 180 }, 'L3-R2': { x: 980, y: 180 }, 'L3-R3': { x: 1060, y: 180 }, 'L3-R4': { x: 1140, y: 180 }
    };

    const cy = cytoscape({
      container: document.getElementById('cy'),
      elements: [
        ...[...nodes].map(n => ({ data: { id: n }, position: positions[n] })),
        ...edges.map(e => ({ data: { id: e.s + e.t, source: e.s, target: e.t, label: e.c.toString() } }))
      ],
      style: [
        {
          selector: 'node',
          style: {
            'label': 'data(id)',
            'background-color': '#007bff',
            'color': '#fff',
            'text-valign': 'center',
            'text-outline-color': '#007bff',
            'text-outline-width': 2
          }
        },
        {
          selector: 'edge',
          style: {
            'width': 2,
            'label': 'data(label)',
            'line-color': '#ccc',
            'target-arrow-shape': 'triangle',
            'target-arrow-color': '#ccc',
            'curve-style': 'bezier'
          }
        },
        {
          selector: '.highlight',
          style: {
            'background-color': 'orange',
            'line-color': 'orange',
            'target-arrow-color': 'orange'
          }
        }
      ],
      layout: { name: 'preset' },
      userPanningEnabled: false,
      userZoomingEnabled: false,
      boxSelectionEnabled: false
    });

    function runDijkstra() {
      cy.elements().removeClass('highlight');
      const start = fromSelect.value;
      const end = toSelect.value;
      const result = dijkstra(edges, start, end);
      document.getElementById("output").textContent = result.path.length ? `Jalur: ${result.path.join(" → ")}\nTotal cost: ${result.cost}` : "Jalur tidak ditemukan.";
      highlightPath(result.path);
    }

    function dijkstra(edgeList, start, end) {
      const graph = {};
      edgeList.forEach(({ s, t, c }) => {
        if (!graph[s]) graph[s] = {};
        if (!graph[t]) graph[t] = {};
        graph[s][t] = c;
        graph[t][s] = c;
      });

      const dist = {}, prev = {}, visited = new Set();
      Object.keys(graph).forEach(n => { dist[n] = Infinity; prev[n] = null; });
      dist[start] = 0;

      while (visited.size < Object.keys(graph).length) {
        let u = Object.keys(dist).filter(n => !visited.has(n)).sort((a, b) => dist[a] - dist[b])[0];
        if (!u || dist[u] === Infinity) break;
        visited.add(u);
        for (let v in graph[u]) {
          let alt = dist[u] + graph[u][v];
          if (alt < dist[v]) {
            dist[v] = alt;
            prev[v] = u;
          }
        }
      }

      const path = [];
      let u = end;
      while (u) {
        path.unshift(u);
        u = prev[u];
      }

      return {
        path: path[0] === start ? path : [],
        cost: dist[end]
      };
    }

    function runViaSwitch() {
  cy.elements().removeClass('highlight');
  const start = fromSelect.value;
  const end = toSelect.value;

  function getSwitch(room) {
    return room.startsWith("L1") ? "SW-L1" : room.startsWith("L2") ? "SW-L2" : "SW-L3";
  }

  const swFrom = getSwitch(start);
  const swTo = getSwitch(end);

  let path = [start, swFrom];
  if (swFrom !== swTo) {
    const switchLevels = ["SW-L1", "SW-L2", "SW-L3"];
    const i1 = switchLevels.indexOf(swFrom);
    const i2 = switchLevels.indexOf(swTo);
    if (i1 < i2) {
      for (let i = i1 + 1; i <= i2; i++) path.push(switchLevels[i]);
    } else {
      for (let i = i1 - 1; i >= i2; i--) path.push(switchLevels[i]);
    }
  }
  path.push(end);

  // Hitung total cost dari path biasa
  let totalCost = 0;
  for (let i = 0; i < path.length - 1; i++) {
    const a = path[i];
    const b = path[i + 1];
    const edge = edges.find(e => (e.s === a && e.t === b) || (e.s === b && e.t === a));
    if (edge) {
      totalCost += edge.c;
    }
  }

  document.getElementById("output").textContent =
    `Jalur Biasa: ${path.join(" → ")}\nTotal cost: ${totalCost}`;
  highlightPath(path);
}

 

    function highlightPath(path) {
      path.forEach((node, i) => {
        if (i < path.length - 1) {
          const next = path[i + 1];
          const edge = cy.getElementById(node + next).length ? node + next : next + node;
          cy.getElementById(edge).addClass('highlight');
        }
        cy.getElementById(node).addClass('highlight');
      });
    }
  </script>
</body>
</html>
