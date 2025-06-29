# -7xzzyuii_-
勝率計算器
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8">
  <title>勝率計算器</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 30px;
      background-color: #f4f8fb;
      color: #333;
      max-width: 800px;
      margin: auto;
    }

    h1 {
      color: #0066cc;
      text-align: center;
    }

    label {
      display: block;
      margin-top: 15px;
    }

    input {
      padding: 6px;
      margin-top: 5px;
      width: 100%;
      box-sizing: border-box;
    }

    button {
      margin-top: 20px;
      padding: 10px 20px;
      background-color: #007bff;
      color: white;
      border: none;
      cursor: pointer;
      width: 100%;
    }

    button:hover {
      background-color: #0056b3;
    }

    .result, .history {
      margin-top: 25px;
      padding: 15px;
      background-color: #e9f5ff;
      border-left: 5px solid #007bff;
    }

    canvas {
      margin-top: 20px;
      max-width: 100%;
    }

    table {
      width: 100%;
      margin-top: 15px;
      border-collapse: collapse;
    }

    th, td {
      padding: 8px;
      border: 1px solid #ccc;
      text-align: center;
    }

    @media (max-width: 600px) {
      body {
        margin: 10px;
      }

      button {
        font-size: 16px;
      }
    }
  </style>
</head>
<body>
  <h1>勝率計算器</h1>

  <label>目前勝場數：</label>
  <input type="number" id="wins" placeholder="例如：50">

  <label>目前總場數：</label>
  <input type="number" id="games" placeholder="例如：80">

  <label>目標勝率 (%)：</label>
  <input type="number" id="targetRate" placeholder="例如：60">

  <label>模擬接下來的勝率 (%)：</label>
  <input type="number" id="simulatedRate" placeholder="例如：55">

  <button onclick="calculate()">計算</button>

  <div class="result" id="output"></div>

  <canvas id="winPieChart"></canvas>

  <div class="history">
    <h3>模擬紀錄</h3>
    <table id="historyTable">
      <thead>
        <tr>
          <th>目前勝率</th>
          <th>目標勝率</th>
          <th>全勝需場次</th>
          <th>模擬勝率</th>
          <th>模擬需場次</th>
        </tr>
      </thead>
      <tbody>
      </tbody>
    </table>
  </div>

  <script>
    let pieChart;

    function calculate() {
      const wins = parseInt(document.getElementById("wins").value);
      const games = parseInt(document.getElementById("games").value);
      const targetRate = parseFloat(document.getElementById("targetRate").value) / 100;
      const simulatedRate = parseFloat(document.getElementById("simulatedRate").value) / 100;

      if (isNaN(wins) || isNaN(games) || isNaN(targetRate) || isNaN(simulatedRate) || games < wins || wins < 0 || games <= 0) {
        document.getElementById("output").innerHTML = "<b>請輸入正確的數值。</b>";
        return;
      }

      const currentRate = (wins / games) * 100;

      // 計算：全勝還需幾場
      let extraGames = 0;
      let tempWins = wins;
      let tempGames = games;
      while ((tempWins / tempGames) < targetRate && extraGames < 10000) {
        tempWins += 1;
        tempGames += 1;
        extraGames++;
      }

      // 模擬勝率計算
      let simGames = 0;
      let simTotalWins = wins;
      let simTotalGames = games;
      while ((simTotalWins / simTotalGames) < targetRate && simGames < 10000) {
        simGames++;
        simTotalGames++;
        simTotalWins += simulatedRate;
      }

      document.getElementById("output").innerHTML = `
        <b>目前勝率：</b>${currentRate.toFixed(2)}%<br>
        <b>全勝情況下，需再打：</b>${extraGames} 場<br>
        <b>若你用 ${(simulatedRate * 100).toFixed(1)}% 勝率打，預估需打：</b>${simGames} 場
      `;

      updateChart(wins, games - wins);
      addToHistory(currentRate, targetRate * 100, extraGames, simulatedRate * 100, simGames);
    }

    function updateChart(wins, losses) {
      const ctx = document.getElementById('winPieChart').getContext('2d');
      if (pieChart) {
        pieChart.destroy();
      }

      pieChart = new Chart(ctx, {
        type: 'pie',
        data: {
          labels: ['勝場', '敗場'],
          datasets: [{
            label: '勝敗比例',
            data: [wins, losses],
            backgroundColor: ['#28a745', '#dc3545'],
          }]
        },
        options: {
          responsive: true,
          plugins: {
            legend: {
              position: 'bottom',
            }
          }
        }
      });
    }

    function addToHistory(current, target, extraGames, simRate, simGames) {
      const table = document.getElementById("historyTable").querySelector("tbody");
      const row = document.createElement("tr");

      row.innerHTML = `
        <td>${current.toFixed(2)}%</td>
        <td>${target.toFixed(1)}%</td>
        <td>${extraGames}</td>
        <td>${simRate.toFixed(1)}%</td>
        <td>${simGames}</td>
      `;

      table.prepend(row);
    }
  </script>
</body>
</html>
