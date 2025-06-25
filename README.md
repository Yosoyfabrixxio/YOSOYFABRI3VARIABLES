<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8"> 
  <title>Graficador de Sistema con 3 Desigualdades</title>
  <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background: #f4f6f9;
      margin: 0;
      padding: 20px;
    }

    .container {
      max-width: 800px;
      margin: auto;
      background: #fff;
      padding: 30px;
      border-radius: 12px;
      box-shadow: 0 6px 20px rgba(0,0,0,0.1);
    }

    h1 {
      text-align: center;
      color: #2c3e50;
    }

    label {
      font-weight: bold;
      display: block;
      margin-top: 15px;
    }

    input {
      width: 100%;
      padding: 10px;
      font-size: 16px;
      margin-top: 5px;
      border-radius: 8px;
      border: 1px solid #ccc;
      box-sizing: border-box;
    }

    button {
      margin-top: 25px;
      width: 100%;
      background-color: #3498db;
      color: white;
      padding: 14px;
      font-size: 16px;
      border: none;
      border-radius: 10px;
      cursor: pointer;
    }

    button:hover {
      background-color: #2980b9;
    }

    #plot {
      margin-top: 30px;
      height: 600px;
    }
  </style>
</head>
<body>

  <div class="container">
    <h1>Sistema de 3 Desigualdades en 2 Variables</h1>

    <label for="ineq1">Desigualdad 1 (ej: x + y &lt;= 7):</label>
    <input type="text" id="ineq1" value="x + y <= 7">

    <label for="ineq2">Desigualdad 2 (ej: x - y &gt;= 1):</label>
    <input type="text" id="ineq2" value="x - y >= 1">

    <label for="ineq3">Desigualdad 3 (ej: x &gt;= 0):</label>
    <input type="text" id="ineq3" value="x >= 0">

    <button onclick="graficar()">Graficar Región de Solución</button>

    <div id="plot"></div>
  </div>

  <script>
    function parseInequality(expr) {
      const parts = expr.match(/(.+?)(<=|>=|<|>)(.+)/);
      if (!parts) return null;
      return {
        left: parts[1].trim(),
        op: parts[2],
        right: parts[3].trim()
      };
    }

    function evalInequality(x, y, ineq) {
      try {
        const L = Function("x", "y", `return ${ineq.left}`)(x, y);
        const R = Function("x", "y", `return ${ineq.right}`)(x, y);
        switch (ineq.op) {
          case "<=": return L <= R;
          case ">=": return L >= R;
          case "<": return L < R;
          case ">": return L > R;
        }
      } catch {
        return false;
      }
    }

    function graficar() {
      const ineqs = [
        parseInequality(document.getElementById("ineq1").value),
        parseInequality(document.getElementById("ineq2").value),
        parseInequality(document.getElementById("ineq3").value)
      ];

      if (ineqs.includes(null)) {
        alert("Por favor ingresa desigualdades válidas.");
        return;
      }

      const x = [], y = [];
      const min = -10, max = 10, step = 0.25;

      for (let xi = min; xi <= max; xi += step) {
        for (let yi = min; yi <= max; yi += step) {
          if (ineqs.every(ineq => evalInequality(xi, yi, ineq))) {
            x.push(xi);
            y.push(yi);
          }
        }
      }

      if (x.length === 0) {
        alert("No se encontró solución. Cambia las desigualdades.");
        Plotly.purge("plot");
        return;
      }

      const trace = {
        x: x,
        y: y,
        mode: "markers",
        type: "scatter",
        name: "Región común",
        marker: {
          color: "rgba(46, 204, 113, 0.6)",
          size: 4
        }
      };

      const layout = {
        title: "Región solución del sistema",
        xaxis: { title: "x", zeroline: true, gridcolor: "#ccc" },
        yaxis: { title: "y", zeroline: true, gridcolor: "#ccc" },
        plot_bgcolor: "#f9f9f9",
        paper_bgcolor: "#ffffff",
        showlegend: false
      };

      Plotly.newPlot("plot", [trace], layout, { responsive: true });
    }
  </script>

</body>
</html>
