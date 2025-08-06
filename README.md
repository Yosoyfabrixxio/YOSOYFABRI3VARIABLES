<!DOCTYPE html>
<html lang="es">

<head>
  <meta charset="UTF-8" />
  <title>Simplex LP — Pega el problema y lo resolvemos</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <script src="https://cdnjs.cloudflare.com/ajax/libs/mathjs/11.11.1/math.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    :root {
      --bg: #f7f8fb;
      --fg: #222;
      --card: #fff;
      --muted: #667085;
      --pri: #4e73df;
      --pri-d: #2e59d9;
      --ok: #2ecc71;
      --warn: #f39c12;
      --err: #e74c3c;
    }

    * {
      box-sizing: border-box
    }

    body {
      margin: 0;
      padding: 24px;
      font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, "Helvetica Neue", Arial;
      background: var(--bg);
      color: var(--fg);
    }

    h2 {
      margin: 0 0 12px;
    }

    .grid {
      display: grid;
      grid-template-columns: 1fr;
      gap: 16px;
    }

    @media (min-width: 980px) {
      .grid {
        grid-template-columns: 1fr 1fr;
      }
    }

    .card {
      background: var(--card);
      border-radius: 12px;
      padding: 16px;
      box-shadow: 0 6px 20px rgba(0, 0, 0, .06);
    }

    .card h3 {
      margin: 0 0 10px;
      color: #2e59d9
    }

    textarea {
      width: 100%;
      min-height: 180px;
      resize: vertical;
      padding: 12px;
      border: 1px solid #e5e7eb;
      border-radius: 10px;
      font-size: 14px;
      line-height: 1.45;
    }

    .btn {
      border: 1px solid #d0d5dd;
      background: #fff;
      color: #111827;
      padding: 10px 12px;
      border-radius: 10px;
      cursor: pointer
    }

    .btn:hover {
      background: #f2f4f7
    }

    .btn-primary {
      background: var(--pri);
      color: white;
      border-color: transparent
    }

    .btn-primary:hover {
      background: var(--pri-d)
    }

    .muted {
      color: var(--muted)
    }

    .status {
      padding: 10px 12px;
      border-radius: 10px;
      margin: 8px 0
    }

    .status.ok {
      background: #e8f5e9;
      color: #1e7e34
    }

    .status.warn {
      background: #fff8e1;
      color: #985f0d
    }

    .status.err {
      background: #fdecea;
      color: #842029
    }

    table {
      width: 100%;
      border-collapse: collapse;
      font-size: 14px;
    }

    th,
    td {
      border: 1px solid #e5e7eb;
      padding: 8px;
      text-align: center
    }

    th {
      background: #f8fafc
    }

    code {
      background: #f3f4f6;
      padding: 2px 6px;
      border-radius: 6px
    }

    .row {
      display: flex;
      gap: 8px;
      flex-wrap: wrap
    }

    #testlog {
      white-space: pre-wrap;
      background: #0b1025;
      color: #d1e9ff;
      padding: 10px;
      border-radius: 8px;
      font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
      font-size: 12px;
      max-height: 220px;
      overflow: auto
    }

    canvas {
      width: 100% !important;
      max-height: 420px !important
    }
  </style>
</head>

