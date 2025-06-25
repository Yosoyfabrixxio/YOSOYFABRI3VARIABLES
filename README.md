<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Graficador 3D de Desigualdades</title>
  <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background: #ecf0f1;
      padding: 0;
      margin: 0;
    }

    .container {
      max-width: 900px;
      margin: 40px auto;
      background: #fff;
      padding: 30px 40px;
      border-radius: 12px;
      box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
    }

    h1 {
      text-align: center;
      color: #2c3e50;
      margin-bottom: 30px;
    }

    label {
      font-weight: 600;
      display: block;
      margin-top: 20px;
    }

    input {
      width: 100%;
      padding: 12px;
      font-size: 16px;
      margin-top: 8px;
      border: 1px solid #ccc;
      border-radius: 8px;
      box-sizing: border-box;
    }

    button {
      margin-top: 30px;
      width: 100%;
      background-color: #27ae60;
      color: white;
      padding: 15px;
      font-size: 16px;
      font-weight: bold;
      border: none;
      border-radius: 10px;
      cursor: pointer;
      transition: background 0.3s ease;
    }

    button:hover {
      background-color: #1e8449;
    }

    #plot {
      margin-top: 40px;
      height: 650px;
    }

    .footer {
      text-align: center;
      color: #999;
      font-size: 14px;
      margin-top: 40px;
    }
  </style>
</head>
<body>

  <div class="container">
    <h1>Visualizador 3D de Sistemas de Desigualdades</h1>

    <label for="ineq1">Primera desigualdad (ej: x + y + z &lt;= 10):</label>
    <input type="text" id="ineq1" value="x + y + z <= 10">

    <label for="ineq2">Segunda desigualdad (ej: x - y + 2*z &gt;= 0):</label>
    <input type="text" id="ineq2" value="x - y + 2*z >= 0">

    <button onclick="graficar3D()">Graficar Región de Solución</button>

    <div id="plot"></div>

    <div class="footer">Hecho por Fabricio con Plotly.js y HTML5 — 2025</div>
  </div>

  <script>
    function parseInequality(expression) {
      const parts = expression.match(/(.+?)(<=|>=|<|>)(.+)/);
      if (!parts) return null;
      return {
        left: parts[1].trim(),
        op: parts[2],
        right: parts[3].trim()
      };
    }

    function evaluateInequality(x, y, z, inequality) {
      try {
        const left = inequality.left.replace(/x/g, `(${x})`)
                                    .replace(/y/g, `(${y})`)
                                    .replace(/z/g, `(${z})`);
        const right = inequality.right.replace(/x/g, `(${x})`)
                                      .replace(/y/g, `(${y})`)
                                      .replace(/z/g, `(${z})`);
        return eval(`${eval(left)} ${inequality.op} ${eval(right)}`);
      } catch {
        return false;
      }
    }

    function graficar3D() {
      const expr1 = document.getElementById('ineq1').value;
      const expr2 = document.getElementById('ineq2').value;

      const ineq1 = parseInequality(expr1);
      const ineq2 = parseInequality(expr2);

      if (!ineq1 || !ineq2) {
        alert("Por favor ingresa dos desigualdades válidas.");
        return;
      }

      const xVals = [], yVals = [], zVals = [];

      const rangeMin = -5, rangeMax = 5, step = 0.5;

      for (let x = rangeMin; x <= rangeMax; x += step) {
        for (let y = rangeMin; y <= rangeMax; y += step) {
          for (let z = rangeMin; z <= rangeMax; z += step) {
            if (evaluateInequality(x, y, z, ineq1) && evaluateInequality(x, y, z, ineq2)) {
              xVals.push(x);
              yVals.push(y);
              zVals.push(z);
            }
          }
        }
      }

      if (xVals.length === 0) {
        alert("No se encontró región solución. Intenta con otras desigualdades.");
        Plotly.purge('plot');
        return;
      }

      const trace = {
        x: xVals,
        y: yVals,
        z: zVals,
        mode: 'markers',
        type: 'scatter3d',
        name: 'Solución',
        marker: {
          color: 'rgba(39, 174, 96, 0.6)',
          size: 3,
          symbol: 'circle'
        }
      };

      const layout = {
        title: `Región común en 3D`,
        scene: {
          xaxis: { title: 'x', backgroundcolor: '#f9f9f9' },
          yaxis: { title: 'y', backgroundcolor: '#f9f9f9' },
          zaxis: { title: 'z', backgroundcolor: '#f9f9f9' },
        },
        paper_bgcolor: '#ffffff',
        margin: { l: 0, r: 0, b: 0, t: 50 }
      };

      Plotly.newPlot('plot', [trace], layout, { responsive: true });
    }
  </script>

</body>
</html>

    
