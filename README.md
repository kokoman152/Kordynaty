# Kordynaty
Liczenie kordynatow
<!DOCTYPE html>
<html lang="pl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Smart Plan Coordinates</title>
  <link rel="manifest" href="manifest.json" />
  <style>
    body { font-family: Arial, sans-serif; margin: 20px; }
    input, button, textarea { margin: 10px 0; padding: 10px; font-size: 16px; width: 100%; box-sizing: border-box; }
    textarea { height: 150px; font-family: monospace; }
    .output { margin-top: 20px; background: #f0f0f0; padding: 10px; }
  </style>
</head>
<body>
  <h2>📐 Smart Plan Coordinates – Prototyp Webowy</h2>
  <p>Wklej współrzędne z planu (np. U=12000 W=2890 S=4000) lub wpisz ręcznie. Możesz dodać kilka punktów, każdy w osobnej linii.</p>
  <textarea id="inputArea" placeholder="U=12000 W=2890 S=4000\nU=11980 W=3890 S=4000"></textarea>
  <input type="number" id="slope" placeholder="Spadek [%]" value="2" />
  <button onclick="calculate()">Oblicz nowe wysokości</button>

  <div class="output" id="outputArea"></div>

  <script>
    if ('serviceWorker' in navigator) {
      navigator.serviceWorker.register('service-worker.js')
        .then(() => console.log('Service Worker zarejestrowany!'))
        .catch(() => console.log('Rejestracja Service Workera nie powiodła się'));
    }

    function parseLine(line) {
      const u = parseFloat((line.match(/U=([\\d\\.]+)/i) || [])[1]);
      const w = parseFloat((line.match(/W=([\\d\\.]+)/i) || [])[1]);
      const s = parseFloat((line.match(/S=([\\d\\.]+)/i) || [])[1]);
      return { u, w, s };
    }

    function distance(p1, p2) {
      const dx = p2.w - p1.w;
      const dy = p2.s - p1.s;
      return Math.sqrt(dx * dx + dy * dy);
    }

    function calculate() {
      const input = document.getElementById("inputArea").value.trim().split("\\n");
      const slope = parseFloat(document.getElementById("slope").value) / 100;
      const points = input.map(parseLine).filter(p => !isNaN(p.u) && !isNaN(p.w) && !isNaN(p.s));

      let output = "<b>📊 Wyniki:</b><br><table border='1' cellpadding='5'><tr><th>Punkt</th><th>W</th><th>S</th><th>U (oryg.)</th><th>Długość [mm]</th><th>Spadek [mm]</th><th>Nowe U</th></tr>";

      for (let i = 0; i < points.length - 1; i++) {
        const p1 = points[i];
        const p2 = points[i + 1];
        const d = distance(p1, p2);
        const deltaH = d * slope;
        const newU = p1.u - deltaH;
        output += `<tr><td>${i + 1}</td><td>${p2.w}</td><td>${p2.s}</td><td>${p1.u.toFixed(2)}</td><td>${d.toFixed(2)}</td><td>${deltaH.toFixed(2)}</td><td>${newU.toFixed(2)}</td></tr>`;
      }

      output += "</table>";
      document.getElementById("outputArea").innerHTML = output;
    }
  </script>
</body>
</html>