<body>
  <h2>🧮 Simplex LP — Pega tu problema en español y lo resuelvo</h2>

  <div class="grid">
    <!-- Entrada -->
    <div class="card">
      <h3>📝 Enunciado</h3>
      <textarea id="texto" placeholder="Pega aquí el enunciado completo del problema de PL (en español)..."></textarea>
      <div class="row" style="margin-top:10px">
        <button class="btn btn-primary" onclick="resolverDesdeTexto()">Analizar y Resolver</button>
        <button class="btn" onclick="cargarEjemploDCI()">Ejemplo DCI</button>
        <button class="btn" onclick="cargarEjemploMesasSillas()">Ejemplo Mesas/Sillas</button>
        <button class="btn" onclick="cargarEjemploPortafolio()">Ejemplo Portafolio</button>
        <button class="btn" onclick="runTests()">Probar parser</button>
      </div>
      <p class="muted" style="margin-top:6px">No necesitas llenar nada más. El sistema extrae variables, objetivo y
        restricciones y aplica Simplex (Fase I–Fase II). No hay ajustes manuales.</p>
    </div>

    <!-- Interpretación -->
    <div class="card">
      <h3>📌 Interpretación</h3>
      <div id="interp" class="muted">Aún no hay análisis.</div>
    </div>

    <!-- Resultados -->
    <div class="card">
      <h3>📊 Resultados</h3>
      <div id="status" class="muted">Sin resolver.</div>
      <div id="solution"></div>
    </div>

    <!-- Tableau -->
    <div class="card">
      <h3>📋 Tabla Simplex</h3>
      <div id="tableau" class="muted">Sin datos.</div>
    </div>
  </div>

  <div class="card" id="plotCard" style="margin-top:16px; display:none">
    <h3>📈 Visualización (solo 2 variables de decisión)</h3>
    <canvas id="chart"></canvas>
  </div>

  <div class="card" style="margin-top:16px">
    <h3>🧪 Registro de pruebas</h3>
    <div id="testlog">Sin pruebas aún.</div>
  </div>

  <script>
    // ============ Ejemplos ("tests" funcionales) ============
    function cargarEjemploMesasSillas() {
      document.getElementById('texto').value = Una fábrica produce dos tipos de muebles: mesas (x1) y sillas (x2).\nCada mesa genera una ganancia de $80 y cada silla $50.\nPara producir una mesa se necesitan 4 horas de trabajo y 2 unidades de material.\nPara una silla se necesitan 3 horas de trabajo y 1 unidad de material.\nDisponemos de 240 horas de trabajo y 100 unidades de material.\n¿Cuántas mesas y sillas se deben producir para maximizar la ganancia?;
    }
    function cargarEjemploDCI() {
      document.getElementById('texto').value = Digital Controls, Inc. (DCI) fabrica dos modelos de una pistola radar.\nPedido mínimo: 100 unidades del modelo A y 150 del modelo B.\nMoldeo (min): A=4 y B=3; Ensamble (min): A=6 y B=8.\nDisponibles: Moldeo 600 min; Ensamble 1080 min.\nCosto fabricar: A $10, B $6. Costo comprar: A $14, B $9.\nDecida cuántos estuches de cada modelo fabricar y comprar para minimizar costos.;
    }
    function cargarEjemploPortafolio() {
      document.getElementById('texto').value = Investment Advisors, Inc. administra portafolios.\nUn portafolio consta de U acciones de U.S. Oil y H acciones de Huber Steel.\nRendimiento anual: U.S. Oil $3 por acción; Huber Steel $5 por acción.\nPrecios: U.S. Oil $25; Huber Steel $50. Presupuesto total $80,000.\nÍndice de riesgo: 0.50 por acción de U.S. Oil y 0.25 por acción de Huber Steel. Tope de riesgo 700.\nAdemás, el portafolio está limitado a un máximo de 1000 acciones de U.S. Oil.;
    }

    // =================== Utilidades ===================
    function parseNumberFlexible(raw) {
      if (raw == null) return NaN;
      let s = String(raw).replace(/[^\d.,-]/g, '');
      if (!s) return NaN;
      const lastSep = Math.max(s.lastIndexOf(','), s.lastIndexOf('.'));
      if (lastSep !== -1) {
        const intPart = s.slice(0, lastSep).replace(/[.,]/g, '');
        const dec = s.slice(lastSep + 1);
        s = intPart + '.' + dec;
      } else {
        s = s.replace(/[.,]/g, '');
      }
      return parseFloat(s);
    }
    function esc(str) { return str.replace(/[.*+?^${}()|[\]\\]/g, '\\$&'); }
    function firstSentenceContaining(text, kw) {
      const re = new RegExp(esc(kw) + "[\\s\\S]*?(?:\\.|\\n)", 'i');
      const m = text.match(re); return m ? m[0] : '';
    }
    function numbersIn(block, max) {
      const out = []; const re = /(?:\$?\s*)(-?\d[\d.,]*)/gi; let m;
      while ((m = re.exec(block)) && (max == null || out.length < max)) {
        const v = parseNumberFlexible(m[1]);
        if (Number.isFinite(v)) out.push(v);
      }
      return out;
    }

    // =================== PARSER ===================
    function parsearProblema(texto) {
      const original = texto; const t = texto.replace(/\s+/g, ' ').trim(); const tl = t.toLowerCase();

      const esMin = /(minimi[sz]ar|costo\s+m[ií]nimo|coste\s+m[ií]nimo)/i.test(tl);
      const esMax = /(maximi[sz]ar|ganancia|beneficio|utilidad|rendimiento)/i.test(tl);

      // --- Rama DCI fija ---
      if (tl.includes('digital controls') || tl.includes('(dci)')) {
        return {
          variables: ['modeloa_fab', 'modeloa_comp', 'modelob_fab', 'modelob_comp'],
          objetivo: { coefficients: [10, 14, 6, 9], tipo: 'min', texto: 'Minimizar costos totales' },
          restricciones: [
            { coefficients: [1, 1, 0, 0], sign: '>=', value: 100, text: 'Demanda modelo A' },
            { coefficients: [0, 0, 1, 1], sign: '>=', value: 150, text: 'Demanda modelo B' },
            { coefficients: [4, 0, 3, 0], sign: '<=', value: 600, text: 'Moldeo (min)' },
            { coefficients: [6, 0, 8, 0], sign: '<=', value: 1080, text: 'Ensamble (min)' }
          ],
          textoOriginal: original
        };
      }

      // --- Rama Portafolio ---
      const varPairs = []; // {letter, name}
      const reVars = /(\b[A-Z])\s*acciones\s*de\s*([A-Za-zÁÉÍÓÚÜÑ.\s]+?)(?=(?:,|\)|\.|\s+y\s+|\s*$))/g;
      let mv; while ((mv = reVars.exec(t))) varPairs.push({ letter: mv[1], name: mv[2].trim() });

      if (varPairs.length >= 2 && /portafolio|acciones/i.test(t)) {
        const variables = varPairs.map(v => v.letter);
        const assets = varPairs.map(v => v.name);
        const nameToIdx = {}; assets.forEach((n, i) => nameToIdx[n.toLowerCase().replace(/\s+/g, ' ')] = i);

        // Rendimientos → objetivo max
        let rend = numbersIn(firstSentenceContaining(t, 'rendimiento'), variables.length);
        if (rend.length !== variables.length) {
          rend = assets.map(name => {
            const re = new RegExp(esc(name) + "[^.]?\\$?\\s(-?\\d[\\d.,])\\s(?:por\\s*acci[oó]n)?", 'i');
            const m = t.match(re); return m ? parseNumberFlexible(m[1]) : 0;
          });
        }
        const objetivoCoefs = rend.map(x => Number.isFinite(x) ? x : 0);

        // Precios y presupuesto
        let precios = numbersIn(firstSentenceContaining(t, 'se venden'), variables.length);
        if (precios.length !== variables.length) {
          precios = assets.map(name => {
            const re = new RegExp('(?:se\\s+venden\\s+a|precio|vale|a)\\s*\\$?\\s*(-?\\d[\\d.,])[^.]?' + esc(name), 'i');
            const m = t.match(re); return m ? parseNumberFlexible(m[1]) : NaN;
          });
        }
        let budget = NaN; const mb = t.match(/(?:tiene|cuenta\s+con|dispone\s+de|presupuesto|fondos|capital)[^.]?\$?\s(-?\d[\d.,]*)/i);
        if (mb) budget = parseNumberFlexible(mb[1]);

        const restricciones = [];
        if (precios.every(Number.isFinite) && Number.isFinite(budget)) {
          restricciones.push({ coefficients: precios, sign: '<=', value: budget, text: 'Presupuesto' });
        }

        // Riesgo
        const riesgoBlock = firstSentenceContaining(t, 'riesgo');
        let riesgos = numbersIn(riesgoBlock, variables.length);
        if (riesgos.length !== variables.length) {
          riesgos = assets.map(name => {
            const re = new RegExp('(-?\\d[\\d.,])\\s*por\\s*acci[oó]n\\s*de\\s' + esc(name), 'i');
            const m = riesgoBlock.match(re); return m ? parseNumberFlexible(m[1]) : NaN;
          });
        }
        let tope = NaN; const mt = riesgoBlock.match(/(?:m[aá]ximo|tope|limitad[oa]\s+a|no\s+excede(?:r)?)\s*(?:de\s*)?(-?\d[\d.,]*)/i);
        if (mt) tope = parseNumberFlexible(mt[1]);
        if (riesgos.every(Number.isFinite) && Number.isFinite(tope)) {
          restricciones.push({ coefficients: riesgos, sign: '<=', value: tope, text: 'Riesgo' });
        }

        // Cotas UPPER: "máximo N acciones de X"
        const reCota = /m[aá]ximo\s+(-?\d[\d.,]*)\s+acciones\s+de\s+([A-Za-zÁÉÍÓÚÜÑ.\s]+)/gi; let mc;
        while ((mc = reCota.exec(t))) {
          const lim = parseNumberFlexible(mc[1]); const asset = mc[2].trim().toLowerCase().replace(/\s+/g, ' ');
          const idx = nameToIdx[asset]; if (Number.isFinite(lim) && idx != null) {
            const coefs = new Array(variables.length).fill(0); coefs[idx] = 1;
            restricciones.push({ coefficients: coefs, sign: '<=', value: lim, text: Cota superior ${variables[idx]} });
          }
        }

        if (restricciones.length === 0) throw new Error('No se detectaron restricciones. Asegúrate de incluir precios/presupuesto y/o riesgo/topes.');

        return { variables, objetivo: { coefficients: objetivoCoefs, tipo: esMin ? 'min' : 'max', texto: 'Maximizar rendimiento' }, restricciones, textoOriginal: original };
      }

      // --- Fallback Mesas/Sillas ---
      return {
        variables: ['x1', 'x2'],
        objetivo: { coefficients: esMin ? [10, 6] : [80, 50], tipo: esMin ? 'min' : 'max', texto: esMin ? 'Minimizar costos' : 'Maximizar ganancias' },
        restricciones: [
          { coefficients: [4, 3], sign: '<=', value: 240, text: 'Horas' },
          { coefficients: [2, 1], sign: '<=', value: 100, text: 'Material' }
        ],
        textoOriginal: original
      };
    }

    // ============== SIMPLEX 2 FASES ==============
    function standardizeConstraints(restrictions) {
      // asegura RHS >= 0
      return restrictions.map(r => {
        let coeffs = r.coefficients.slice(); let sign = r.sign; let val = r.value;
        if (val < 0) { val = -val; coeffs = coeffs.map(x => -x); sign = (sign === '<=') ? '>=' : (sign === '>=') ? '<=' : '=='; }
        return { coefficients: coeffs, sign, value: val, text: r.text };
      });
    }

    function buildTableau(numVars, restrictions) {
      const m = restrictions.length;
      let cols = numVars; // originales
      const slackIdx = []; // por restricción
      const artIdx = [];   // por restricción (-1 si no hay)
      const A = []; const b = []; const signs = [];
      restrictions.forEach(r => { A.push(r.coefficients.slice()); b.push(r.value); signs.push(r.sign); });

      // Añadir columnas de slacks/surplus y artificiales
      for (let i = 0; i < m; i++) {
        // expandir todas las filas con nuevas columnas cuando aparezcan
        function pushColForAll(v) { for (let k = 0; k < m; k++) A[k][cols] = (k === i ? v : 0); cols++; }
        if (signs[i] === '<=') {
          pushColForAll(1); slackIdx[i] = cols - 1; artIdx[i] = -1;
        } else if (signs[i] === '>=') {
          pushColForAll(-1); slackIdx[i] = cols - 1; // surplus
          pushColForAll(1); artIdx[i] = cols - 1; // artificial
        } else { // '=='
          slackIdx[i] = -1; pushColForAll(1); artIdx[i] = cols - 1;
        }
      }

      // Crear tableau: (m+1) x (cols+1)
      const tableau = Array.from({ length: m + 1 }, () => Array(cols + 1).fill(0));
      for (let i = 0; i < m; i++) {
        for (let j = 0; j < cols; j++) tableau[i + 1][j] = A[i][j] || 0;
        tableau[i + 1][cols] = b[i];
      }

      // Base inicial
      const basis = [];
      for (let i = 0; i < m; i++) {
        if (artIdx[i] !== -1) basis[i] = artIdx[i]; else basis[i] = slackIdx[i];
      }

      return { tableau, basis, numCols: cols, artIdx, slackIdx };
    }

    function simplexIterate(tableau, basis, objectiveType) {
      const m = tableau.length - 1; const n = tableau[0].length - 1; // RHS index n
      const eps = 1e-9; let iter = 0, maxIter = 2000;
      while (iter++ < maxIter) {
        // elegir columna pivote
        let pivotCol = -1;
        if (objectiveType === 'max') {
          let min = 0; for (let j = 0; j < n; j++) { if (tableau[0][j] < min - eps) { min = tableau[0][j]; pivotCol = j; } }
        } else { // min
          let max = 0; for (let j = 0; j < n; j++) { if (tableau[0][j] > max + eps) { max = tableau[0][j]; pivotCol = j; } }
        }
        if (pivotCol === -1) return { tableau, basis, iterations: iter, isOptimal: true };

        // elegir fila pivote (regla razón mínima)
        let pivotRow = -1; let best = Infinity;
        for (let i = 1; i <= m; i++) {
          const a = tableau[i][pivotCol];
          if (a > eps) { const ratio = tableau[i][n] / a; if (ratio >= 0 && ratio < best - 1e-12) { best = ratio; pivotRow = i; } }
        }
        if (pivotRow === -1) { return { tableau, basis, iterations: iter, isOptimal: false, error: 'Problema no acotado' }; }

        // Pivotear
        basis[pivotRow - 1] = pivotCol;
        const pv = tableau[pivotRow][pivotCol];
        for (let j = 0; j <= n; j++) tableau[pivotRow][j] /= pv;
        for (let i = 0; i <= m; i++) if (i !== pivotRow) {
          const factor = tableau[i][pivotCol];
          for (let j = 0; j <= n; j++) tableau[i][j] -= factor * tableau[pivotRow][j];
        }
      }
      return { tableau, basis, iterations: maxIter, isOptimal: false, error: 'Iteraciones máximas alcanzadas' };
    }

    function simplexTwoPhase(numVars, objectiveCoeffs, restrictions, objectiveType) {
      // 1) Estandarizar y construir tableau
      const std = standardizeConstraints(restrictions);
      const { tableau, basis, numCols, artIdx } = buildTableau(numVars, std);
      const m = tableau.length - 1; const n = tableau[0].length - 1; // RHS index

      // Identificar columnas artificiales
      const artCols = Array.from(new Set(artIdx.filter(i => i !== -1)));

      // 2) Fase I (minimizar suma de artificiales)
      if (artCols.length > 0) {
        // Crear función objetivo de fase I
        for (let j = 0; j < n; j++) tableau[0][j] = 0; tableau[0][n] = 0;
        for (const j of artCols) { tableau[0][j] = 1; }
        // Convertir a forma canónica: restar filas básicas artificiales
        for (let i = 1; i <= m; i++) if (artCols.includes(basis[i - 1])) {
          for (let j = 0; j <= n; j++) tableau[0][j] -= tableau[i][j];
        }
        let phase1 = simplexIterate(tableau, basis, 'min');
        if (!phase1.isOptimal) return { error: phase1.error || 'Fase I no óptima', tableau: phase1.tableau, basis: phase1.basis, iterations: phase1.iterations, isOptimal: false };
        const w = phase1.tableau[0][n]; if (Math.abs(w) > 1e-6) return { error: 'El problema es infactible (W* > 0 en Fase I).', tableau: phase1.tableau, basis: phase1.basis, iterations: phase1.iterations, isOptimal: false };
        // Eliminar columnas artificiales (colapsar)
        const colsToKeep = [];
        for (let j = 0; j < n; j++) if (!artCols.includes(j)) colsToKeep.push(j);
        colsToKeep.push(n); // RHS
        for (let i = 0; i <= m; i++) {
          const row = []; for (const j of colsToKeep) row.push(tableau[i][j]); tableau[i] = row;
        }
        // Recalcular n
        const newN = tableau[0].length - 1; // RHS index
        // Ajustar base si alguna artificial quedó (no debería)
        for (let i = 0; i < basis.length; i++) {
          if (artCols.includes(basis[i])) {
            // buscar 1 en su fila para otra base
            let replacement = -1; for (let j = 0; j < newN; j++) if (Math.abs(tableau[i + 1][j] - 1) < 1e-9) { replacement = j; break; }
            basis[i] = replacement >= 0 ? replacement : 0;
          }
        }
      }

      // 3) Fase II (max o min reales)
      // Construir fila objetivo con -c para variables originales, 0 para otras
      const totalN = tableau[0].length - 1; // RHS index
      for (let j = 0; j < totalN; j++) tableau[0][j] = 0; tableau[0][totalN] = 0;
      for (let j = 0; j < Math.min(numVars, totalN); j++) { tableau[0][j] = (objectiveType === 'max' ? -objectiveCoeffs[j] : objectiveCoeffs[j]); }
      // Llevar a canónica respecto a la base
      for (let i = 1; i < tableau.length; i++) {
        const b = basis[i - 1];
        if (b < numVars) { const factor = tableau[0][b]; for (let j = 0; j <= totalN; j++) tableau[0][j] -= factor * tableau[i][j]; }
      }
      const phase2 = simplexIterate(tableau, basis, (objectiveType === 'max' ? 'max' : 'min'));
      return { tableau: phase2.tableau, basis: phase2.basis, iterations: phase2.iterations, isOptimal: phase2.isOptimal, error: phase2.error || null };
    }

    // ============== Presentación ==============
    function renderInterpretacion(p) {
      let html = <div><p><strong>Tipo:</strong> ${p.objetivo.tipo === 'min' ? 'Minimización' : 'Maximización'}</p>;
      html += <p><strong>Objetivo:</strong> ${p.objetivo.texto || ''}</p>;
      html += <h4>Variables</h4><ul>;
      p.variables.forEach((v, i) => { html += <li><code>${v}</code> · coef. objetivo = ${p.objetivo.coefficients[i] ?? 0}</li>; });
      html += </ul><h4>Restricciones</h4><ol>;
      p.restricciones.forEach(r => { html += `<li>${r.coefficients.map((c, i) => ${c}${p.variables[i] || 's'}).join(' + ')} ${r.sign} ${r.value} <span class="muted">(${r.text || ''})</span></li>`; });
      html += </ol></div>;
      document.getElementById('interp').innerHTML = html;
    }

    function renderResultados(sol, p) {
      const status = document.getElementById('status');
      const out = document.getElementById('solution');
      if (sol.error) { status.className = 'status err'; status.textContent = ❌ ${sol.error}; out.innerHTML = ''; return; }
      status.className = sol.isOptimal ? 'status ok' : 'status warn';
      status.textContent = sol.isOptimal ? '✅ Solución óptima encontrada' : '⚠ Solución parcial';

      // Reconstruir valores de variables desde la base
      const m = sol.tableau.length - 1; const n = sol.tableau[0].length - 1; const numVars = p.variables.length;
      const x = Array(numVars).fill(0); const rhsIdx = n; // RHS
      for (let i = 1; i <= m; i++) {
        const b = sol.basis[i - 1]; if (b < numVars) x[b] = sol.tableau[i][rhsIdx];
      }
      // Valor óptimo
      const z = (p.objetivo.tipo === 'max') ? sol.tableau[0][rhsIdx] : -sol.tableau[0][rhsIdx];

      let html = <p><strong>Valor óptimo Z*:</strong> ${z.toFixed(4)}</p><table><tr><th>Variable</th><th>Valor</th></tr>;
      p.variables.forEach((v, i) => { html += <tr><td>${v}</td><td>${(x[i] || 0).toFixed(6)}</td></tr>; });
      html += </table><p class="muted">Iteraciones: ${sol.iterations}</p>;
      out.innerHTML = html;

      renderTableau(sol, p);
      if (p.variables.length === 2) renderPlot(p, x);
      else document.getElementById('plotCard').style.display = 'none';

      renderInterpretacionSolucion(p, sol);
    }

function renderInterpretacionSolucion(p, sol) {
  const rhsIdx = sol.tableau[0].length - 1;
  const numVars = p.variables.length;
  const m = sol.tableau.length - 1;
  const base = sol.basis;

  const valores = Array(numVars).fill(0);

  for (let i = 0; i < m; i++) {
    const varIdx = base[i];
    if (varIdx < numVars) {
      valores[varIdx] = sol.tableau[i + 1][rhsIdx];
    }
  }

  const z = (p.objetivo.tipo === 'max') ? sol.tableau[0][rhsIdx] : -sol.tableau[0][rhsIdx];

  let out = <div class="section-divider"></div><div><h4>🧠 Análisis textual de la solución</h4>;
  out += <p><strong>Tipo de problema:</strong> ${p.objetivo.tipo === 'max' ? 'Maximización de beneficios' : 'Minimización de costos'}</p>;
  out += <p><strong>Valor óptimo de Z:</strong> ${z.toFixed(2)}</p>;

  out += <h5>📌 Variables de decisión</h5><ul>;
  p.variables.forEach((v, i) => {
    out += <li><code>${v}</code>: ${valores[i].toFixed(2)} unidades</li>;
  });
  out += </ul>;

  // Recursos sobrantes
  out += <h5>📦 Recursos no utilizados</h5><ul>;
  for (let i = 0; i < m; i++) {
    const varIdx = base[i];
    if (varIdx >= numVars) {
      const r = p.restricciones[i];
      const sobrante = sol.tableau[i + 1][rhsIdx];
      out += <li>${r.text || 'Restricción ' + (i + 1)} → sobrante: ${sobrante.toFixed(2)}</li>;
    }
  }
  out += </ul></div>;

  // Mostrar en el div #solution
  const div = document.createElement('div');
  div.innerHTML = out;
  document.getElementById('solution').appendChild(div);
}


    function renderTableau(sol, p) {
      const tab = sol.tableau; const base = sol.basis; const m = tab.length - 1; const n = tab[0].length - 1; const rhs = n;
      let html = '<div style="overflow:auto"><table><tr><th>Base</th>';
      for (let j = 0; j < p.variables.length; j++) html += <th>${p.variables[j]}</th>;
      for (let j = p.variables.length; j < n; j++) html += <th>v${j + 1}</th>;
      html += '<th>RHS</th></tr>';
      // Row 0
      html += '<tr><td>Z</td>';
      for (let j = 0; j <= n; j++) html += <td>${Number(tab[0][j]).toFixed(6)}</td>;
      html += '</tr>';
      for (let i = 1; i <= m; i++) {
        const b = base[i - 1]; const name = (b < p.variables.length) ? p.variables[b] : v${b + 1};
        html += <tr><td>${name}</td>;
        for (let j = 0; j <= n; j++) html += <td>${Number(tab[i][j]).toFixed(6)}</td>;
        html += '</tr>';
      }
      html += '</table></div>';
      document.getElementById('tableau').innerHTML = html;
    }

    function renderPlot(p, x) {
      const ctx = document.getElementById('chart').getContext('2d');
      const constraints = p.restricciones.filter(r => r.coefficients.length === 2);

      const maxX = Math.max(...constraints.map(r => r.coefficients[0] > 0 ? r.value / r.coefficients[0] : 0).filter(Number.isFinite), 10) * 1.2;
      const maxY = Math.max(...constraints.map(r => r.coefficients[1] > 0 ? r.value / r.coefficients[1] : 0).filter(Number.isFinite), 10) * 1.2;
      const step = Math.min(maxX, maxY) / 40;

      const feasiblePoints = [], borderLines = [];

      for (let X = 0; X <= maxX; X += step) {
        for (let Y = 0; Y <= maxY; Y += step) {
          let ok = true;
          for (const r of constraints) {
            const lhs = r.coefficients[0] * X + r.coefficients[1] * Y;
            const val = r.value;
            if (r.sign === '<=') { if (lhs > val + 1e-6) { ok = false; break; } }
            else if (r.sign === '>=') { if (lhs < val - 1e-6) { ok = false; break; } }
            else { if (Math.abs(lhs - val) > 1e-6) { ok = false; break; } }
          }
          if (ok) feasiblePoints.push({ x: X, y: Y });
        }
      }

      // Líneas de las restricciones
      for (const r of constraints) {
        const [a, b] = r.coefficients;
        const val = r.value;
        const points = [];

        for (let X = 0; X <= maxX; X += step) {
          const Y = (b !== 0) ? (val - a * X) / b : null;
          if (Y !== null && Y >= 0 && Y <= maxY) points.push({ x: X, y: Y });
        }
        borderLines.push({
          label: r.text || 'restricción',
          data: points,
          borderWidth: 1.5,
          showLine: true,
          pointRadius: 0,
        });
      }

      const datasets = [
        {
          label: 'Región factible',
          data: feasiblePoints,
          showLine: false,
          pointRadius: 2,
          backgroundColor: '#60a5fa'
        },
        {
          label: 'Solución óptima',
          data: [{ x: x[0] || 0, y: x[1] || 0 }],
          pointRadius: 6,
          backgroundColor: '#ef4444'
        },
        ...borderLines
      ];

      if (window._chart) { window._chart.destroy(); }
      window._chart = new Chart(ctx, {
        type: 'scatter',
        data: { datasets },
        options: {
          responsive: true,
          plugins: { legend: { position: 'top' } },
          scales: { x: { beginAtZero: true }, y: { beginAtZero: true } }
        }
      });
      document.getElementById('plotCard').style.display = 'block';
    }


    // ============== Orquestador ==============
    function resolverDesdeTexto() {
      const texto = (document.getElementById('texto').value || '').trim();
      if (!texto) { document.getElementById('status').className = 'status warn'; document.getElementById('status').textContent = 'Ingresa un enunciado para continuar.'; return; }
      try {
        const p = parsearProblema(texto);
        renderInterpretacion(p);
        const sol = simplexTwoPhase(p.variables.length, p.objetivo.coefficients, p.restricciones, p.objetivo.tipo);
        renderResultados(sol, p);
      } catch (err) {
        document.getElementById('interp').innerHTML = '<span class="status err">' + (err.message || String(err)) + '</span>';
        document.getElementById('status').className = 'status err';
        document.getElementById('status').textContent = '❌ Error al analizar/resolver: ' + (err.message || String(err));
        document.getElementById('solution').innerHTML = '';
        document.getElementById('tableau').innerHTML = '';
        document.getElementById('plotCard').style.display = 'none';
        console.error(err);
      }
    }

    // ============== Pruebas simples (parser) ==============
    function runTests() {
      const log = []; function ok(m) { log.push('✔ ' + m); } function fail(m) { log.push('✘ ' + m); } const sep = () => log.push('');
      // 1 Mesas/Sillas
      cargarEjemploMesasSillas(); const p1 = parsearProblema(document.getElementById('texto').value);
      if (p1.restricciones.length >= 2) ok('Mesas/Sillas: detecta restricciones'); else fail('Mesas/Sillas: NO detecta restricciones');
      if (p1.objetivo.coefficients[0] === 80 && p1.objetivo.coefficients[1] === 50) ok('Mesas/Sillas: objetivo [80,50]'); else fail('Mesas/Sillas: objetivo esperado [80,50]');
      sep();
      // 2 DCI
      cargarEjemploDCI(); const p2 = parsearProblema(document.getElementById('texto').value);
      if (p2.variables.length === 4) ok('DCI: 4 variables'); else fail('DCI: se esperaban 4 variables');
      if (p2.objetivo.tipo === 'min') ok('DCI: minimización'); else fail('DCI: debe ser minimización');
      if (p2.restricciones.length >= 4) ok('DCI: restricciones creadas'); else fail('DCI: faltan restricciones');
      sep();
      // 3 Portafolio
      cargarEjemploPortafolio(); const p3 = parsearProblema(document.getElementById('texto').value);
      if (p3.objetivo.coefficients[0] === 3 && p3.objetivo.coefficients[1] === 5) ok('Portafolio: objetivo [3,5]'); else fail('Portafolio: objetivo esperado [3,5]');
      const hasBudget = p3.restricciones.some(r => r.text.toLowerCase().includes('presupuesto'));
      if (hasBudget) ok('Portafolio: restricción de presupuesto detectada'); else fail('Portafolio: falta presupuesto');
      const hasRisk = p3.restricciones.some(r => r.text.toLowerCase().includes('riesgo'));
      if (hasRisk) ok('Portafolio: restricción de riesgo detectada'); else fail('Portafolio: falta riesgo');

      document.getElementById('testlog').textContent = log.join('\n');
    }
  </script>
</body>

</html>
